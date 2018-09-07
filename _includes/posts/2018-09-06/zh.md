## vue 数据绑定问题

> 今天，我下面的人问了我一个问题，在使用`element-ui`的时候`select`组件的时候点击没有任何反应，但是在关闭弹出层的时候闪了下最后应该出现的状态

这个问题，我原来就遇到过的，今天在这里记录一下，方便后面的同学排查此问题

出现这个问题，从深层次来说说,`vue`和`react`都会出现此问题

请看代码示例:
```
  import Vue form 'vue'

  class Home extends Vue {
    data () {
      modelForm: {
        roleIdList: ['1', '2'],
      },
      roleTypeList: [
        {
          value: '1',
          lable: '类型1',
        },
        {
          value: '2',
          lable: '类型2',
        },
        {
          value: '3',
          lable: '类型3',
        }
      ]
    }

    render () {
      return (
        <div>
          <el-form model={this.modelForm}>
            <el-form-item label="角色类型" prop="roleIdList">
              <el-select
                id="roleIdList"
                v-model={this.modelForm.roleIdList}
                multiple={true}
                filterable={true}
                style="width:100%"
              >
                {
                  this.roleTypeList.map((item: any) => (
                    <el-option value={item.value} label={item.label} >{item.label}</el-option>
                  ))
                }
              </el-select>
            </el-form-item>
          </el-form>
        </div>
      )
    }
  }
```

你会发现点击`select`下面的选项，`select`的状态没发生任何变化，这是因为`vue`数据双向绑定，最多只能监听到对象下面第二层的基本类型数据，
在这里`modelForm`是第一层，`roleIdList`是第二层，但是`roleIdList`是数组类型，不是js里面的基本数据类型，在这里说一下`js`里面的数据基本类型有5种：
1. `Boolean`
2. `String`
3. `Number`
4. `Null`
5. `undefind`
如果再加上`ES6`里面的两种就有7种:
1. `Map`
2. `Set`

所以在上面的代码中`roleIdList`数组的变化是监听不到的，除非是对`roleIdList`数据进行重新赋值，就是`this.modelForm.roleIdList = []`，这种`vue`对数据的`set`事件才能执行，在这里`select`的变化，不是进行的赋值操作，而是进行的`this.modelForm.roleIdList.push('2')`或者`this.modelForm.roleIdList.splice()`操作，是用的数组方法。

在`vue`的数据双向绑定中是使用的`ES5`中的`Object.defineProperty`方法，
```
  var Book = {}
  var name = '';
  Object.defineProperty(Book, 'name', {
    set: function (value) {
      name = value;
      console.log('你取了一个书名叫做' + value);
    },
    get: function () {
      return '《' + name + '》'
    }
  })
  
  Book.name = 'vue权威指南';  // 你取了一个书名叫做vue权威指南
  console.log(Book.name);  // 《vue权威指南》
```

这个只有进行赋值操作才会触发`set`函数，所以点击`select`没有触发`set`，自然`vue`不会去更新视图了，解决办法就是用一个第一层的变量，跟上面的`roleIdList`一同发生变化，并影响页面上的元素去触发`vue`去更新视图，

```
  import Vue form 'vue'

  class Home extends Vue {
    data () {
      modelForm: {
        roleIdList: ['1', '2'],
      },
      roleTypeList: [
        {
          value: '1',
          lable: '类型1',
        },
        {
          value: '2',
          lable: '类型2',
        },
        {
          value: '3',
          lable: '类型3',
        }
      ],
      change: false,
    }

    selectChange() {
      this.change = !this.change;
    }

    render () {
      return (
        <div>
          <el-form model={this.modelForm}>
            <el-form-item label="角色类型" prop="roleIdList">
              <el-select
                id="roleIdList"
                v-model={this.modelForm.roleIdList}
                multiple={true}
                filterable={true}
                on-change={this.selectChange}
                placeholder={this.change ? '请选择角色类型' : '请选择角色类型'}
                style="width:100%"
              >
                {
                  this.roleTypeList.map((item: any) => (
                    <el-option value={item.value} label={item.label} >{item.label}</el-option>
                  ))
                }
              </el-select>
            </el-form-item>
          </el-form>
        </div>
      )
    }
  }
```