---
title: 《The Road To Learn React》读书笔记
categories: programming
tags: javascript, react
---

# 《The Road To Learn React》读书笔记

* JSX中若需给组件传递内联样式，则需传递一个对象，且由于引入变量需花括号，则可能会存在两个花括号

  ```jsx
  <span style={{ width: '40%' }}>
    <a href={item.url}>{item.title}</a>
  </span>
  ```

* `ReactDOM.render()`会使用JSX来替换HTML中的一个DOM节点，有两个传入参数，第一个是准备渲染的JSX，第二个参数指定了React应用在HTML中的放置的位置

  ```jsx
  import React from 'react';
  import ReactDOM from 'react-dom';
  import App from './App';
  import './index.css';

  ReactDOM.render(
    <App />,
    document.getElementById('root')
  );
  ```

* JavaScript内建函数可以在JSX中使用，`map`可以被用来把列表成员渲染成HTML的元素

  ```jsx
  import React, { Component } from 'react';
  import './App.css';
  const list = [
    {
      title: 'React',
      url: 'https://facebook.github.io/react/',
      author: 'Jordan Walke',
      num_comments: 3,
      points: 4,
      objectID: 0,
    }, 
    {
      title: 'Redux',
      url: 'https://github.com/reactjs/redux',
      author: 'Dan Abramov, Andrew Clark',
      num_comments: 2,
      points: 5,
      objectID: 1,
    }, 
  ];

  class App extends Component {
    render() {
      return (
        <div className="App">
          {list.map(function(item) {
            return <div>{item.title}</div>;
          })}
        </div> 
      );
    }
  }

  export default App;
  ```

* 使用ES6编写的组件有一个构造函数时需要强制地调用`super()`方法，也可以调用`super(props)`，它会在你的构造函数中设置`this.props`以供在构造函数中访问它们，否则访问`this.props`会得到`undefined`

* 每次你修改组件的内部状态(`state`),组件的`render`方法会再次运行，但必须使用`setState()`方法来修改它而不是直接修改`state`属性的值

* 当传递方法给事件处理器，若方法带有参数，则必须将其封装到另一个函数中，否则改函数在渲染时将立即执行，且方法也不会成功传递给事件处理器

  ```jsx
  // 正确
  <button
    onClick={() => this.onDismiss(item.objectID)}
    type="button"
  >
    Dismiss
  </button>

  // 错误
  <button
    onClick={this.onDismiss(item.objectID)}
    type="button"
  >
    Dismiss
  </button>
  ```

* 在元素中使用监听时(即传递给事件处理器的方法)可以在回调函数的签名中访问到React的合成事件，可以使用事件对象的`target`属性获得当前元素

  ```jsx
  class App extends Component {
    // ...
    onSearchChange(event) {
      this.setState({ searchTerm: event.target.value });
    }
    // ...
  }
  ```

* `props`对象中存在`children`属性，可将元素从上层传递到你的组件中，且不仅可以传递文本，还可以传递一个元素或者元素树(它还可以再次封装成组件)

  ```jsx
  class App extends Component {
    // ...
    render() {
      const { searchTerm, list } = this.state;
      return (
        <div className="App">
          <Search
            value={searchTerm}
            onChange={this.onSearchChange}
          >
            Search
          </Search>
          <Table
            list={list}
            pattern={searchTerm}
            onDismiss={this.onDismiss}
          />
        </div>
      );
    }
  }

  class Search extends Component {
    render() {
      const { value, onChange, children } = this.props;
      return (
        <form>
          {children} <input
            type="text"
            value={value}
            onChange={onChange}
        />
        </form>
      );
    }
  }
  ```

* 函数式组件无状态且无生命周期

