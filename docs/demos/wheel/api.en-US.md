### Wheel Props

| Name      | Type    | Description                                          | Default       | Since |
| --------- | ------- | --------------------------------------------- | ------------ | --- |
| horizontal      | `boolean`  | 设置滚轮的是否为横向模式 | `false` | - |
| value    | `string \| number`  | 当前激活元素的索引，可以使用 v-model 双向绑定 | `0`            | - |
| candidate | `number`  | 设置滚轮上下的候选个数，可选范围为 0 ~ 3      | `2`            | - |
| arrow     | `boolean` | 设置是否使用滚轮的箭头指示器                  | `false`        | - |
| disable-validate | `boolean`                           | 是否禁用触发表单字段验证                                                         | `false`                 | - |

### Wheel Events

| Name      | Description                                               | Parameters          | Since |
| --------- | -------------------------------------------------- | ------------- | --- |
| change | 当前激活的元素发生改变时触发，返回该元素的索引和值 | `(value: string \| number)` | - |

### WheelItem Props

| Name  | Type                        | Description         | Default | Since |
| ----- | --------------------------- | ------------ | ------ | --- |
| value | `number \| string` | 滚轮元素的值 | `null`   | - |
| disabled | `boolean` | 设置是否禁用元素 | `false`   | - |
