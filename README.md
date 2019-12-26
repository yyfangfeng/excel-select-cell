## 最近因公司需求
### 需要编写 Excel 合并单元格，拖拽选取单元格，拖拽改变单元格大小等功能

</br>

> 因公司使用框架是公司一位大佬写的

> 所以为了能够与公司项目有较好的黏合性，需要使用 `js` 或 `jquery` 开发

<br/>

> 而在网上基本找不到大佬分享的代码，更多的是叫别人使用插件

> 虽然有找到一位网友分享的代码，但是因为这位网友的代码放在很多 `td` 的 `table` 里会出现卡顿，性能较差，所以不采用

> 后来决定自己开发。。。

<br/>

## 示例

![img](https://file.fffsilly.top/image/ssrblog/html_201912261331.gif)

<br/>

[github地址](https://github.com/yyfangfeng/excel-select-cell)

<br/>

## 技术难点

* `table` 里 `td` 合并的问题
* 拖拽选择 `td` 时，如果碰到合并后的 `td` 还需要判断是否还有选择到其他 `td`

<br/>

## 实现思路

1. 先把 `table` 转成数组，合并的单元格需要按以下方式插入

```
// 以下 merge 代表此标签 “ <td colSpan="3" rowSpan="3">123</td> ”

let table_arr = [
    [td, td, td, td, td, td, ....],
    [td, td, td, td, td, td, ....],
    [td, td, merge, merge, merge, td, ....],
    [td, td, merge, merge, merge, td, ....],
    [td, td, merge, merge, merge, td, ....],
    [td, td, td, td, td, td, ....],
    ....
]
```

2. 给每个 `td` 遍历设置唯一标识，在这里我使用了 `dataset` 来设置

```
// 给 td 设置唯一标识
function setTdSpan (table_arr) {
    setTimeout(() => {
        table_arr.forEach((row, row_i) => {
            row.forEach((col, col_i) => {
                let row_num = row_i
                let col_num = col_i
                if (col.rowSpan > 1) row_num = row_i - col.rowSpan + 1
                if (col.colSpan > 1) col_num = col_i - col.colSpan + 1
                $(col).attr('data-tdspan', `${row_num}-${col_num}`)
                $(col).attr('data-endtdspan', `${row_i}-${col_i}`)
            })
        })
    })
}
```

3. 然后根据鼠标按下，移动，弹起位置，获取到单元格 `td` 的下标

4. 然后就计算某个下标 `td` 到某个下标 `td`，计算出的选择框大小、位置

5. 最后给选择框设置 `css` 属性

<br/>

## 结束

> 最后欢迎大家来指出问题，一起学习

> 觉得不错的也可以给俺一个 `star`，爱你兄弟 ^▽^