## 最近因公司需求
### 需要编写 Excel 合并单元格，拖拽选取单元格，拖拽改变单元格大小等功能

</br>

因公司使用框架是公司一位大佬写的

所以为了能够与公司项目有较好的黏合性，需要使用 `js` 或 `jquery` 开发

<br/>

而在网上基本找不到大佬分享的代码，更多的是叫别人使用插件

虽然有找到一位网友分享的代码，但是因为这位网友的代码放在很多 `td` 的 `table` 里会出现卡顿，性能较差，所以不采用

后来决定自己开发。。。

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

```javascript
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

```javascript
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

```javascript
// 获取鼠标位置的 td
function switchTd (params, fn) {
    let table = $('table')[0]
    let { x, y } = params

    table_arr.forEach((tr, tr_ind) => {
        tr.forEach((td, td_ind) => {
            if (tr_ind === 0 || td_ind === 0) return

            let td_start_left = td.offsetLeft
            let td_start_top = td.offsetTop

            let td_end_left = td_start_left + td.offsetWidth
            let td_end_top = td_start_top + td.offsetHeight

            if ((x >= td_start_left && x <= td_end_left) && (y >= td_start_top && y <= td_end_top)) {
                typeof fn === 'function' && fn({
                    tr_ind, td_ind, td,
                    end_tr_ind: subStrTdSpan(td.dataset.endtdspan, 'tr'),
                    end_td_ind: subStrTdSpan(td.dataset.endtdspan, 'td'),
                    left: table.offsetLeft + td_start_left,
                    top: table.offsetTop + td_start_top
                })
            }
        })
    })
}
```

```javascript
// 拖拽选取
$(document).on('mousedown', (e) => {
    $('.select_area').css({'cursor': 'default'})
    let start_left = e.clientX + $(document).scrollLeft()
    let start_top = e.clientY + $(document).scrollTop()
    // 用于判断是否相同的 td
    let tr_td = ''
    // 起止 td 下标
    let tr_num = 0
    let td_num = 0

    switchTd({
        x: start_left - $('table').offset().left,
        y: start_top - $('table').offset().top
    }, (ind_data) => {
        tr_num = ind_data.tr_ind
        td_num = ind_data.td_ind
        if (tr_num === 0 || td_num === 0) return
        collisionCell({
            tr_num, td_num,
            tr_ind: ind_data.end_tr_ind,
            td_ind: ind_data.end_td_ind
        })
        tr_td = ind_data.td.dataset.tdspan
    })
    $(document).on('mousemove', (e1) => {
        e1.preventDefault()

        let move_left = e1.clientX + $(document).scrollLeft()
        let move_top = e1.clientY + $(document).scrollTop()

        switchTd({
            x: move_left - $('table').offset().left,
            y: move_top - $('table').offset().top
        }, (ind_data) => {
            if (tr_num === 0 || td_num === 0) return
            if (tr_td === ind_data.td.dataset.tdspan) return
            collisionCell({
                tr_num, td_num,
                tr_ind: ind_data.end_tr_ind,
                td_ind: ind_data.end_td_ind
            })
            tr_td = ind_data.td.dataset.tdspan
        })
    })
})
$(document).on('mouseup', () => {
    $(document).unbind('mousemove')
    setSelectTdArr()
    console.log(start_end_data)
    console.log(select_td_arr)
})
```

4. 然后就计算某个下标 `td` 到某个下标 `td`，计算出的选择框大小、位置

```javascript
// 碰撞机制 1，计算起止位置
function collisionCell (params) {
    let { tr_num, td_num, tr_ind, td_ind } = params

    // 判断起始位置，终点位置
    let start_tr_ind = tr_num <= tr_ind? tr_num: tr_ind
    let end_tr_ind = start_tr_ind === tr_num? tr_ind: tr_num
    let start_td_ind = td_num <= td_ind? td_num: td_ind
    let end_td_ind = start_td_ind === td_num? td_ind: td_num

    start_end_data = {
        s_tr: start_tr_ind, s_td: start_td_ind,
        e_tr: end_tr_ind, e_td: end_td_ind
    }

    comElementTdSpan({
        s_tr: start_tr_ind, s_td: start_td_ind,
        e_tr: end_tr_ind, e_td: end_td_ind
    })
}
```

