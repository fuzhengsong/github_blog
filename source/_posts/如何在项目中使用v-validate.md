---
title: 如何在项目中使用v-validate
date: 2018-01-05 10:48:26
tags:
- 验证
- vue
- js
categories:
 - js
---
##  引入项目

官方文档都有具体的引入方法：

第一种： npm引入项目

``` sql
npm install vee-validate --save
```
  在vue项目中使用vee-validate

``` javascript
import Vue from 'vue';
import VeeValidate from 'vee-validate';

Vue.use(VeeValidate);
```

第二种： 直接引入项目

``` vbscript-html
  <script src="path/to/vue.js"></script>
  <script src="path/to/vee-validate.js"></script>
  <script>
    Vue.use(VeeValidate); // good to go. 
  </script>
```
为了避免一些冲突，官方文档给出了一些配置：

``` javascript
const config = {
  // 这两个数据对象会注入到vue实例的data对象中去
  errorBagName: 'errors', // change if property conflicts, 错误信息对象
  fieldsBagName: 'fields',   // 验证字段的状态对象
  delay: 0,
  locale: 'en',     // 显示的语言
  dictionary: null,
  strict: true,
  classes: false,
  classNames: {
    touched: 'touched', // the control has been blurred
    untouched: 'untouched', // the control hasn't been blurred
    valid: 'valid', // model is valid
    invalid: 'invalid', // model is invalid
    pristine: 'pristine', // control has not been interacted with
    dirty: 'dirty' // control has been interacted with
  },
  events: 'input|blur',  // 触发验证的事件
  inject: true,
  validity: false,
  aria: true,
  i18n: null, // the vue-i18n plugin instance,
  i18nRootKey: 'validations' // the nested key under which the validation messsages will be located
};
```
在项目中使用这些配置： 

``` actionscript
Vue.use(VeeValidate，config);
```
<!--more-->
## 如何使用呢
引入了库之后，如何在项目页面中使用：
### 基本使用
根据veeValidate中的内置规则进行验证：

``` scala
 <div class="form-group">
      <label for="exampleInputEmail1">Email address</label>
      <input type="email"
             class="form-control"
             id="exampleInputEmail1"
             placeholder="Email"
             name="email"
             v-validate="'required|email'"
             autocomplete='off'
      >
      <span v-show="errors.has('email')" class="text-left text-danger">{{ errors.first('email') }}</span>
    </div>
```
v-validate可以添加需要校验的规则，需要注意的是**name字段是必须的**，不然这个字段不会被验证。
这里的errors属性就是被注入到data中的错误信息对象：

==errors.first('field') - 获取关于当前field的第一个错误信息
collect('field') - 获取关于当前field的所有错误信息(list)
has('field') - 当前filed是否有错误(true/false)
all() - 当前表单所有错误(list)
any() - 当前表单是否有任何错误(true/false)
add(String field, String msg) - 添加错误
clear() - 清除错误
count() - 错误数量
remove(String field) - 清除指定filed的所有错误==

errors.has('email') 表示的就是email字段是否有错误信息，返回的是一个boolean值
errors.first('email') 表示的就是获取email字段的第一个错误信息，因为2个验证规则required和email对应了2个错误信息。

### 默认规则和自定义规则
#### 默认规则
列几条表单验证中的内置的常用规则。
alpha - 只包含英文字符
alpha_dash - 可以包含英文、数字、下划线、破折号
alpha_num - 可以包含英文和数字
between:{min},{max} - 在min和max之间的数字
confirmed:{target} - 必须和target一样  常用在密码的再次判断 target可用表单内的字段
例子：

``` nix
  <input
        type="text"
        class="form-control"
        id="exampleInputPassword1"
        placeholder="Time"
        name="password"
        v-model="password"
        v-validate="'required'"
        autocomplete='off'>
  <input
        type="text"
        class="form-control"
        id="exampleInputPassword1"
        placeholder="Time"
        name="passwordAgain"
        v-model="passwordAgain"
        v-validate="'confirmed:password'"
        autocomplete='off'>
```

