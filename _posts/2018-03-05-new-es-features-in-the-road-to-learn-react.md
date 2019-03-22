---
title: 《The Road To Learn React》中的ECMAScript新特性
categories: programming
tags: javascript, react
---

# 《The Road To Learn React》中的ECMAScript新特性

#### [`const`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const)/[`let`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)

用`const`定义常量，`let`定义变量

#### [箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

__一个普通的函数表达式总会定义它自己的this对象，但是箭头函数表达式仍然会使用包含它的语境下（上下文）的this对象__

如果函数只有一个参数，可以移除掉参数的括号，但是如果有多个参数就必须保留这个括号

```javascript
// allowed
item => { ... }
// allowed
(item) => { ... }
// not allowed
item, key => { ... }
// allowed
(item, key) => { ... }
```

#### [类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)

用关键字`class`可以定义一个类，类都有一个用来实例化自己的构造函数`constructor`，这个构造函数可以用来传入参数来赋给类的实例。用关键字`extends`来定义类的继承关系

```javascript
import React, { Component } from 'react';

class App extends Component {
  constructor(props) {
    super(props);
  }

  render() {
    // ...
  }
}

const app = new App();
app.render();
```

#### [Object.assign()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

第一个参数作为目标对象，后面的所有参数作为源对象，然后把所有的源对象合并到目标对象中

#### [对象初始化](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Object_initializer)

* 当对象中的属性名与变量名相同时，可以通过简写属性更加简洁地初始化对象

  ```javascript
  const name = 'Robin';
  const user = { name: name, };
  // 等同于
  const user = { name, };
  ```

* ES6中还能更简洁地初始化一个对象的方法

  ```javascript
  // ES5
  var userService = {
    getUserName: function (user) {
      return user.firstname + ' ' + user.lastname;
    },
  };

  // ES6
  const userService = {
    getUserName(user) {
      return user.firstname + ' ' + user.lastname;
    },
  };
  ```

* 可以在ES6中使用计算属性名

  ```javascript
  // ES5
  var user = {
    name: 'Robin',
  };

  // ES6
  const key = 'name';
  const user = {
    [key]: 'Robin',
  };
  ```

* 扩展操作符

  `...`结果类似Ruby中数组的`*`，当使用它时，数组或对象中的每一个值都会被拷贝到一个新的数组或对象。有了扩展操作符，上述提到的`Object.assign()`几乎就用不到了

  ```javascript
  const userList = ['Robin', 'Andrew', 'Dan'];
  const additionalUser = 'Jordan';
  const allUsers = [ ...userList, additionalUser ];
  console.log(allUsers); // output: ['Robin', 'Andrew', 'Dan', 'Jordan']

  const userNames = { firstname: 'Robin', lastname: 'Wieruch' };
  const age = 28;
  const user = { ...userNames, age };
  console.log(user); // output: { firstname: 'Robin', lastname: 'Wieruch', age: 28 }
  ```

#### [剩余参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Rest_parameters)

如果函数的最后一个命名参数以`...`为前缀，则它将成为一个数组，类似Ruby里形参中出现的`*`

```javascript
function(a, b, ...theArgs) {
  // ...
}
```

#### [解构](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

在ES6中有一种更方便的方法来访问对象和数组的属性，对象能将属性解构到与key同名的变量，而数组则按元素的顺序进行解构

```javascript
const user = {
  firstname: 'Robin',
  lastname: 'Wieruch',
};

// ES5
var firstname = user.firstname;
var lastname = user.lastname;
console.log(firstname + ' ' + lastname); // output: Robin Wieruch

// ES6
const { firstname, lastname } = user;
console.log(firstname + ' ' + lastname); // output: Robin Wieruch
```

```javascript
const users = ['Robin', 'Andrew', 'Dan'];
const [ userOne, userTwo, userThree ] = users;
console.log(userOne, userTwo, userThree); // output: Robin Andrew Dan
```

若知道传入方法的参数是会一个变量或者是数组，则可以在形参处解构

```javascript
const Search = (props) => {
  const { value, onChange, children } = props;
  // ...
}

// 等同于

const Search = ({ value, onChange, children }) => {
  // ...
}
```

可以将解构与剩余参数一起使用

```javascript
const user = { firstname: 'Robin', lastname: 'Wieruch', };
const { firstname, ...args } = user

console.log(firstname) // Robin
console.log(args) // {lastname: 'Wieruch'}

const array = [1, 2, 3, 4, 5];
const { i, j, ...k } = array;

console.log(i) // 1
console.log(j) // 2
console.log(k) // [3, 4, 5]
```

可以在解构对象的时候给变量一个默认值

```javascript
const { onClick, className = '', children, } = this.props;
```

#### [模版字符串](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/template_strings)

使用反引号<code>``</code>初始化字符串可以在其中使用插值功能

```javascript
// ES6
const url = `${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${DEFAULT_QUERY}`;

// ES5
var url = PATH_BASE + PATH_SEARCH + '?' + PARAM_SEARCH + DEFAULT_QUERY;
```

#### 模块[`import`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import)/[`export`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/export)

ES6中你可以从模块中导入和导出某些功能，这些功能可以是函数、类、组件、常量等等

* 导出一个或者多个变量，这称为一个命名的导出

  ```javascript
  // file1.js
  const firstname = 'robin';
  const lastname = 'wieruch';

  export { firstname, lastname };
  ```

  可以省略多余行直接导出变量

  ```javascript
  export const firstname = 'robin';
  export const lastname = 'wieruch';
  ```

  ```javascript
  // file2.js
  import { firstname, lastname } from './file1.js';

  console.log(firstname); // output: robin
  ```

* 可以用对象的方式导入另外文件的全部变量

  ```javascript
  // file2.js
  import * as person from './file1.js';

  console.log(person.firstname); // output: robin
  ```

* 输入多个文件时可能存在有相同命名的导出的时候，此时可以在导入时使用别名

  ```javascript
  // file2.js

  import { firstname as foo } from './file1.js';

  console.log(foo); // output: robin
  ```

* 还存在一种default语句，在导入default输出时可省略花括号

  ```javascript
  // file1.js
  const robin = { firstname: 'robin', lastname: 'wieruch', };

  export default robin;
  ```

  ```javascript
  import developer from './file1.js';

  console.log(developer); // output: { firstname: 'robin', lastname: 'wieruch' }
  ```

  输入的名称可以与导入的 default 名称不一样

  ```javascript
  // file1.js
  const firstname = 'robin';
  const lastname = 'wieruch';
  const person = { firstname, lastname, };

  export { firstname, lastname, };
  export default person;
  ```

  ```javascript
  // file2.js
  import developer, { firstname, lastname } from './file1.js';

  console.log(developer); // output: { firstname: 'robin', lastname: 'wieruch' }
  console.log(firstname, lastname); // output: robin wieruch
  ```

* 若一个目录下存在多个模块，可以在该目录下定义一个`index.js`，在该文件中引入其他模块并导出，则其他文件在引入该目录下的模块时可以只导入这个`index.js`，且可以省略`index.js`

  ```
  假设有如下目录结构

  src/
    index.js
    App/
      index.js
    Buttons/
      index.js
      SubmitButton.js
      SaveButton.js
      CancelButton.js
  ```

  ```javascript
  // src/Buttons/index.js
  import SubmitButton from './SubmitButton';
  import SaveButton from './SaveButton';
  import CancelButton from './CancelButton';

  export { SubmitButton, SaveButton, CancelButton, };
  ```

  ```javascript
  // src/App/index.js
  import { SubmitButton, SaveButton, CancelButton } from '../Buttons';
  ```

