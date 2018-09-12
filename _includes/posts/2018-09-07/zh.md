> 最近一个项目，因为一些原因不得不转用`vue`, 产品和测试都想要原来老系统那种tab页面功能
如图所示：
![tab](/img/2018-09-07-1.png)

下面我就记录一下`vue`tab页面缓存功能实现中遇到过的坑，希望对同志们有所帮助

```javascript (type)
class App extends Vue {
  @Prop() private menuData!: menuItem[];
  // data
  onTabs: any = '1';

  @Watch('$route', { immediate: true, deep: true })
  routeChange(to: any, from: any) {
    this.$store.dispatch('AddTabPane', to.path);
  }

  @Emit()
  removeTab(index: string) {
    // console.log(this);
    this.$store.dispatch('RemoveTab', index);
  }
  @Emit()
  tabChange(index: any) {
    this.tabList.forEach((item: any, indexs: number) => {
      if (item.name === index.name) {
        console.log(item);
        this.$router.push({ name: item.name, params: { id: item.params }, query: item.query });
        this.$store.dispatch('TabChange', index.name);
      }
    });
  }

  tabList = [];
  render() {
    return (
      <el-tabs class="page-tabs" v-model={this.onTabs} type="card" closable={tabList.length > 1} on-tab-click={this.tabChange} on-tab-remove={this.removeTab}>
        {
          tabList.map((item: any, index: number) => <el-tab-pane key={item.path}
          label={item.name} name={item.name}>
          </el-tab-pane>)
        }
      </el-tabs>
    )
  }
}
```

上面的代码没什么坑的地方，拿到tab菜单数据渲染，然后监听删除和点击切换tab事件就行。
```javascript (type)
  @Watch('$route', { immediate: true, deep: true })
  routeChange(to: any, from: any) {
    this.$store.dispatch('AddTabPane', to.path);
  }
```
这段代码是监听路由变化的事件，拿到要跳转的路由地址去执行`vuex`里面的`AddTabPane`的方法
`AddTabPane`方法如下：
```javascript (type)
  AddTabPane: (context: any, url: string) => new Promise((resolve, reject) => {
    const {
      menuData, // 菜单数据
      tabList, // tab数据 
      tabActiveKey, // 当前激活tab-id
      keepList, // keep-alive里面的incloud属性值，用于动态清除路由缓存
    } = context.state;
    let resultData = { tabList, tabActiveKey, key: '' };
    let haveMenu = false; // 用户判断是否已经缓存了路由
    const ArrPath = utils.routeToArray(url);
    tabList.map((item: any) => {
      if (ArrPath.routeArr.indexOf(item.path.replace(/\/:\w+/g, '')) > -1) { // 判断是否有了需要新增的tab页面
        resultData.tabActiveKey = item.name; // 激活当前需要新增的tab页面
        haveMenu = true; 
        return false;
      }
      return item;
    });
    if (!haveMenu) { // 如果没有包含新增的，就新增一个tab页面
      const rep = /\?.+/g;
      const query = window.location.href.match(rep); // 用于保存路由的query参数
      resultData = findMenu(menuData, ArrPath.routeArr, tabList, tabActiveKey, ArrPath.params, qs.parse(query ? query[0].substring(1, query[0].length) : '')); // 递归循环匹配到相应的路由数据
      if (resultData.tabActiveKey) { // 判断是否匹配相应的路由数据
        context.dispatch('addKeep', resultData.key); // 保存数据
      }
    }
    context.commit('KEEP_CHANGE', keepList); // 保存数据
    context.commit('TAB_CHANGE', resultData); // 保存数据
    resolve(true); // 异步返回成功
  }),
```

<span style="color: red">
  这里有个坑，我填了好久，在这里提醒各位同学，关于 `keep-alive`的`include`的，
</span>
请看官方文档说，这里我给大家截了图
![keep-alive](/img/2018-09-07-2.png)

文档上说， `include`匹配首先检查组件自身的`name`选项，如果`name`选项不可用，则匹配它的局部注册名称 (父组件`components`选项的键值)。匿名组件不能被匹配。

在这个项目里面我使用了`typescript`, 我设置了每个路由的`name`，但是`include`就是不能成功匹配到路由的`name`属性，经过多次的尝试，发现，在`typescript`里面生成的路由，`name`属性就是`class`类的类名。

所以要有个一个数组报错需要缓存的路由的`class`类名值
```javascript (type)
  // 新增缓存页面
  addKeep: async (context: any, name: string) => {
    // 新增tab，增加缓存状态
    const { keepList } = context.state;
    if (keepList.indexOf(name) === -1) {// 如果包含了就不用了新增了
      keepList.push(name);
    }
    await context.commit('KEEP_CHANGE', keepList); // 保存数据
  },
```