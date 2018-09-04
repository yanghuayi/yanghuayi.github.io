## typescript 声明第三方组件方法

### 工作背景
> 最近在做共享汽车的的监控平台，使用的是`typescript` + `vue` + `jsx` + `element-ui`+`vuex` + `vue-cli@v3.0`开发的，由于上一个项目使用的是`react` + `redux` + `ant-design`开发的，发现`element-ui`想对于`ant-design`还是有不小的差距，如果不是公司产品、测试、项目经理都想要路由缓存功能做tab页面，个人还是更倾向于用`react`架构。

最开始发现了ant-desgin-vue这个项目，但是没有`types`声明文件，不支持`typescript`。没办法，只有退而求次用`element-ui`, 但是在使用中，感觉跟`ant-design`差距太大。
最近终于抽出时间来弄`ant-design-vue`这个项目的`types`声明文件，废话不多说，上代码：
##### 先新建一个`shims-ant-design-vue.d.ts`文件
```
  declare module 'ant-design-vue' {
    const AntdVue: any;
    export defalut AntdVue
  }
```

上面的代码是最粗暴的写法。直接声明`ant-design-vue`类型为`any`;
