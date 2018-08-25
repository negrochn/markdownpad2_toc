# MarkdownPad2导出HTML支持[TOC]

## 为什么&效果图

### 为什么

为什么要吃力不讨好的实现这个功能呢？
- [TOC]语法挺有用的，特别是标题较多的时候
- 百度上并未找到MarkdownPad2导出HTML支持[TOC]的完美方案
- 顺便锻炼一下JavaScript

### 效果图

想要实现什么样的效果呢？No picture, no bibi.

![impression drawing](https://github.com/02954/markdownpad2_toc/blob/master/images/impressionDrawing.png)

最后测试用的Markdown语法栗子内容如下：
```
# 《Vue2.0开发去哪儿网App》知识点
[TOC]
## 深入理解Vue组件
### 4-8 动态组件与v-once指令
#### 动态组件
#### v-once指令
## Vue中的动画特效
### 5-1 Vue动画 - Vue中的CSS动画原理
#### 动画原理
### 5-2 在Vue中使用animate.css库
#### keyframes动画
#### 自定义类名
#### animate.css库
```

## 动手实现

开始动手实现吧~

### 实现的HTML效果

等等，莫急，方案都没想好，怎么编码...

例如，Markdown语法如下
```
# 一
[TOC]
## 二
### 三
## 二
```
我们期望展示的效果如下图，且发现Markdown语法中的`#`、`##`等标题最终转换为HTML中的`<h1>`、`<h2>`等元素。

`[TOC]`转换为`<p>[TOC]</p>`，显然`<p>[TOC]</p>`无法满足我们的期望，所以这里就是我们需要编码实现的地方。

![HTML Dom](https://github.com/02954/markdownpad2_toc/blob/master/images/htmlDom.png)

那么，我们要将`<p>[TOC]</p>`，转换成什么样子呢？
```
<p>
    <div class="toc">
        <ul data-level="1">
            <li data-level="1">
                <a href="#一-0">一</a>
                <ul data-level="2">
                    <li data-level="2">
                        <a href="#二-1">二</a>
                        <ul data-level="3">
                            <li data-level="3">
                                <a href="#三-2">三</a>
                            </li>
                        </ul>
                    </li>
                    <li data-level="2">
                        <a href="#二-3">二</a>
                    </li>
                </ul>
            </li>
        </ul>
    </div>
</p>
```
为啥要转换成这样子呢？因为这样子就能显示我们想要的效果呀。

为啥`ul`和`li`元素有一个`data-level`属性呢？看名字就知道是为了方便知道目前所在的层级。

那么我们先把上面的`<html>`理解一下吧

首先，将`p`元素的`[TOC]`替换为`div`元素，且该`div`元素有一个叫`toc`的class
```
<p>
    <div class="toc"></div>
</p>
```
接着，要实现层次效果，我们需要用到`ul`、`li`元素，搭建一级标题吧
```
<p>
    <div class="toc">
        <ul data-level="1">
            <li data-level="1">
                <a>一</a>
            </li>
        </ul>
    </div>
</p>
```
放到chrome中看看效果呢

![data-level=1](https://github.com/02954/markdownpad2_toc/blob/master/images/data-level=1.png)

继续搭，不准停...
```
<p>
    <div class="toc">
        <ul data-level="1">
            <li data-level="1">
                <a>一</a>
                <ul data-level="2">
                    <li data-level="2">
                        <a>二</a>
                    </li>
                </ul>
            </li>
        </ul>
    </div>
</p>
```
继续放到chrome中

![data-level=2](https://github.com/02954/markdownpad2_toc/blob/master/images/data-level=2.png)

那如果有两个标题二呢？再加个`li`呗
```
<p>
    <div class="toc">
        <ul data-level="1">
            <li data-level="1">
                <a>一</a>
                <ul data-level="2">
                    <li data-level="2">
                        <a>二</a>
                    </li>
                    <li data-level="2">
                        <a>二</a>
                    </li>
                </ul>
            </li>
        </ul>
    </div>
</p>
```
继续放到chrome中，很棒

![data-level=2_2](https://github.com/02954/markdownpad2_toc/blob/master/images/data-level=2_2.png)

如果又要在第一个标题二下加一个标题三呢？
```
<p>
    <div class="toc">
        <ul data-level="1">
            <li data-level="1">
                <a>一</a>
                <ul data-level="2">
                    <li data-level="2">
                        <a>二</a>
                        <ul data-level="3">
                            <li data-level="3">
                                <a>三</a>
                            </li>
                        </ul>
                    </li>
                    <li data-level="2">
                        <a>二</a>
                    </li>
                </ul>
            </li>
        </ul>
    </div>
</p>
```
![data-level=3](https://github.com/02954/markdownpad2_toc/blob/master/images/data-level=3.png)

想要的效果就完成啦，棒棒哒~

那么我们通过JavaScript要实现的过程大概也就是酱紫滴

### 编码

边编码，边讲解方案

#### 初始HTML

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>MarkdownPad2导出HTML支持[TOC]</title>
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
</head>
<body>
    <h1>一</h1>
    <p>[TOC]</p>
    <h2>二</h2>
    <h3>三</h3>
    <h2>二</h2>
</body>
</html>
```
引入jquery.js，加入`h1`、`p`、`h2`等元素，效果如下

![code initial](https://github.com/02954/markdownpad2_toc/blob/master/images/codeInitial.png)

#### 整改p元素

步骤：
1. 创建class为toc的div元素
2. 找到p元素，怎么找通过`[TOC]`来找p元素
3. 将p元素的文本元素清空，并将步骤1中的div元素append到该p元素上
```
<script>
    $(document).ready(function() {
        // 创建class为toc的div元素
        let $toc = $('<div></div>');
        $toc.attr('class', 'toc');

        // 清除p元素的文本元素，并将$toc元素append到p元素
        $('p:contains([TOC]):first').text('').append($toc);
    });
</script>
```
如果Markdown文档中有多个[TOC]语法，我们只取第一个，其余的不做转换，谁没事搞这么多个[TOC]呢，是不？当然有人站出来说，一定要有多个[TOC]，行行行，那你把`:first`出去就行了。

另外，这个代码还有点小缺陷，如果有些人习惯用[toc]呢？发现`<p>[toc]</p>`并没有变化，所以需要整改一下
```
<script>
    $(document).ready(function() {
        // 新增Contains选择器，:contains的变异版，增加忽略大小写功能
        $.expr[':'].Contains = function(a, i, m){
            return jQuery(a).text().toUpperCase().indexOf(m[3].toUpperCase()) >= 0;
        };

        // 创建class为toc的div元素
        let $toc = $('<div></div>');
        $toc.attr('class', 'toc');

        // 清除p元素的文本内容，并将$toc元素append到p元素
        $('p:Contains([TOC]):first').text('').append($toc);
    });
</script>
```
新增了一个`:Contains`，与`:contains`功能一样增加忽略大小写功能，这样就算写成`[toc]`也是没问题了，效果如下

![HTML p](https://github.com/02954/markdownpad2_toc/blob/master/images/htmlP.png)

#### 实现hx元素转换

下一步当然就是得往`<div class="toc"></div>`中append一些`ul`、`li`元素了

处理第一个hx元素的步骤：
1. 获取Markdown文档中所有hx元素$headers
2. 获取第一个hx元素firstHeader，并获取第一个hx元素的层级firstLevel
3. 对firstLevel循环处理，创建ul、li元素并按层级append
4. ul、li元素append完成后，找到firstLevel对应的li元素，并append一个a元素用于显示标题内容

```
// 获取h1-h6元素
let $headers = $('h1,h2,h3,h4,h5,h6');

// 获取第一个hx元素，注意是dom对象，非jQuery对象
let firstHeader = $headers[0];

if (firstHeader) {
    // 获取第一个hx元素的层级
    let firstLevel = parseInt(firstHeader.tagName.
        replace('H', ''), 10);
    // 给a元素唯一的href属性值
    let id = `${firstHeader.textContent}-0`;

    for (let i = 1; i <= firstLevel; i++) {
        // 创建ul和li元素，并添加data-level属性
        let $ul = $('<ul></ul>'),
            $li = $('<li></li>');
        $li.attr('data-level', i);
        $ul.attr('data-level', i).append($li);

        // 获取data-level为i-1的li元素
        let $pLi = $toc.find(`li[data-level=${i - 1}]`);

        if ($pLi.length > 0) {
            // 找到data-level为i-1的元素，直接append到该li元素上即可
            $pLi.append($ul);
        } else {
            // 未找到data-level为i-1的元素，说明是顶层的ul，
            // 直接append到class为toc的div元素上
            $toc.append($ul);
        }
    }

    // 找到所属层级的li元素，并添加a元素
    $toc.find(`li[data-level=${firstLevel}]`).append(
        $(`<a href="#${id}">${firstHeader.textContent}</a>`));
    // 与a元素的href对应，用于文档内跳转
    $headers.eq(0).attr('id', id);
}
```
$headers内容如下：

![jQuery Headers](https://github.com/02954/markdownpad2_toc/blob/master/images/jqHeaders.png)

界面效果如下：

![First Header1](https://github.com/02954/markdownpad2_toc/blob/master/images/htmlFirstHeader1.png)

如果Markdown文档中没有从`#`开始，直接用了`##`，那么我们试下效果（相当于把文档中的`<h1>一</h1>`删除）

![First Header2](https://github.com/02954/markdownpad2_toc/blob/master/images/htmlFirstHeader2.png)

效果ok，就是有点小问题，a元素在chrome中呈现下划线效果，建议去掉，所以增加以下代码，把原来的`<h1>一</h1>`加回来，嗯，现在效果完美了
```
<style>
    .toc a {
        text-decoration: none;
    }
</style>
```
![First Header3](https://github.com/02954/markdownpad2_toc/blob/master/images/htmlFirstHeader3.png)


处理后续hx元素的步骤：
1. 从第二个hx元素开始，循环
2. 获取当前hx的层级以及上一个hx的层级
3. 当当前层级大于上一个层级，则从上一个层级+1到当前层级做循环添加ul和li元素
4. 如果当前层级和上一个层级相等，则找到相同层级的li元素的父元素，创建li元素并添加上去
5. 如果当前层级小于上一个层级，则找到最后一个相同层级的li元素的父元素，添加$l
6. 设置hx元素的id，与a的href保持一致

```
// 从第二个hx元素开始，循环$headers
for (let i = 1; i < $headers.length; i++) {
    // 获取当前的hx元素的层级以及上一个hx元素的层级
    let curLevel = parseInt($headers[i].tagName.replace('H', ''),
        10),
        lastLevel = parseInt($headers[i - 1].tagName.
            replace('H', ''), 10);
    // 当前hx元素的文本元素
    let textContent = $headers[i].textContent,
        id = `${textContent}-i`;
    // 给a元素唯一的href属性值，与hx元素的id对应
    let $a = $(`<a href="#${id}">${textContent}</a>`);

    if (lastLevel < curLevel) {
        // 如果当前层级比上一个层级大，则从上一个层级+1到当前层级循环，
        // 循环添加ul和li元素
        for (let j = lastLevel + 1; j <= curLevel; j++) {
            // 创建ul和li元素，并添加data-level属性
            let $u = $('<ul></ul>'),
                $l = $('<li></li>');
            $l.attr('data-level', j);
            $u.attr('data-level', j).append($l);

            // 找到data-level为j-1的最后一个li元素，并添加$u
            $toc.find(`li[data-level=${j - 1}]:last`).append($u);
        }
        // 找到data-level为curLevel的最后一个li，并添加$a
        $toc.find(`li[data-level=${curLevel}]:last`).append($a);
    } else if (lastLevel === curLevel) {
        // 如果当前层级和上一个层级相等，则找到相同层级的li元素的父元素，
        // 创建li元素并添加上去
        // 已经存在data-level为curLevel的ul，只需要创建li即可
        let $l = $('<li></li>');

        // 给li元素添加data-level树形，并添加$a
        // 找到data-level为curLevel的最后一个li元素的父元素，添加$l
        $toc.find(`li[data-level=${curLevel}]:last`).parent().
            append($l.attr('data-level', curLevel).append($a));
    } else {
        // 如果当前层级小于上一个层级，则找到最后一个相同层级的li元素的
        // 父元素，添加$l
        // 找到data-level为curLevel的最后一个li元素,必然存在
        let $sublingLi = $toc.
            find(`li[data-level=${curLevel}]:last`);
        let $l = $('<li></li>');

        // 找到$sublingLi的父元素,添加li元素即可
        $sublingLi.parent().append($l.attr('data-level', curLevel).
            append($a));
    }

    $headers.eq(i).attr('id', id);
}
```

全部代码如下：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>MarkdownPad2导出HTML支持[TOC]</title>
    <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
    <style>
        .toc a {
            text-decoration: none;
        }
    </style>
</head>
<body>
    <h1>一</h1>
    <p>[TOC]</p>
    <h2>二</h2>
    <h3>三</h3>
    <h2>二</h2>

    <script>
        $(document).ready(function() {
            // 新增Contains选择器，:contains的变异版，增加忽略大小写功能
            $.expr[':'].Contains = function(a, i, m){
                return jQuery(a).text().toUpperCase().indexOf(m[3].
                    toUpperCase()) >= 0;
            };

            // 创建class为toc的div元素
            let $toc = $('<div></div>');
            $toc.attr('class', 'toc');

            // 获取h1-h6元素
            let $headers = $('h1,h2,h3,h4,h5,h6');

            // 获取第一个hx元素，注意是dom对象，非jQuery对象
            let firstHeader = $headers[0];

            if (firstHeader) {
                // 获取第一个hx元素的层级
                let firstLevel = parseInt(firstHeader.tagName.
                    replace('H', ''), 10);
                // 给a元素唯一的href属性值
                let id = `${firstHeader.textContent}-0`;

                for (let i = 1; i <= firstLevel; i++) {
                    // 创建ul和li元素，并添加data-level属性
                    let $ul = $('<ul></ul>'),
                        $li = $('<li></li>');
                    $li.attr('data-level', i);
                    $ul.attr('data-level', i).append($li);

                    // 获取data-level为i-1的li元素
                    let $pLi = $toc.find(`li[data-level=${i - 1}]`);

                    if ($pLi.length > 0) {
                        // 找到data-level为i-1的元素，直接append到该li元素上即可
                        $pLi.append($ul);
                    } else {
                        // 未找到data-level为i-1的元素，说明是顶层的ul，
                        // 直接append到class为toc的div元素上
                        $toc.append($ul);
                    }
                }

                // 找到所属层级的li元素，并添加a元素
                $toc.find(`li[data-level=${firstLevel}]`).append(
                    $(`<a href="#${id}">${firstHeader.textContent}</a>`));
                // 与a元素的href对应，用于文档内跳转
                $headers.eq(0).attr('id', id);
            }

            // 从第二个hx元素开始，循环$headers
            for (let i = 1; i < $headers.length; i++) {
                // 获取当前的hx元素的层级以及上一个hx元素的层级
                let curLevel = parseInt($headers[i].tagName.replace('H', ''),
                    10),
                    lastLevel = parseInt($headers[i - 1].tagName.
                        replace('H', ''), 10);
                // 当前hx元素的文本元素
                let textContent = $headers[i].textContent,
                    id = `${textContent}-i`;
                // 给a元素唯一的href属性值，与hx元素的id对应
                let $a = $(`<a href="#${id}">${textContent}</a>`);

                if (lastLevel < curLevel) {
                    // 如果当前层级比上一个层级大，则从上一个层级+1到当前层级循环，
                    // 循环添加ul和li元素
                    for (let j = lastLevel + 1; j <= curLevel; j++) {
                        // 创建ul和li元素，并添加data-level属性
                        let $u = $('<ul></ul>'),
                            $l = $('<li></li>');
                        $l.attr('data-level', j);
                        $u.attr('data-level', j).append($l);

                        // 找到data-level为j-1的最后一个li元素，并添加$u
                        $toc.find(`li[data-level=${j - 1}]:last`).append($u);
                    }
                    // 找到data-level为curLevel的最后一个li，并添加$a
                    $toc.find(`li[data-level=${curLevel}]:last`).append($a);
                } else if (lastLevel === curLevel) {
                    // 如果当前层级和上一个层级相等，则找到相同层级的li元素的父元素，
                    // 创建li元素并添加上去
                    // 已经存在data-level为curLevel的ul，只需要创建li即可
                    let $l = $('<li></li>');

                    // 给li元素添加data-level树形，并添加$a
                    // 找到data-level为curLevel的最后一个li元素的父元素，添加$l
                    $toc.find(`li[data-level=${curLevel}]:last`).parent().
                        append($l.attr('data-level', curLevel).append($a));
                } else {
                    // 如果当前层级小于上一个层级，则找到最后一个相同层级的li元素的
                    // 父元素，添加$l
                    // 找到data-level为curLevel的最后一个li元素,必然存在
                    let $sublingLi = $toc.
                        find(`li[data-level=${curLevel}]:last`);
                    let $l = $('<li></li>');

                    // 找到$sublingLi的父元素,添加li元素即可
                    $sublingLi.parent().append($l.attr('data-level', curLevel).
                        append($a));
                }

                $headers.eq(i).attr('id', id);
            }


            // 清除p元素的文本内容，并将$toc元素append到p元素
            $('p:Contains([TOC]):first').text('').append($toc);
        });
    </script>
</body>
</html>
```
效果如下：

![HTML Headers](https://github.com/02954/markdownpad2_toc/blob/master/images/htmlHeaders.png)

### MarkdownPad2效果

以上代码编写完毕，但只是一个demo，我们的目的是为了让MarkdownPad2导出HTML时支持[TOC]，所以，先删除不必要的代码
```
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.js"></script>
<script>
    $(document).ready(function() {
        // 新增Contains选择器，:contains的变异版，增加忽略大小写功能
        $.expr[':'].Contains = function(a, i, m){
            return jQuery(a).text().toUpperCase().indexOf(m[3].
                toUpperCase()) >= 0;
        };

        // 创建class为toc的div元素
        let $toc = $('<div></div>');
        $toc.attr('class', 'toc');

        // 获取h1-h6元素
        let $headers = $('h1,h2,h3,h4,h5,h6');

        // 获取第一个hx元素，注意是dom对象，非jQuery对象
        let firstHeader = $headers[0];

        if (firstHeader) {
            // 获取第一个hx元素的层级
            let firstLevel = parseInt(firstHeader.tagName.
                replace('H', ''), 10);
            // 给a元素唯一的href属性值
            let id = `${firstHeader.textContent}-0`;

            for (let i = 1; i <= firstLevel; i++) {
                // 创建ul和li元素，并添加data-level属性
                let $ul = $('<ul></ul>'),
                    $li = $('<li></li>');
                $li.attr('data-level', i);
                $ul.attr('data-level', i).append($li);

                // 获取data-level为i-1的li元素
                let $pLi = $toc.find(`li[data-level=${i - 1}]`);

                if ($pLi.length > 0) {
                    // 找到data-level为i-1的元素，直接append到该li元素上即可
                    $pLi.append($ul);
                } else {
                    // 未找到data-level为i-1的元素，说明是顶层的ul，
                    // 直接append到class为toc的div元素上
                    $toc.append($ul);
                }
            }

            // 找到所属层级的li元素，并添加a元素
            $toc.find(`li[data-level=${firstLevel}]`).append(
                $(`<a href="#${id}">${firstHeader.textContent}</a>`));
            // 与a元素的href对应，用于文档内跳转
            $headers.eq(0).attr('id', id);
        }

        // 从第二个hx元素开始，循环$headers
        for (let i = 1; i < $headers.length; i++) {
            // 获取当前的hx元素的层级以及上一个hx元素的层级
            let curLevel = parseInt($headers[i].tagName.replace('H', ''),
                10),
                lastLevel = parseInt($headers[i - 1].tagName.
                    replace('H', ''), 10);
            // 当前hx元素的文本元素
            let textContent = $headers[i].textContent,
                id = `${textContent}-i`;
            // 给a元素唯一的href属性值，与hx元素的id对应
            let $a = $(`<a href="#${id}">${textContent}</a>`);

            if (lastLevel < curLevel) {
                // 如果当前层级比上一个层级大，则从上一个层级+1到当前层级循环，
                // 循环添加ul和li元素
                for (let j = lastLevel + 1; j <= curLevel; j++) {
                    // 创建ul和li元素，并添加data-level属性
                    let $u = $('<ul></ul>'),
                        $l = $('<li></li>');
                    $l.attr('data-level', j);
                    $u.attr('data-level', j).append($l);

                    // 找到data-level为j-1的最后一个li元素，并添加$u
                    $toc.find(`li[data-level=${j - 1}]:last`).append($u);
                }
                // 找到data-level为curLevel的最后一个li，并添加$a
                $toc.find(`li[data-level=${curLevel}]:last`).append($a);
            } else if (lastLevel === curLevel) {
                // 如果当前层级和上一个层级相等，则找到相同层级的li元素的父元素，
                // 创建li元素并添加上去
                // 已经存在data-level为curLevel的ul，只需要创建li即可
                let $l = $('<li></li>');

                // 给li元素添加data-level树形，并添加$a
                // 找到data-level为curLevel的最后一个li元素的父元素，添加$l
                $toc.find(`li[data-level=${curLevel}]:last`).parent().
                    append($l.attr('data-level', curLevel).append($a));
            } else {
                // 如果当前层级小于上一个层级，则找到最后一个相同层级的li元素的
                // 父元素，添加$l
                // 找到data-level为curLevel的最后一个li元素,必然存在
                let $sublingLi = $toc.
                    find(`li[data-level=${curLevel}]:last`);
                let $l = $('<li></li>');

                // 找到$sublingLi的父元素,添加li元素即可
                $sublingLi.parent().append($l.attr('data-level', curLevel).
                    append($a));
            }

            $headers.eq(i).attr('id', id);
        }


        // 清除p元素的文本内容，并将$toc元素append到p元素
        $('p:Contains([TOC]):first').text('').append($toc);
    });
</script>
```
然后打开MarkdownPad2，工具-->选项-->高级-->HTML Head编辑器，将上述代码拷贝至代码编辑器中，保存并关闭-->保存并关闭

MarkdownPad2左侧填入一下内容：
```
# 《Vue2.0开发去哪儿网App》知识点
[TOC]
## 深入理解Vue组件
### 4-8 动态组件与v-once指令
#### 动态组件
#### v-once指令
## Vue中的动画特效
### 5-1 Vue动画 - Vue中的CSS动画原理
#### 动画原理
### 5-2 在Vue中使用animate.css库
#### keyframes动画
#### 自定义类名
#### animate.css库
```
文件-->导出-->导出HTML，填写文件名，保存，效果如下：

![result](https://github.com/02954/markdownpad2_toc/blob/master/images/result.png)

### GitHub项目

GitHub项目地址：https://github.com/02954/markdownpad2_toc