* 箭头函数若函数体只存在一条语句，则返回值为该语句的值，且不用将语句写在花括号内，也不用显式调用`return`语句。由于该特性，函数式组件一般这么写

  ```jsx
  const Search = ({ value, onChange, children }) =>
    <form>
      {children} 
      <input
        type="text"
        value={value}
        onChange={onChange}
      />
    </form>
  ```

  若存在多行，则必须将语句包含在花括号内，且显示调用`return`

  ```jsx
  const Search = ({ value, onChange, children }) => {
    // do something
    return (
      <form>
        {children}
        <input
          type="text"
          value={value}
          onChange={onChange}
        />
      </form>
    ); 
  }
  ```

* 生命周期方法

  * 在组件被挂载的过程中有4个生命周期方法，调用顺序如下

    ```javascript
    constructor()
    componentWillMount() 
    render()
    componentDidMount()
    ```

  * 当组件的状态或者属性改变的时候组件更新有5个生命周期方法，调用顺序如下

    ```javascript
    componentWillReceiveProps() 
    shouldComponentUpdate()
    componentWillUpdate()
    render()
    componentDidUpdate()
    ```

  * 组件卸载也有生命周期，它只有一个生命周期方法`componentWillUnmount()`

  所有生命周期方法使用场景如下

    * constructor(props)
    
      它在组件初始化时被调用。在这个方法中,你可以设置初始化状态以及绑定类方法。

    * componentWillMount()
    
      它在render()方法之前被调用。这就是为什么它可以用作去设置组件内部的状态,因为它不会触发组件的再次渲染。但一般来说,还是推荐在constructor()中去初始化状态。

    * render()
    
      这个生命周期方法是必须有的,它返回作为组件输出的元素。这个方法应该是一个纯函数,因此不应该在这个方法中修改组件的状态。它把属性和状态作为输入并且返回(需要渲染的)元素

    * componentDidMount()
    
      它仅在组件挂载后执行一次。这是发起异步请求去API获取数据的绝佳时期。获取到的数据将被保存在内部组件的状态中然后在render()生命周期方法中展示出来。

    * componentWillReceiveProps(nextProps)
    
      这个方法在一个更新生命周(update lifecycle)中被调用。新的属性会作为它的输入。因此你可以利用this.props来对比之后的属性和之前的属性,基于对比的结果去实现不同的行为。此外,你可以基于新的属性来设置组件的状态。

    * shouldComponentUpdate(nextProps, nextState)
    
      每次组件因为状态或者属性更改而更新时,它都会被调用。你将在成熟的React应用中使用它来进行性能优化。在一个更新生命周期中,组件及其子组件将根据该方法返回的布尔值来决定是否重新渲染。 这样你可以阻止组件的渲染生命周期(render lifecycle)方法,避免不必要的渲染。

    * componentWillUpdate(nextProps, nextState)
    
      这个方法是render()执行之前的最后一个方法。你已经拥有下一个属性和状态,它们可以在这个方法中任由你处置。你可以利用这个方法在渲染之前进行最后的准备。注意在这个生命周期方法中你不能再触发setState()。如果你想基于新的属性计算状态,你必须利用componentWillReceiveProps()。

    * componentDidUpdate(prevProps,prevState)
    
      这个方法在render()之后立即调用。你可以用它当成操作DOM或者执行更多异步请求的机会。

    * componentWillUnmount()
    
      它会在组件销毁之前被调用。你可以利用这个生命周期方法去执行任何清理任务。

  还有另一个生命周期方法`componentDidCatch(error, info)`，用来捕获组件的错误。

* 可以在JSX中使用`if-else`来实现最简单的条件渲染；也可以使用三目运算符，若要渲染空则返回`null`即可；还可以用逻辑运算符`&&`，因为其能阻断执行

  ```jsx
  <div className="page">
    <div className="interactions">
      <Search
        value={searchTerm}
        onChange={this.onSearchChange}
      >
        Search
      </Search>
  </div>
    { result
      ? <Table
          list={result.hits}
          pattern={searchTerm}
          onDismiss={this.onDismiss}
        />
      : null
    }
  </div>
  ```

  ```jsx
  { result &&
    <Table
      list={result.hits}
      pattern={searchTerm}
      onDismiss={this.onDismiss}
    />
  }
  ```

