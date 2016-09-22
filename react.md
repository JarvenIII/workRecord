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

- action/import.js
```
import * as RestUtil from '../../../utils/RestUtil';
import {
  API_GUESTS,
  API_IMPORT_GUESTS,
  API_EXPORT_GUESTS,
  API_CANCEL_IMPORT_GUESTS,
  API_IGNORE_IMPORT_PROGRESS,
  API_IMPORT_PROGRESS,
  API_EXPORT_PROGRESS,
  API_CANCEL_EXPORT,
  API_IGNORE_EXPORT_GUESET
} from '../../../constants/API';
import { API_FETCH_CHAINS_RESTAURANTS } from '../constants/API';
import { GUEST_GENDER, GUEST_TYPE } from '../../../constants/Guest';
import { IMPORTING_STATUS } from '../constants/Enum';
import { LIST, IMPORT, EXPORT } from '../constants/Action';
import _ from 'lodash';

// 暴露出一个公共的获取客户列表的方法
const fetchGuests = async (fetchCondition = {}, dispatch) => {
  const { searchTerm: searchWord, searchTermKey: searchBy, tagIds, sortBy,
    sortDirection, page, perPage, restaurantId, searchGuestType: typeId,
    searchGuestGender: gender } = fetchCondition;
  const params = { searchWord, searchBy, tagIds, sortBy, sortDirection,
      page, perPage, restaurantId, typeId, gender };
  let fetchResult = await RestUtil.get(API_GUESTS, { params });

  const defaultResult = {
    items: [],
    _links: {},
    _meta: {
      totalCount: fetchResult.items.length,
      pageCount: 1,
      currentPage: 1,
      perPage: 20
    }
  };

  fetchResult = Object.assign(defaultResult, fetchResult);

  // Assign default values for missing fields.
  // TODO; May be removed after integration.
  for (let i = 0; i < fetchResult.items.length; i++) {
    const defaultInfo = {
      _id: '1',
      firstName: 'b',
      lastName: '',
      phone: '不知道',
      gender: GUEST_GENDER.MALE,
      birthday: new Date(),
      email: 'example@example.com',
      type: GUEST_TYPE.VIP,
      systemTagIds: [],
      customTagIds: [],
      reserveCount: 10,
      lastSeatedAt: new Date(),
      createdAt: new Date()
    };

    fetchResult.items[i] = Object.assign({}, defaultInfo, fetchResult.items[i]);
  }

  dispatch({ type: LIST.FINISH_FETCH, fetchResult });
};

const fetch = (fetchCondition = {}) => {
  return async (dispatch) => {
    fetchGuests(fetchCondition, dispatch);
  };
};

const startFetch = () => {
  return (dispatch) => {
    dispatch({ type: LIST.START_FETCH });
  };
};

const changeSearchTerm = (newSearchTerm) => {
  return (dispatch) => {
    dispatch({ type: LIST.CHANGE_SEARCH_TERM, newSearchTerm });
  };
};

const selectTag = (tag) => {
  return (dispatch) => {
    dispatch({ type: LIST.SELECT_TAG, tag });
  };
};

const changeSort = (newSort) => {
  return (dispatch) => {
    dispatch({ type: LIST.CHANGE_SORT, newSort });
  };
};

const changePage = (newPage) => {
  return (dispatch) => {
    dispatch({ type: LIST.CHANGE_PAGE, newPage });
  };
};

const openDelete = (guestId) => {
  return (dispatch) => {
    dispatch({ type: LIST.OPEN_DELETE, guestId });
  };
};

const openGuestDetail = () => {
  return async (dispatch) => {
    dispatch({ type: LIST.OPEN_GUEST_DETAIL });
  };
};

const closeGuestDetail = () => {
  return async (dispatch) => {
    dispatch({ type: LIST.CLOSE_GUEST_DETAIL });
  };
};

const toggleImportGuests = () => {
  return (dispatch) => {
    dispatch({ type: IMPORT.TOGGLE_IMPORT_POPUP });
  };
};

const toggleExportGuests = () => {
  return (dispatch) => {
    dispatch({ type: EXPORT.TOGGLE_EXPORT_POPUP });
  };
};

const exportGuests = (condition) => {
  const { searchTerm: searchWord, searchTermKey: searchBy, tagIds, sortBy,
    sortDirection, restaurantId, searchGuestType: typeId,
    searchGuestGender: gender } = condition;
  const params = { searchWord, searchBy, tagIds, sortBy, sortDirection,
      restaurantId, typeId, gender };

  return async () => {
    await RestUtil.post(API_EXPORT_GUESTS, params);
  };
};

const importGuests = (fileUrl, shouldOverwrite) => {
  return async (dispatch) => {
    await RestUtil.post(API_IMPORT_GUESTS, { fileUrl, shouldOverwrite });
    dispatch({ type: IMPORT.FETCH_IMPORT_PROGRESS, importingProgress: { status: IMPORTING_STATUS.PARSING } });
  };
};

const cancelImport = () => {
  return async (dispatch) => {
    await RestUtil.post(API_CANCEL_IMPORT_GUESTS);
    dispatch({
      type: IMPORT.FETCH_IMPORT_PROGRESS,
      importingProgress: { status: IMPORTING_STATUS.IMPORT_CANCEL, total: 0, succeeded: 0 }
    });
  };
};

const ignoreImportProgress = () => {
  return async (dispatch) => {
    await RestUtil.post(API_IGNORE_IMPORT_PROGRESS);
    dispatch({
      type: IMPORT.FETCH_IMPORT_PROGRESS,
      importingProgress: { status: IMPORTING_STATUS.IMPORT_CANCEL, total: 0, succeeded: 0 }
    });
  };
};

const fetchImportProgress = () => {
  return async (dispatch) => {
    let importingProgress = {
      _id: 1,
      status: 0,
      total: 0,
      succeeded: 0,
      failed: 0,
      failedRecordsUrl: ''
    };

    try {
      const importingProgressTemp = await RestUtil.get(API_IMPORT_PROGRESS);
      if (importingProgressTemp) {
        importingProgress = importingProgressTemp;
      }
    } catch (error) {
      console.log('Get import progress failed!');
    }

    dispatch({ type: IMPORT.FETCH_IMPORT_PROGRESS, importingProgress });
  };
};

const fetchExportProgress = () => {
  return async (dispatch) => {
    const exportingPrograss = await RestUtil.get(API_EXPORT_PROGRESS);
    dispatch({
      type: EXPORT.FETCH_EXPORT_PROGRESS,
      exportingPrograss: exportingPrograss || { status: IMPORTING_STATUS.PARSING, total: 0, succeeded: 0 }
    });
  };
};

const cancelExport = () => {
  return async (dispatch) => {
    await RestUtil.post(API_CANCEL_EXPORT);
    dispatch({
      type: EXPORT.FETCH_EXPORT_PROGRESS,
      exportingPrograss: { status: IMPORTING_STATUS.IMPORT_CANCEL, total: 0, succeeded: 0 }
    });
  };
};

const ignoreExport = () => {
  return async (dispatch) => {
    await RestUtil.post(API_IGNORE_EXPORT_GUESET);
    dispatch({
      type: EXPORT.FETCH_EXPORT_PROGRESS,
      exportingPrograss: { status: IMPORTING_STATUS.IMPORT_CANCEL, total: 0, succeeded: 0 }
    });
  };
};

const clearFetchCondition = () => {
  return { type: LIST.CLEAR_FETCH_CONDITION };
};

const setFetchCondition = (fetchCondition = {}) => {
  return { type: LIST.SET_FETCH_CONDITION, fetchCondition };
};

const fetchChainsRestaurants = (restaurant) => {
  return async (dispatch) => {
    let restaurantId = restaurant._id;
    let checkedRestaurant;
    const restaurants = (await RestUtil.get(API_FETCH_CHAINS_RESTAURANTS) || {}).items || [];
    let isUncheckedRestaurant = true;
    for (const restaurantItem of restaurants) {
      if (restaurantId === restaurantItem._id) {
        checkedRestaurant = { ...restaurantItem };
        isUncheckedRestaurant = false;
      }
    }
    if (isUncheckedRestaurant && !_.isEmpty(restaurants)) {
      checkedRestaurant = { ...restaurants[0] };
      restaurantId = restaurants[0]._id;
    }

    dispatch({ type: LIST.FETCH_CHAINS_RESTAURANTS, restaurants, restaurantId, checkedRestaurant });
  };
};

const selectChainsRestaurant = (checkedRestaurant) => {
  return { type: LIST.SELECT_CHAINS_RESTAURANT, checkedRestaurant };
};

const setState = (state) => {
  return { type: LIST.GUESTS_SET_STATE, payload: state };
};

export {
  startFetch,
  fetch,
  changeSearchTerm,
  selectTag,
  changeSort,
  changePage,
  openDelete,
  openGuestDetail,
  closeGuestDetail,
  toggleImportGuests,
  toggleExportGuests,
  importGuests,
  cancelImport,
  ignoreImportProgress,
  fetchImportProgress,
  cancelExport,
  ignoreExport,
  fetchExportProgress,
  exportGuests,
  fetchGuests,
  clearFetchCondition,
  setFetchCondition,
  fetchChainsRestaurants,
  selectChainsRestaurant,
  setState
};

```

