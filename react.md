# 相关链接
- 1. React:
http://reactjs.cn/react/docs/getting-started.html
http://todomvc.com/examples/react/#/

- 2. Redux:
http://redux.js.org/docs/introduction/Motivation.html

- 3. packages: react-router,  axios,  react-redux

- 4. ES6

- 5. sass／bootstrap

- 6. tools: gulp, webpack

# React

## 第一步：拆分用户界面为一个组件树
- 为所有组件（及子组件）命名并画上线框图。
- 单一功能原则，指的是，理想状态下一个组件应该只做一件事，假如它功能逐渐变大就需要被拆分成更小的子组件。

## 第二步： 利用 React ，创建应用的一个静态版本
- var与let的区别：作用域不同。var声明的变量是全局变量，let只作用于块级。
- 最顶层是一个名为ReactDOM的元素，调用你声明的组件
```
ReactDOM.render(
  <FilterableProductTable products={PRODUCTS} />,
  document.getElementById('container')
);
```

- 将你声明的组件一一实现（用var声明组件）
```
var FilterableProductTable = React.createClass({
  render: function() {
    return (
      <div>
        <SearchBar />
        <ProductTable products={this.props.products} />
      </div>
    );
  }
});
```

- 为了创建一个渲染数据模型的应用，你将会构造一些组件，这些组件重用其它组件，并且通过 props 传递数据。 props 是一种从父级向子级传递数据的方式。 state用于实现交互功能，也就是说，数据随着时间变化。

# 第三步：识别出最小的（但是完整的）代表 UI 的 state

- 1. 是否是从父级通过 props 传入的
- 2. 是否会随着时间改变
- 3. 能根据组件中其它 state 数据或者 props 计算出来吗
- 以上问题的答案如果是“是”，那么就不是state
- 定义state
```
var FilterableProductTable = React.createClass({
  getInitialState: function() {
    return {
      filterText: '',
      inStockOnly: false
    };
  },

  render: function() {
    return (
      <div>
        <SearchBar
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
        />
        <ProductTable
          products={this.props.products}
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
        />
      </div>
    );
  }
});
```

# 例子
- modules/Account/AccountSettingMain.jsx
```
import React, { Component, PropTypes } from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import { pushState } from 'redux-router';

import { setState } from './actions/accountSetting.js';

class AccountSettingMain extends Component {
  static propTypes = {
    children: PropTypes.object,
    setState: PropTypes.func.isRequired,
    pushState: PropTypes.func.isRequired,
    i18n: PropTypes.func.isRequired
  }

  constructor(props) {
    super(props);
  }
  
  // 初始化的时候调用
  componentDidMount = async () => {
  }
  
  render = () => {
    const { i18n } = this.props;
    return ();
  }
}

const MapStateToProps = (state) => {
  return {
    ...state.AccountSetting,
    i18n: state.i18n.get('app')
  };
}

const mapDispatchToProps = (dispatch) => {
  return {
    setState: bindActionCreators(setState, dispatch),
    pushState: bindActionCreators(pushState, dispatch)
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(AccountSettingMain);
```

- actions/accountSetting.js
```
import { ACCOUNT_SETTING_SET } from './ActionTypes.js';

export const setState = (state) => {
  return { type: ACCOUNT_SETTING_SET, payload: state };
}

```

- reducers/accountSetting.js
```
import { ACCOUNT_SETTING_SET } from './ActionTypes.js';

static defaultState = {

};

export default (state = defaultState, action) => {
  switch (action.type) {
    case ACCOUNT_SETTING_SET:
      return {...state, ...action.payload};
    default:
      return state;
  }
}
```