```javascript
// 碰撞机制 2，计算碰撞机制后的起止下标
let select_td_arr2 = [] // 用于选择时判断
function comElementTdSpan (params) {
    let { s_tr, s_td, e_tr, e_td } = params
    let std = table_arr[s_tr][s_td]
    let etd = table_arr[e_tr][e_td]
    let start_tr_ind = subStrTdSpan(std.dataset.tdspan, 'tr')
    let start_td_ind = subStrTdSpan(std.dataset.tdspan, 'td')
    let end_tr_ind = subStrTdSpan(etd.dataset.endtdspan, 'tr')
    let end_td_ind = subStrTdSpan(etd.dataset.endtdspan, 'td')
    select_td_arr2 = []
    
    let left = null, top = null, right = null, bottom = null
    for (let tr_ind = start_tr_ind; tr_ind <= end_tr_ind; tr_ind++) {
        for (let td_ind = start_td_ind; td_ind <= end_td_ind; td_ind++) {
            let td = table_arr[tr_ind][td_ind]
            select_td_arr2.push(td)
            
            let left_td = td.getBoundingClientRect().left
            let top_td = td.getBoundingClientRect().top
            let right_td = td.getBoundingClientRect().right
            let bottom_td = td.getBoundingClientRect().bottom

            left = left !== null ? (left_td < left ? left_td : left) : left_td
            top = top !== null ? (top_td < top ? top_td : top) : top_td
            right = right !== null ? (right_td > right ? right_td : right) : right_td
            bottom = bottom !== null ? (bottom_td > bottom ? bottom_td : bottom) : bottom_td
        }
    }
    left = left + $(document).scrollLeft()
    top = top + $(document).scrollTop()
    right = right + $(document).scrollLeft()
    bottom = bottom + $(document).scrollTop()

    let s_tr2 = null, s_td2 = null, e_tr2 = null, e_td2 = null
    select_td_arr2.forEach((td) => {
        let s_tr_ind2 = subStrTdSpan(td.dataset.tdspan, 'tr')
        let s_td_ind2 = subStrTdSpan(td.dataset.tdspan, 'td')
        let e_tr_ind2 = subStrTdSpan(td.dataset.endtdspan, 'tr')
        let e_td_ind2 = subStrTdSpan(td.dataset.endtdspan, 'td')

        s_tr2 = s_tr2 !== null ? (s_tr2 < s_tr_ind2 ? s_tr2 : s_tr_ind2) : s_tr_ind2
        s_td2 = s_td2 !== null ? (s_td2 < s_td_ind2 ? s_td2 : s_td_ind2) : s_td_ind2
        e_tr2 = e_tr2 !== null ? (e_tr2 > e_tr_ind2 ? e_tr2 : e_tr_ind2) : e_tr_ind2
        e_td2 = e_td2 !== null ? (e_td2 > e_td_ind2 ? e_td2 : e_td_ind2) : e_td_ind2
    })
    setSelectCss({
        left: left,
        top: top,
        width: right - left,
        height: bottom - top
    })
    if (s_tr !== s_tr2 || s_td !== s_td2 || e_tr !== e_tr2 || e_td !== e_td2) {
        start_end_data = {
            s_tr: s_tr2, s_td: s_td2,
            e_tr: e_tr2, e_td: e_td2
        }
        comElementTdSpan({
            s_tr: s_tr2, s_td: s_td2,
            e_tr: e_tr2, e_td: e_td2
        })
    }
}
```

5. 然后给选择框设置 `css` 属性

```javascript
// 设置选取框的样式
let sel_html = `<div class="select_area"></div>`
$('body').append($(sel_html))
function setSelectCss(params) {
    let { width, height, left, top, css_obj } = params
    $('.select_area').css(Object.assign({
        'width': width + 'px',
        'height': height + 'px',
        'left': left + 'px',
        'top': top + 'px',
        'border': '2px solid rgb(47, 137, 220)',
        'box-sizing': 'border-box',
        'position': 'absolute',
        'background-color': 'rgba(23, 133, 231, 0.1)'
    }, css_obj))
}
```

6. 最后把被选择框框选的 `td` 放入数组

```javascript
// 把选取的 td 插入 select_td_arr 数组里
function setSelectTdArr () {
    let select_area = $('.select_area')
    select_td_arr = []
    $('table').find('td').each(function () {
        let t1 = $(this).offset().top
        let l1 = $(this).offset().left
        let r1 = $(this).offset().left + $(this).innerWidth()
        let b1 = $(this).offset().top + $(this).innerHeight()

        let t2 = select_area.offset().top
        let l2 = select_area.offset().left
        let r2 = select_area.offset().left + select_area.innerWidth()
        let b2 = select_area.offset().top + select_area.innerHeight()

        if (t2 < b1 && l2 < r1 && r2 > l1 && b2 > t1) {
            select_td_arr.push($(this)[0])
        }
    })
}
```

<br/>

> 完整代码可到 [github](https://github.com/yyfangfeng/excel-select-cell) 里查看

<br/>

## 结束

最后欢迎大家来指出问题，一起学习

觉得不错的也可以给俺一个 `star`，爱你兄弟 ^▽^