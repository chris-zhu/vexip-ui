### Tree Props

| Name          | Type              | Description                                                                                                                                                        | Default     | Since |
| ------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | --- |
| data          | `InitDataOptions[]`             | 树数据源，支持传入待构建的数组结构或者已处理的树结构                                                                                                    | `[]`         | - |
| arrow         | `'auto' \| boolean` | 设置树节点是否带有箭头指示，设置为 `'auto'` 时会根据节点是否有下级自动显示隐藏                                                                                | `'auto'`     | - |
| no-build-tree | `boolean`           | 设置是否禁用内置的构建树，当 `data` 的数据源为树形结构时设置                                                                                                  | `false`      | - |
| empty-tip     | `string`            | 数据为空时显示的提示                                                                                                                                        | `locale.empty` | - |
| disabled      | `boolean`           | 设置树是否为禁用状态，若设置后，所有的树节点将被禁用                                                                                                        | `false`      | - |
| readonly      | `boolean`           | 设置树是否为只读状态，若设置后，所有的树节点将为只读                                                                                                        | `false`      | - |
| checkbox      | `boolean`           | 设置是否开启节点的复选框                                                                                                                                    | `false`      | - |
| draggable     | `boolean`           | 设置节点是否可拖拽                                                                                                                                          | `false`      | - |
| renderer      | `(data: { data: Data, node: TreeNodeOptions, depth: number }) => any`          | 使用 render 函数进行节点渲染数                                                                                           | `null`       | - |
| id-key        | `string`            | 设置数据源的 `id` 字段                                                                                                                                        | `'id'`       | - |
| children-key  | `string`            | 设置数据源的 `children` 字段                                                                                                                                  | `'children'` | - |
| parent-key    | `string`            | 设置数据源的 `parent` 字段                                                                                                                                    | `'parent'`   | - |
| label-key     | `string`            | 设置数据源的 `label` 字段                                                                                                                                     | `'label'`    | - |
| multiple      | `boolean`           | 设置是否开启多选模式                                                                                                                                        | `false`      | - |
| indent        | `string \| number`  | 设置每层树节点的缩进距离                                                                                                                                    | `'1.2em'`    | - |
| accordion     | `boolean`           | 设置是否开启手风琴模式                                                                                                                                      | `false`      | - |
| appear        | `boolean`           | 设置树节点过渡效果的 `appear` 值                                                                                                                              | `false`      | - |
| floor-select  | `boolean`           | 开启后，当选择存在下级的节点时，会触发节点的展开收起，无下级时才会触发选择取消事件                                                                          | `false`      | - |
| async-load    | `(node: Readonly<TreeNodeOptions>) => void \| boolean \| Promise<any>`          | 节点初次加载触发的回调函数，接受 `node` 对象作为参数，如果返回 `false` 则表示加载失败，支持异步函数和 `Promise`                                                | `null`       | - |
| cache-node    | `boolean`           | 设置是否开启节点数据缓存机制，开启后当每次 `data` 发生变化时，同个对象引用或者 `id` 值相同的节点，除了 `id`、`parent`、`children` 和 `label` 属性外其余属性将不会被刷新 | `false`      | - |
| root-id       | `string \| number`  | 设置根节点的 `id` 值，设置后，当 parent 值与该值相等的节点，将作为第一级节点展示                                                                              | `null`       | - |

一些预设的类型：

```ts
type Data = Record<string, unknown>
enum DropType {
  BEFORE,
  INNER,
  AFTER
}

interface InitDataOptions extends Data {
  visible?: boolean,
  selected?: boolean,
  expanded?: boolean,
  disabled?: boolean,
  checked?: boolean,
  loading?: boolean,
  loaded?: boolean,
  readonly?: boolean,
  arrow?: boolean | 'auto' | null,
  checkbox?: boolean | null
}

type TreeNodeOptions = Required<InitDataOptions> & {
  data: InitDataOptions,
  partial: boolean
} & Data
```

### Tree Events

| Name           | Description                                                                                                                                                        | Parameters                             | Since |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- | --- |
| node-change | 当节点的复选框状态发生改变时触发，返回当前节点的数据和节点对象                                                                                              | `(data: InitDataOptions, node: TreeNodeOptions, checked: boolean)`                       | - |
| node-click | 当节点被点击时触发，返回当前节点数据和节点对象 | `(data: InitDataOptions, node: TreeNodeOptions)` | - |
| node-select | 当节点被选择时触发，返回当前节点的数据和节点对象，如果开启的多选模式，则返回的参数类型均为数组                                                                                                            | `(data: InitDataOptions \| InitDataOptions[], nodes: TreeNodeOptions \| TreeNodeOptions[])`                       | - |
| node-cancel | 当节点被取消选择时触发，返回当前节点的数据和节点对象                                                                                                        | `(data: InitDataOptions, node: TreeNodeOptions)`                       | - |
| node-expand | 当节点被展开时触发，返回当前节点的数据和节点对象                                                                                                            | `(data: InitDataOptions, node: TreeNodeOptions)`                       | - |
| node-reduce | 当节点被收起时触发，返回当前节点的数据和节点对象                                                                                                            | `(data: InitDataOptions, node: TreeNodeOptions)`                       | - |
| drag-start  | 当节点将要开始拖拽时触发，返回当前节点的数据和节点对象                                                                                                      | `(data: InitDataOptions, node: TreeNodeOptions)`                       | - |
| drag-over   | 当节点正在拖拽时触发，返回当前节点的数据和节点对象                                                                                                          | `(data: InitDataOptions, node: TreeNodeOptions)`                       | - |
| drop        | 当节点被其他的拖拽节点放入时触发，返回当前节点的数据和节点对象                                                                                              | `(data: InitDataOptions, node: TreeNodeOptions， dropType: DropType)`                       | - |
| drag-end    | 当节点结束拖拽时触发，返回当前节点的数据和节点对象                                                                                                          | `(data: InitDataOptions, node: TreeNodeOptions)`                       | - |