- reducer/import.js
```
import _ from 'lodash';

import { LIST, DELETE, IMPORT, EXPORT } from '../constants/Action';
import { IMPORTING_STATUS, GUEST_SEARCH_KEYS } from '../constants/Enum';

const defaultCondition = { fetchCondition: { tagIds: [], page: 1, perPage: 20, searchTermKey: GUEST_SEARCH_KEYS[0] } };
const defaultState = { ...defaultCondition, isExpandTagsPane: false };

export default (state = defaultState, action) => {
  let newState = {};
  switch (action.type) {
    case LIST.START_FETCH:
      newState = { shouldFetch: true, lastFetchCondition: { ...state.fetchCondition } };
      break;
    case LIST.CLEAR_FETCH_CONDITION:
      newState = _.cloneDeep(defaultCondition);
      break;
    case LIST.SET_FETCH_CONDITION:
      newState = { fetchCondition: { ...state.fetchCondition, ...action.fetchCondition } };
      break;
    case LIST.FINISH_FETCH:
      newState = { shouldFetch: false, fetchResult: action.fetchResult };
      break;
    case LIST.FETCH_CHAINS_RESTAURANTS:
      newState = {
        shouldFetch: true,
        restaurants: action.restaurants,
        checkedRestaurant: action.checkedRestaurant,
        fetchCondition: { ...state.fetchCondition, restaurantId: action.restaurantId }
      };
      break;
    case LIST.SELECT_CHAINS_RESTAURANT:
      newState = {
        checkedRestaurant: { ...action.checkedRestaurant },
        fetchCondition: { ...state.fetchCondition, restaurantId: action.checkedRestaurant._id }
      };
      break;
    case LIST.CHANGE_SEARCH_TERM:
      newState = { fetchCondition: { ...state.fetchCondition, searchTerm: action.newSearchTerm } };
      break;
    case LIST.SELECT_TAG:
      let tagIds = state.fetchCondition.tagIds;
      if (tagIds.includes(action.tag._id)) {
        tagIds = tagIds.filter(tagId => tagId !== action.tag._id);
      } else {
        tagIds = tagIds.concat([action.tag._id]);
      }
      newState = { fetchCondition: { ...state.fetchCondition, tagIds } };
      break;
    case LIST.CHANGE_SORT:
      newState = { fetchCondition: { ...state.fetchCondition, ...action.newSort, page: 1 } };
      break;
    case LIST.CHANGE_PAGE:
      newState = { fetchCondition: { ...state.fetchCondition, page: action.newPage } };
      break;
    case LIST.OPEN_GUEST_DETAIL:
      newState = { guestDetail: { isOpen: true } };
      break;
    case LIST.CLOSE_GUEST_DETAIL:
      newState = {
        guestDetail: {
          isOpen: false
        }
      };
      break;
    case LIST.OPEN_DELETE:
      newState = {
        showDeleteConfirm: true
      };
      break;
    case DELETE.SUCCESS_DELETE:
    case DELETE.FAIL_DELETE:
      newState = {
        showDeleteConfirm: false,
        shouldFetch: true
      };
      break;
    case DELETE.CANCEL_DELETE:
      newState = {
        showDeleteConfirm: false
      };
      break;
    case IMPORT.TOGGLE_IMPORT_POPUP:
      newState = { isShowImportGuests: !state.isShowImportGuests };
      break;
    case IMPORT.RECEIVE_GUESTS_IMPORTING_PROGRESS_FROM_TUISONGBAO:
      newState = { importingProgress: action.guests };

      if (state.importingProgress && state.importingProgress.status === IMPORTING_STATUS.IMPORT_CANCEL) {
        newState = {};
      }
      break;
    case IMPORT.FETCH_IMPORT_PROGRESS:
      newState = { importingProgress: action.importingProgress };
      break;
    case EXPORT.FETCH_EXPORT_PROGRESS:
      newState = { exportingPrograss: action.exportingPrograss };
      break;
    case EXPORT.TOGGLE_EXPORT_POPUP:
      newState = { isShowExportGuests: !state.isShowExportGuests };
      break;
    case EXPORT.RECEIVE_GUESTS_EXPORTING_PROGRESS_FROM_TUISONGBAO:
      newState = { exportingPrograss: action.guests };

      if (state.exportingPrograss && state.exportingPrograss.status === IMPORTING_STATUS.IMPORT_CANCEL) {
        newState = { exportingPrograss: {} };
      }
      break;
    case LIST.GUESTS_SET_STATE:
      newState = { ...action.payload };
      break;
    default:
      break;
  }

  return { ...state, ...newState };
};

```
