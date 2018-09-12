
## 关于antd中form.create组件refs调用子组件方法

> 今天在修复一个应用bug时候遇到一个问题，特意记录一下，方便以后的同学能尽快解决此问题


今天遇到一个场景，组件外部变化，影响form组件，需要重置form组件到初始化值，我们知道在antd中的form有一个方法叫`form.create`,使用表单的时候，一般情况是要使用此方法包裹react组件，这样props里面才有`resetFields`等这些方法

```javascript (type)
class CustomizedForm extends React.Component { ... }

/** use wrappedComponentRef **/
const EnhancedForm =  Form.create()(CustomizedForm);
<EnhancedForm wrappedComponentRef={(form) => this.form = form} />
this.form // => The instance of CustomizedForm
```
