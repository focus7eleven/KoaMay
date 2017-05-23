#### 核心概念

- 核心是store
- store中的存储是响应式的
- 改变sotre中的状态的唯一途径是显式地 **commit mutations** 。
- 由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在**计算属性**中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutations。