decimal:{decimals?} - 数字, decimals表示可以保留的小数点后位数
digits:{length} - 数字， length表示数的位数
dimensions:{width},{height} - 符合宽高规定的图片
email - 邮箱格式
ext:[extensions] - 后缀名
image - 图片
in:[list] - 包含在数组list内的值
ip - ipv4地址
max:{length} - 最大长度为length的字符
min - max相反
max_value:{value} -数字， 最大值为value
min_value:{vaule}
mimes:[list] - 文件类型
not_in - in相反
numeric - 只允许数字
regex:{pattern} - 值必须符合正则pattern
required - 不解释
size:{kb} - 文件大小不超过
url:{domain?} - (指定域名的)url

#### 自定义规则
1. 函数的形式

``` coffeescript
	const validator = (value, args) => {
  		// Return a Boolean or a Promise.
	};
```
基本的形式，返回一个boolean值或者promise对象，错误信息为默认。

2. 对象的形式

``` stata
const validator = {
  getMessage(field, args) {
    // will be added to default English messages.
    // Returns a message.
  },
  validate(value, args) {
    // Returns a Boolean or a Promise.
  }
};
```
可以添加自定义的错误信息。

例子：

``` cs
   // 函数方法自定义
      const validator = (value, args) => {
        return value > 10
      };
      Validator.extend('minNum',validator);

    // 对象方法自定义
      const myRule = {
        getMessage(field, params, data) {
          return (data && data.message) || 'Something went wrong';
        },
        validate(value) {
          return new Promise(resolve => {
            resolve({
              valid: value === 'trigger' ? false : !! value,
              data: value !== 'trigger' ? undefined : { message: 'Not this value' }
            });
          });
        }
      };
      Validator.extend('isTrigger',myRule);
```
在对象方法中，可以在getMessage函数中返回自定义的错误信息（默认添加到英语错误信息中），也可以在validate函数中通过传参的形式，对不同的情况报不同的错误。
validate函数返回一个**boolean值时**，只是判断是否验证通过，如果没有通过，就报getMessage中返回的错误信息。
validate函数返回一个**对象时**，对象中需要包括一个valid值和一个data值，valid值如果返回true则验证通过，返回false则验证失败，data值返回的是错误信息，可以根据不同的情况返回不同的data值，然后在getMessage函数的第三个参数中，接受到这个返回的错误信息。

3. 将自定义规则添加到validator中
一步到位的写法：

``` php
Validator.extend('truthy', {
  getMessage: field => 'The ' + field + ' value is not truthy.',
  validate: value => !! value
});
```

### 修改默认规则的报错信息

``` coffeescript

// 定义一个字典，里面是自定义的信息，还能定义不同的语言
const dictionary = {
  en: {
    messages:{
      alpha: () => 'Some English Message'
    }
  },
  cn: {
  	messages: {
	 alpha: () => '请输入英文字母'
	}
  }
};

// 把这个字典挂到Validator上去
Validator.localize(dictionary);

// 把当前的报错语言改成中文
validator.localize('cn'); 
```

### 通过JS 添加验证规则

1. 直接new一个新的Validator实例
``` javascript
import { Validator } from 'vee-validate';
const validator = new Validator({
  email: 'required|email',
  name: 'required|alpha|min:3'
});
```

2. 通过attach方法添加

``` groovy
const validator = new Validator();

validator.attach({ name: 'email', rules: 'required|email' }); // attach field.

 //添加校验字段 并给当前字段设置一个别名，在errors对象中可获取到
validator.attach({ name: 'name', rules: 'required|alpha', alias: 'Full Name' });

validator.detach('email'); // 删除一个校验字段
```


### 如何在表单提交是验证

验证单个字段

``` nimrod
@param{string} - 需要验证的字段
@param{sting} - 验证的值

validator.validate('email', 'foo@bar.com').then(result => {
  console.log(result);  // true
});
```

多个字段一起验证
``` nimrod
  submitForm(){
        let validator = this.$validator;
		// 对表单中的所有字段进行验证
        validator.validateAll().then(result =>{
		
          // 返回一个result  如果为false，没有通过
          if(!result){
            const messagesObj = {
              'email': {
                'required': '请填写手机号码',
                'mobile': '请填写正确的手机格式'
              },
              'password': {
                'required': '请填写密码',
                'min': '密码至少6位数',
              },
            };
			// this.errors的item属性中是错误信息list
            let messages = this.errors.items[0];
            let msg = messagesObj[messages.field][messages.rule];
            alert(msg);
          }else {
            // 提交表单
          }
        })
```