* 可以使用[`Jest`](https://facebook.github.io/jest/)来编写快照测试。在测试文件中，用`test`方法来实现一个快照测试，并可以将其包裹在`describe`方法块中

  ```jsx
  import React from 'react';
  import ReactDOM from 'react-dom';
  import renderer from 'react-test-renderer';
  import App from './App';

  describe('App', () => {
    it('renders without crashing', () => {
      const div = document.createElement('div');
      ReactDOM.render(<App />, div);
    });

    test('has a valid snapshot', () => {
      const component = renderer.create(
        <App />
      );
      let tree = component.toJSON();
      expect(tree).toMatchSnapshot();
    });
  });
  ```

  运行该测试后，会在测试文件的同目录下创建一个目录，用于保存快照。基本上`renderer.create()`函数会创建一份App组件的快照，它会模拟渲染，并将DOM存储在快照中。

  若App的输出发生改变，则该测试会失败，并列出更改且询问是否更新快照，若确定更新快照，则后期执行测试会通过。

  还可以给表格组件一些初始化的`props`来做渲染一个简单的列表

  ```jsx
  import App, { Search, Button, Table } from './App';

  describe('Table', () => {
    const props = {
      list: [
        { title: '1', author: '1', num_comments: 1, points: 2, objectID: 'y' },
        { title: '2', author: '2', num_comments: 1, points: 2, objectID: 'z' },
      ],
    };

    it('renders without crashing', () => {
      const div = document.createElement('div');
      ReactDOM.render(<Table { ...props } />, div);
    });

    test('has a valid snapshot', () => {
      const component = renderer.create(
        <Table { ...props } />
      );
      let tree = component.toJSON();
      expect(tree).toMatchSnapshot();
    });
  });
  ```

* 可以使用[`Enzyme`](https://github.com/airbnb/enzyme)来编写单元测试。`create-react-app`并不包含它，所以需手动安装，并在测试文件中引入并配置

  ```bash
  npm install --save-dev enzyme react-addons-test-utils enzyme-adapter-react-16
  ```

  ```jsx
  import React from 'react';
  import ReactDOM from 'react-dom';
  import renderer from 'react-test-renderer';
  //
  import Enzyme from 'enzyme';
  import Adapter from 'enzyme-adapter-react-16';
  //
  import App, { Search, Button, Table } from './App';

  Enzyme.configure({ adapter: new Adapter() });
  ```

  可以使用`shallow()`/`mount()`/`render()`方法渲染组件，并编写相应的断言

  ```jsx
  describe('Table', () => {
    const props = {
      list: [
        { title: '1', author: '1', num_comments: 1, points: 2, objectID: 'y' },
        { title: '2', author: '2', num_comments: 1, points: 2, objectID: 'z' },
      ],
    };

    it('shows two items in list', () => {
      const element = shallow(
        <Table { ...props } />
      );
      expect(element.find('.table-row').length).toBe(2);
    });
  });
  ```

  `shallow()`仅执行浅渲染，即不会渲染组件的子组件；若需要渲染子组件，可使用`render()`；`mount()`方法同样会渲染子组件，并会执行组件的生命周期方法

* 可以使用[`PropTypes`](https://github.com/facebook/prop-types)来检查props的类型，同样需要手动安装并引入

  ```bash
  npm install prop-types
  ```

  ```jsx
  import PropTypes from 'prop-types';

  const Button = ({ onClick, className = '', children }) =>
    <button
      onClick={onClick}
      className={className}
      type="button"
  >
    {children}
  </button>

  Button.propTypes = {
    onClick: PropTypes.func,
    className: PropTypes.string,
    children: PropTypes.node,
  };
  ```

  可以验证的类型如下

  ```jsx
  // 基本类型
  PropTypes.array // 数组
  PropTypes.bool // 布尔
  PropTypes.func // 函数
  PropTypes.number // 数值
  PropTypes.string // 字符串
  PropTypes.object // 对象
  // 其他类型
  PropTypes.node // 一个可渲染的片段(节点)，如children
  PropTypes.element // 一个React元素
  ```

  默认仅验证参数的类型，还可以验证参数的必填性。默认所有参数都是可选传递，参数可以为`null`或者`undefined`，用`isRequired`可以将参数设为必填

  ```jsx
  Button.propTypes = {
    onClick: PropTypes.func.isRequired,
    className: PropTypes.string,
    children: PropTypes.node.isRequired,
  };
  ```

  可以明确定义数组中元素的类型

  ```jsx
  Table.propTypes = {
    list: PropTypes.arrayOf(
      PropTypes.shape({
        objectID: PropTypes.string.isRequired,
        author: PropTypes.string,
        url: PropTypes.string,
        num_comments: PropTypes.number,
        points: PropTypes.number,
      })
    ).isRequired,
    onDismiss: PropTypes.func.isRequired,
  };
  ```

* 可以使用DOM元素的`ref`属性获得元素自身

  ```jsx
  // ES6组件，通过ref属性获得元素后则可在生命周期方法中引用元素
  class Search extends Component {
    componentDidMount() {
      if (this.input) {
        this.input.focus();
      }
    }

    render() {
      const { value, onChange, onSubmit, children, } = this.props;

      return (
        <form onSubmit={onSubmit}>
          <input
          type="text"
          value={value}
          onChange={onChange}
          ref={(node) => { this.input = node; }} // 将函数的node参数赋值给this的input属性
        />
          <button type="submit">
            {children}
          </button>
        </form>
      );
    }
  }

  // 函数式组件，由于没有this和生命周期方法，则可以将元素赋值给局部变量

  const Search = ({ value, onChange, onSubmit, children, }) => {
    let input; // ref属性中会将node赋值给该局部变量

    return (
      <form onSubmit={onSubmit}>
        <input
          type="text"
          value={value}
          onChange={onChange}
          ref={(node) => input = node}
        />
        <button type="submit">
          {children}
        </button>
      </form>
    );
  }
  ```

* 高阶组件，类似高阶函数，即返回组件的组件，高阶组件常用`with`做前缀来命名

  ```jsx
  const withLoading = (Component) => ({ isLoading, ...rest }) =>
    isLoading
      ? <Loading />
      : <Component { ...rest } />

  const ButtonWithLoading = withLoading(Button);
  ```

  ```jsx
  class App extends Component {
    // ...
    render() {
      // ...
      return (
        <div className="page">
          // ...
          <div className="interactions">
            <ButtonWithLoading
              isLoading={isLoading}
              onClick={() => this.fetchSearchTopStories(searchKey, page + 1)}>
              More
            </ButtonWithLoading>
          </div>
        </div> 
      );
    }
  }
  ```

* 可以使用[`classnames`](https://github.com/JedWatson/classnames)来实现条件渲染DOM元素的className

  ```jsx
  import React, { Component } from 'react';
  import fetch from 'isomorphic-fetch';
  import { sortBy } from 'lodash';
  import classNames from 'classnames';
  import './App.css';

  const Sort = ({ sortKey, activeSortKey, onSort, children, }) => {
    const sortClass = classNames(
      'button-inline',
      { 'button-active': sortKey === activeSortKey }
    );
    return (
      <Button
        onClick={() => onSort(sortKey)}
        className={sortClass}
      >
        {children}
      </Button>
    );
  }
  ```

* `setState()`方法不仅可以接收对象，还可以传入一个函数来更新状态信息。由于`setState()`方法是异步执行的，若传入的对象依赖于`this.state`/`this.props`，很可能方法调用时该属性的值在别处由其他地方修改过，则结果很不稳定容易导致bug。但若传入函数，则通过回调函数能保持执行时的状态及属性。

  ```jsx
  this.setState((prevState, props) => {
    // ...
  });
  ```

