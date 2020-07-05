---
title: react+ts踩坑记0：前言
date: 2020/07/05 13:35:00
updated: 2020/07/05 13:35:00
categories: 
- react+ts踩坑记
tags: 
- react
- typescript
---

# 简介
已经研究了一年左右react，目前时机成熟，知识储备足够，我们决定正式在生产环境使用react+ts，期间遇到了一些只在生产环境需要考虑的问题，也踩了许多坑，这个系列会持续更新，记录react在针对生产环境开发和部署时遇到的实际问题、分析过程和解决方案。

**本系列文章适合有webpack基础、对前端架构设计有一定了解、写过react+ts代码的前端工程师阅读，不适合萌新和不熟悉ts的同学。**

写这个系列的主要目的是分享解决问题的思路和方案，而不是教程，要读懂这些文章，你必须：
- 了解webpack配置和前端工程化
- 理解为什么要使用文中所提到的技术和工具
- 写过生产环境中的代码
- 理解typescript

# 技术栈
我们在生产环境运行的项目主要涉及以下技术：
- react 16.8+ 支持hooks即可
- typescript 3.8+ 高高益善
- react-router-dom 5.0+ 新版更优雅
- redux 4+, react-redux 状态管理
- rxjs 6+, redux-observable 适合我们的异步中间件
- react-transition-group 满足简单的动画需求
- @loadable/component 按需加载组件
- axios 考虑到团队，虽然已经有了rxjs，依然保留axios
- eslint, prettier, husky 开发规范
- less, postcss 处理样式
- jest 测试框架

还有一些辅助工具：
- js-cookie cookie处理都靠它
- lodash-es 老牌工具库
- qs 虽然node内置的querystring模块也能用，但qs显然更好用
- moment 时间处理
- classnames 方便地处理classname
- typesafe-actions 优雅地构造状态管理的action
- utility-types 好用的ts类型扩展工具

# 参考资料
react本身并不难，但是配合typescript使用后，会遇到一些奇怪的问题，这里有一些参考资料，能给react+ts提供思路：
- [react-typescript-cheatsheet](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet)，提供许多demo代码
- [react-redux-typescript-guide](https://github.com/piotrwitek/react-redux-typescript-guide)，解决90%的react相关的类型困惑

# 项目的结构和选型
## 初始脚手架
项目基于 `create-react-app` 生成，我们有维护webpack配置的能力，并且希望最大限度地自定义打包过程，因此用 `eject` 弹出全部配置，目录结构基本维持原状。

## 路由与鉴权
我们基于 `react-router-dom` 进行了大刀阔斧的改造，这里不提供细节和实现方法。由于团队之前一直使用vue，更习惯于vue的路由配置，改造后的路由写法与vue基本相同，类似这样：
```typescript
export const routes: RouteInterface[] = [
  {
    path: '/',
    exact: true,
    component: loadable(() => import('@/pages/index/index')),
    name: 'Index',
    meta: {
      title: '首页',
    },
  },
  {
    path: '/login',
    component: loadable(() => import('@/components/Login/index')),
    exact: true,
    name: 'Login',
  },
  {
    path: '/a',
    component: RouteDemoA,
    name: 'demoA',
    meta: {
      title: 'DemoA',
    },
    routes: [
      {
        path: '/a/b',
        component: RouteDemoB,
        exact: true,
        name: 'demoB',
        meta: {
          title: 'DemoB',
          auth: true,
        },
      },
    ],
  },
  // module routes
  ...moduleRoute,
  // 404 Not Found
  {
    path: '*',
    component: loadable(() => import('@/pages/status/404')),
    name: '404',
    meta: {
      title: '404',
    },
  },
]
```

## 状态管理
我们的应用重状态管理，hooks不能完全满足需求，在不使用装饰器的情况下，`redux` 是最适合我们的选择。redux的目录结构采用 `ducks` 方案，比较贴近我们之前在vuex中的设计，类似这样：

```javascript
// vuex
import { USER_AVATAR_DEFAULT } from '@/config'
import UserService from '@/services/user.service'

const moduleName = 'USER'

export const types = {
  mutations: {
    SET_USER_INFO: `${moduleName}/SET_USER_INFO`,
  },
  actions: {
    GET_USER_INFO: `${moduleName}/GET_USER_INFO`,
  }
}

const initState = {
  userInfo: {},
}

const mutations = {
  [types.mutations.SET_USER_INFO] (state, userInfo) {
    state.userInfo = userInfo
  },
}

const actions = {
  async [types.actions.GET_USER_INFO] ({ commit }) {
    const { data } = await UserService.getUserInfo()
    const userInfo = {
      userId: data.member_id,
      nickname: data.nickname,
      avatar: data.avatar || USER_AVATAR_DEFAULT,
    }
    commit(types.mutations.SET_USER_INFO, userInfo)
  },
}

export default {
  state: initState,
  mutations,
  actions
}
```

```typescript
import { DeepReadonly } from 'utility-types'
import { ActionType, createAction } from 'typesafe-actions'
// types
export const SET_USER = 'user/SET_USER'
export const FETCH_USER_REQUEST = 'user/FETCH_USER_REQUEST'
export const FETCH_USER_FAILURE = 'user/FETCH_USER_FAILURE'

export type SET_USER = typeof SET_USER
export type FETCH_USER_REQUEST = typeof FETCH_USER_REQUEST
export type FETCH_USER_FAILURE = typeof FETCH_USER_FAILURE
// state
export type UserState = DeepReadonly<{
  userId: string
  nickname: string
  avatar: string
}>

export interface User {
  userId: string
  nickname: string
  avatar: string
}

const initialState: UserState = {
  userId: '',
  nickname: '',
  avatar: '',
}
// action
export const actions = {
  [FETCH_USER_REQUEST]: createAction(FETCH_USER_REQUEST)(),
  [FETCH_USER_FAILURE]: createAction(FETCH_USER_FAILURE)(),
  [SET_USER]: createAction(SET_USER, (user: User) => ({ user }))(),
}

export type SetUser = ActionType<typeof actions[SET_USER]>
export type FetchUserRequest = ActionType<typeof actions[FETCH_USER_REQUEST]>
export type FetchUserFailure = ActionType<typeof actions[FETCH_USER_FAILURE]>
export type UserAction = ActionType<typeof actions>
// reducer
export function user(state = initialState, action: UserAction): UserState {
  switch (action.type) {
    case SET_USER:
      return {
        ...state,
        ...action.payload.user,
      }
    default:
      return state
  }
}
```

关于异步方案的选型，我们的业务涉及非常复杂的数据流程，`redux-saga`的写法怪异，因此我们选择 `rxjs` 作为异步方案，`redux-observable` 是个现成的可使用redux异步中间件。