### Tree Slots

| Name  | Description                                                                                                                                                                                                     | Parameters | Since |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | --- |
| label | 节点标签的插槽                                                                                                                                                         | `(data: InitDataOptions, node: TreeNodeOptions, depth: number)` | - |
| node  | 节点内容的插槽，使用该插槽将完全覆盖节点各元素，包括箭头、复选框等，插槽额外的三个方法 `toggleCheck`、`toggleExpand`、`toggleSelect`，分别用于触发节点的勾选、展开、选择 | `(data: InitDataOptions, node: TreeNodeOptions, depath: number, toggleCheck: (checked?: boolean) => void, toggleExpand: (expanded?: boolean) => Promise<void>, toggleSelect: (able?: boolean) => Promise<void>)` | - |
| empty | 当数据为空时的提示文字的插槽                                                                                                                                                                             | - | - |

### Tree Methods

| Name                    | Description                                                                                                                                        | Signature              | Since |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | --- |
| parseAndTransformData | 触发组件内部重新进行数据的解析和转换 | `() => void` | -
| forceUpdateData              | 强制更新数据，一般在手动更改了数据源后更新树使用                                                                                                | `() => void`                | - |
| syncNodeStateIntoData   | 讲 node 中的状态属性同步到 data 中，将会覆盖 `visible`、`selected`、`expanded`、`disabled`、`checked`、`loading`、`readonly` 字段，谨慎使用 | `() => void`                 | - |
| getCheckedNodes         | 获取所有复选框被勾选的节点对象                                                                                                              | `() => TreeNodeOptions[]`                 | - |
| getCheckedNodeData      | 获取所有复选框被勾选的节点数据                                                                                                              | `() => InitDataOptions[]`                 | - |
| getSelectedNodes        | 获取所有被选择的节点对象                                                                                                                    | `() => TreeNodeOptions[]`                 | - |
| getSelectedNodeData     | 获取所有被选择的节点数据                                                                                                                    | `() => InitDataOptions[]`                 | - |
| getExpandedNodes        | 获取所有展开的节点对象                                                                                                                      | `() => TreeNodeOptions[]`                 | - |
| getDisabledNodes        | 获取所有被禁用的节点对象                                                                                                                    | `() => TreeNodeOptions[]`                 | - |
| getNodeChildren | 获取节点对象的子节点对象 | `(node: TreeNodeOptions) => TreeNodeOptions[]` | - |
| getParentNode           | 根据节点对象获取其父节点对象，不存在则返回 `null`                                                                                             | `(node: TreeNodeOptions) => TreeNodeOptions \| null`              | - |
| getSiblingNodes         | 根据节点对象获取所有兄弟节点对象，默认不包含自身                                                                                            | `(node: TreeNodeOptions, includeSelf: boolean) => TreeNodeOptions[]` | - |
| getPrevSiblingNode      | 根据节点对象获取上一个兄弟节点对象，不存在则返回 `null`                                                                                       | `(node: TreeNodeOptions) => TreeNodeOptions \| null`              | - |
| getNextSiblingNode      | 根据节点对象获取下一个兄弟节点对象，不存在则返回 `null`                                                                                       | `(node: TreeNodeOptions) => TreeNodeOptions \| null`              | - |
| getNodeByData           | 根据数据获取节点对象                                                                                                                        | `<T extends Data>(data: T) => TreeNodeOptions \| null`              | - |
| expandNodeByData        | 根据数据更改节点的展开状态                                                                                                                  | `<T extends Data>(data: T, expanded: boolean) => void`    | - |
| selectNodeByData        | 根据数据更改节点的选择状态                                                                                                                  | `<T extends Data>(data: T, selected: boolean) => void`    | - |
| checkNodeByData         | 根据数据更改节点的复选框被选状态                                                                                                            | `<T extends Data>(data: T, checked: boolean) => void`     | - |
| toggleNodeLoadingByData | 根据数据更改节点的加载状态                                                                                                                  | `<T extends Data>(data: T, loading: boolean) => void`     | - |

### TreeNode Props

> 以下属性在节点初始化时会从 data 的同名属性中获取初始值，若未定义则使用默认值，注意节点的刷新会触发该节点重新初始化。

| Name     | Type              | Description                                                                                               | Default  | Since |
| -------- | ----------------- | -------------------------------------------------------------------------------------------------- | ------- | --- |
| label    | `string`            | 节点显示的标签内容                                                                                 | `''`      | - |
| selected | `boolean`           | 设置节点的选择状态                                                                                 | `false`   | - |
| expanded | `boolean`           | 设置节点的展开状态                                                                                 | `false`   | - |
| disabled | `boolean`           | 设置节点的禁用状态，不设置时会使用 Tree 的同名状态                                                 | `false`   | - |
| readonly | `boolean`           | 设置节点的只读状态，不设置时会使用 Tree 的同名状态                                                 | `false`   | - |
| checkbox | `boolean`           | 设置节点是否具有复选框，不设置时会使用 Tree 的状态                                                 | `false`    | - |
| arrow    | `'auto' \| boolean` | 设置节点是否带有箭头指示，设置为 'auto' 时会根据是否有下级自动显示隐藏，不设置时会使用 Tree 的状态 | `'auto'`  | - |
| checked  | `boolean`           | 设置节点的复选框被选状态                                                                           | `false`   | - |
| loading | `boolean` | 设置节点是否处于加载中状态 | `false` | - |
| loaded | `boolean` | 设置节点是否已加载 | `false` | - |
