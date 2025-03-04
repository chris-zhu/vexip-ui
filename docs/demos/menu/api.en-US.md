### Menu Props

| Name          | Type                                               | Description                                                                           | Default       | Since |
| ------------- | -------------------------------------------------- | ------------------------------------------------------------------------------ | ------------ | --- |
| active        | `string`                                           | 设置默认激活的菜单                                                             | `null`       | - |
| accordion     | `boolean`                                          | 设置是否开启手风琴模式，该模式下同级菜单只能展开一个                           | `0`          | - |
| marker-type   | `'top' \| 'right' \| 'bottom' \| 'left' \| 'none'` | 设置选中菜单的标记的类型                                                       | `'right'`    | - |
| reduced       | `boolean`                                          | 设置菜单是否搜索                                                               | `false`      | - |
| horizontal    | `boolean`                                          | 设置是否为横向菜单                                                             | `false`      | - |
| group-type    | `'collapse' \| 'dropdown'`                         | 在展开状态时子菜单的形式                                                       | `'collapse'` | - |
| theme         | `'light' \| 'dark'`                                | 设置菜单的主题                                                                 | `'light'`    | - |
| tooltip-theme | `'light' \| 'dark'`                                | 设置菜单气泡提示的主题                                                         | `'dark'`     | - |
| transfer      | `boolean \| string`                                | 设置其下 MenuItem 的 `transfer` 属性，当 MenuItem 单独设置了该属性时优先级更高 | `false`      | - |

### Menu Events

| Name   | Description                                                       | Parameters              | Since |
| ------ | ---------------------------------------------------------- | ----------------- | --- |
| select | 当菜单被选择时触发，返回被选菜单的 label                   | `(label: string)` | - |
| expand | 当菜单被展开组 (子菜单) 时触发，返回被展开组的菜单的 label | `(label: string)` | - |
| reduce | 当菜单被收起组 (子菜单) 时触发，返回被收起组的菜单的 label | `(label: string)` | - |

### MenuItem Props

| Name            | Type                | Description                                                                                    | Default  | Since |
| --------------- | ------------------- | --------------------------------------------------------------------------------------- | ------- | --- |
| label           | `string`            | 菜单的唯一标识                                                                          | `null`  | - |
| icon            | `string`            | 设置菜单的图标，菜单收缩状态的图标需通过该属性或同名插槽设置                            | `null`  | - |
| disabled        | `boolean`           | 设置菜单是否为禁用状态                                                                  | `false` | - |
| transfer        | `boolean \| string` | 当子元素处于下拉状态时，设置其子元素的渲染位置，设置为 `true` 时默认渲染至 `<body>`                             | `false` | - |
| transition-name | `string`            | 当子元素处于下拉状态时，设置子元素的过渡效果，未设置时会根据是否为横向菜单分别取值 `'vxp-drop'` 或 `'vxp-zoom'` | `null`  | - |

### MenuItem Slots

| Name    | Description                 | Parameters | Since |
| ------- | -------------------- | ---- | --- |
| default | 菜单的内容插槽       | -    | - |
| icon    | 菜单的图标内容插槽   | -    | - |
| group   | 菜单的子菜单内容插槽 | -    | - |

### MenuGroup Props

| Name  | Type     | Description       | Default | Since |
| ----- | -------- | ---------- | ------ | --- |
| label | `string` | 分组的名称 | `''`   | - |
