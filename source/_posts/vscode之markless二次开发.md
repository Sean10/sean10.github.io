# 理解

`markdown`是有对应的标准的. 似乎这篇可以参考?

下面todo中提到的一些`list`的识别场景, 其实这篇`spec`有做一定的边界澄清?


[CommonMark Spec](https://spec.commonmark.org/0.30/)

[中文版CommonMark Spec](http://yanxyz.github.io/commonmark-spec/)


# 单元测试

## TODO:针对vscode渲染后的页面, 如何进行单元测试感知? html的识别?

## TODO:针对vscode提供的API, 如何验证其渲染效果?


# Issue

## 搜索, 点到行后, 渲染的位置偏移跳转太严重了.

## 如何支持直接复制渲染后的内容?

## comments 现在

## 长期运行后, 似乎会出现资源占用问题, 卡顿


疑似是1.0.25这次这个增加自定义引起的这个问题

似乎是1.0.25去掉了memoize引起的.

## comments 现在老是自动弹出来.

右键了这个弹出来的comments的`hide comments`了, 不知道会怎么样. 无效
似乎
跟之前设置的Panel: Opens Maximized 设置有关



## 是否有持久化渲染的实现方式

* vscode-markdown 看看?
* 新开的那种to-side的preview 窗口是否是?

如何区分切换文件和文件内编辑?
* onDidChangeActiveTextEditor
* 其实切换文件也不重要, 关键是感知到文件是第一次被打开就行?

* 似乎变量是持久化渲染的, 所以理论上是可以做计数器的?
* 插件的变量是全局的, 所以需要维护一张表, 打开的map, 占用这个内存? 而不是每次切换就打开一次?

* TODO: 这个可以作为功能处理吧? 如果需要频繁切换, 减少图片渲染的代价则处理. 纳入计划中.

## 每次切换文件, 为什么comment重新渲染时, 焦点位置就变了呢? 使用comment, 是否可以仅修改的图片变更渲染, 而其他的不变? 应该可以做到.

即使用内存来减少这部分切换

判定是文件刚切换带来的行切换事件,从而绕过这种渲染?

TODO:指针判定里增加哪些位置未渲染的逻辑吧, 这样仅当指针出现在存在未渲染的页面时,才会对那些位置进行渲染. 其他情况下指针的迁移, 不触发渲染.

## 报错 `Error setting up request: SyntaxError: Invalid regular expression: /*.md/: Nothing to repeat`, 忽然就无法渲染了

## 会出现图片重复渲染的情况

好像就是在那几行写新的对象, 那个comment并没有自动被换行符往后推. 

好像就是在图片渲染那行上面, 敲回车, 会逐渐走到图片下面去. 而不是让图片切换行? 这个所以应该是我这个插件处理的问题吧?

## 是否可以从comment切换成其他的来进行渲染? 既然html都支持的话.

## 偶尔会出现某个单元没渲染的情况似乎...

## TODO: 打开文件, 好像如果不点一下其他位置, 好像还不会触发渲染?

## TODO:support data src image preview

## page jump when switch to other file and back. maybe image preview's problem

可能只要不点到那里, 就不触发图片重新渲染更加合适? 但是如果是批量位置修改呢? 被选中的时候再重新渲染?

主要涉及需要重新渲染的时候, 应该是用户点击那行的时候. 但是每次切换的时候, 实际上会触发一次

搞个索引存储吧, 每行尽量只渲染一次吧. 渲染完, 如果没有选中, 就不再触发?

`state.activeEditor.document.lineAt`来存储, 好像不太行? 怎么把一行里所有的内容都存储进去呢? 以range为范围? 然后通过一张map记录? 为了减少时间复杂度?

如果`open`触发的不是`onDidChangeActiveTextEditor`这个的话, 其实把这个关闭就可以了?

切换文件的时候, `onDidChangeTextEditorVisibleRanges`这个也被触发了. 关键问题是, 图片的渲染应该是我在切换页面之后发生的, 所以导致页面上滑了...

应该是其中初始化置空那次的问题.

但是`onDidChangeActiveTextEditor`感知不到每个文件是第一次打开, 所以导致其实切换

`ImageComment`的高度固化有可能不

到底是因为切换走了, 导致渲染全部没了, 还是说只是代码里的因素呢?`Comment`

记错了, 不是以文本为单位的. 还是以进程为单位的, 也就代表确实需要一个线程处理所有的markdown文件渲染了...这种情况下, 也就是最好uri也保存一下map...

TODO: 图片是在线程里, 开了一张map. `createCommentThread`

确实是离开的时候, 触发了dispose图片引起的? 

好像不清理线程, 就会出现图片重复被塞进渲染线程里的情况?

好像每次都会被push进去的原因是Range对象是不同的? 在string化的时候, 也是可变的. 是不是因为这个导致不同? 不对啊, 我维持了一张`rangeMap`, 那不应该出现那个不匹配的问题才对啊. 临时变量的地址应该是不同的, 所以最好转换一下成string?

奇怪, 怎么每次每个对象都触发了2次呢?

另外, 这个列表应该会触发几次, 理论上这张map里应该有3个或者以下的对象才对.

试了下, 官方的`comment-sample`也会触发这个跳变的情况. 所以这里其实要解决的历史已经渲染的, 实时渲染的问题?

## 图片渲染不支持`webp`格式, 也可能是本地`file`协议不支持? (官方已合入支持)

[MarkdownString cannot render webp image · Issue \#144188 · microsoft/vscode](https://github.com/microsoft/vscode/issues/144188)

但是`markdown all in one`的`preview`是支持的, 所以可能我的用法不太对?

看了下这个库里只是调用了官方的`markdown.showPreview`command, 所以可能得看官方的里是怎么渲染webp的

` <img src="../imgs/leveldb3.webp" alt="" class="loading" id="image-hash--205375520" data-src="../imgs/leveldb3.webp">`

`addImageRenderer`, 官方使用的`MarkdownIt`触发的image渲染应该.

那问题就是这个,`MarkdownString`并不是用的`MarkdownIt`, 走的两套代码方案实现的? 看官方怎么回复吧.

yes~就是一开始发现的那个`private readonly validExtensions = new Set(['.svg', '.png', '.jpg', '.jpeg', '.gif', '.bmp']);`这里加上就支持了.


## 疑似某种文件格式下的bug

`TypeError: Cannot read property 'start' of undefined`
at vscode-file://vscode-app/Users/sean10/.vscode/extensions/sean10.markless-sean10-1.0.24/out/main.js:53
	at Generator.next (<anonymous>)
	at o (vscode-file://vscode-app/Users/sean10/.vscode/extensions/sean10.markless-sean10-1.0.24/out/main.js:1)

那个租房的文章里的.

## performance问题, 总觉得输入的时候稍微有一点卡顿的现象

TODO:有没有能够精确的profile我的插件的实际cpu占用时间呢?
有点怀疑是我引入的那个log库占用的资源.

## image渲染后默认显示大小不是配置的`Panel: Opens Maximized`

`createCommentThread`这个接口操作的.

`panel`应该属于`comment`的父类的感觉.

`collapsibleState`目前已经设置了`Expand`了. 所以问题是, `expand`的`panel`的默认加载模式?

所以, 其实需要知道的是, 什么时候他才读取`Panel: Opens Maximized`里的变量.

`private panelOpensMaximized() {`似乎是在这里控制?

`toggleMaximizedPanel` 这个函数为什么默认触发的始终是`temrinal`的呢?

TODO:这个变量哪次提交引入的呢`PANEL_OPENS_MAXIMIZED`, 是不是我用错了?

* 是不是可以像人为手动折叠一次一样, 自动完成, 这样就能放大成最大化的了.
  * 不行, 试了下, 通过属性设置的`Expanded`依旧是部分图片状态.

所以关键还是在这个手动折叠展开, 用的哪个函数了.




> A collection of comments representing a conversation at a particular range in a document.

`Comment`这个类型似乎还有`edit`和`preview`模式. 不过初步看与这个问题的解决方案无关, 可能如果官方不提供api, 那就得更换`comment`以外的方案?

## table不识别插入的`<br/>`问题

代码里用的`remark-gfm`的渲染, 理论上不应该存在这个问题才对?.

试了下demo, 似乎是`remark-gfm`的问题? 难道是`<br/>`不符合gfm标准?

``` js
async function main() {
  const file = await unified()
    .use(remarkParse)
    .use(remarkGfm)
    .use(remarkRehype, {allowDangerousHtml: true})
    // .use(rehypeSanitize)
    .use(rehypeStringify)
    .process(data)
    fs.writeFileSync("output.html", String(file))

  // console.log(String(file))
}
```

根据[HTML blocks inside table cells not treated as blocks · Issue \#10 · remarkjs/remark\-gfm](https://github.com/remarkjs/remark-gfm/issues/10)找到设置允许危险的方式, 可以渲染`html`

看来还是那个`XSS`的风险, 所以导致默认关闭了. 并不是`remarkGfm`的问题.

所以我也可以增加这个属性, 默认关闭即可.

这里作者用的不是`remark`系列的了, `mdast-util-to-hast`和`hast-util-to-html`应该是`remark-rehype`的前身.

在`toHast`和`toHtml`处, 都打开这个`{allowDangerousHtml: true}`就可以了. 但是的确危险系数还是比较高的.

之后考虑直接更换为`remark-rehype`.

### 启用了2个危险属性之后, 变成空白了...

应该是`svg`没有渲染出来把?

开了debug之后, 可以看到确实html的AST是有的, 应该就是`svg`渲染的问题了.

确实, 是`<br/>`在被转换后变成了`<br>`, 导致不符合`svg`语法了. 符合语法之后就正常了. 我把这个作为一个选项开启吧.

## TODO: 原生HTML的渲染能力

在打开了这个`allowDangerousHtml`之后, 也变得可以实现了

现在遇到的问题是, 理论上`<h1>`的样例, 应该能显示出来才对, 但是没进入到`html`的node解析逻辑里...

看起来是`remark`这个解析库拦截掉了? 并没有, 单独打印出`AST`还是有的, 但是的确进不到那个内部逻辑了现在.

带结局

## quote之后的内容导致卡顿

疑似是我行的长度过长之后的渲染就会比较卡顿? 至少滚动就不流畅了.

应该是`blockquote`这段的. 

但是卡顿这个我怎么排查是哪的呢? 好像是css切换那块的. 我把`filter`换掉能不能解决呢?

去掉`filter`的渲染之后就正常了.

## 拆分无序列表和有序列表的渲染

### 理解
`state.offset`用途?

`ListItems`是哪里得到的`type`?

好像某个第三方库的, 不是这套代码里的.

搜一下.

疑似是来自这段的.

```
const parser = require('unified')()
    .use(require('remark-math'))
    .use(require('remark-parse'))
    .use(require('remark-gfm'))
    .parse;
```

嗯, 没错, 应该是`remark`这个`markdown`解析库里的`listitem`.

怎么区分有序列表和无序列表呢?


根据下面这段, 一开始`type`为`list`, 具有`ordered`属性, `children`中才是具体的列表内容.

所以如果要改, 预计需要在判断`listitem`前就进行区分处理.
```
 {
      "type": "list",
      "ordered": true,
      "start": 1,
      "spread": true,
      "children": [
        {
          "type": "listItem",
          "spread": false,
          "checked": null,
          "children": [
            {
              "type": "paragraph",
              "children": [
                {
                  "type": "text",
                  "value": "First",
                  "position": {
                    "start": {
                      "line": 71,
                      "column": 4,
                      "offset": 437
                    },
                    "end": {
                      "line": 71,
                      "column": 9,
                      "offset": 442
                    }
                  }
                }
              ],
              "position": {
                "start": {
                  "line": 71,
                  "column": 4,
                  "offset": 437
                },
                "end": {
                  "line": 71,
                  "column": 9,
                  "offset": 442
                }
              }
            }
          ],
          "position": {
            "start": {
              "line": 71,
              "column": 1,
              "offset": 434
            },
            "end": {
              "line": 71,
              "column": 9,
              "offset": 442
            }
          }
        },
        {
          "type": "listItem",
          "spread": false,
          "checked": null,
          "children": [
            {
              "type": "paragraph",
              "children": [
                {
                  "type": "text",
                  "value": "Second",
                  "position": {
                    "start": {
                      "line": 73,
                      "column": 4,
                      "offset": 447
                    },
                    "end": {
                      "line": 73,
                      "column": 10,
                      "offset": 453
                    }
                  }
                }
              ],
              "position": {
                "start": {
                  "line": 73,
                  "column": 4,
                  "offset": 447
                },
                "end": {
                  "line": 73,
                  "column": 10,
                  "offset": 453
                }
              }
            }
          ],
          "position": {
            "start": {
              "line": 73,
              "column": 1,
              "offset": 444
            },
            "end": {
              "line": 73,
              "column": 10,
              "offset": 453
            }
          }
        },
```

todo: `extension.js`里`start`和`end`和`node`这几个参数哪里传进来的呢?

现阶段传入的每个`node`, ordered属性并不在单个上面.

预计需要改成三层

* list
  * chilren
    * listItem
      * children
        * paragraph


无序的则是
* listItem
  * chilren
    * paragraph

或者
* listItem
  * children
    * paragraph
    * list
      * children
        * listItem
          * children
            * paragraph

存在多层关联的这种.不能直接以node的type来区分了.

这个节点的遍历关系? 是DFS还是BFS?

目前并没有找到


## 增加http链接图片支持

只是单纯的增加`http`的`schema`的透传就可以, 剩下的交给`vscode`自身支持

似乎支持了?`1.62`里?


> MarkdownString.supportHtml
> The new supportHtml property on MarkdownString enables rendering of a safe subset of raw HTML that appears inside the Markdown text.
> 
> The supportHtml property defaults to false. When disabled, VS Code will strip out any raw HTML tags that appear in the Markdown text.



## 修复`latex`显示的渲染结果太小的问题

用`Elements`定位了下, 实际上是`svg`渲染的图片的渲染尺寸不对.

大致是通过 html/svg+xml base64 渲染 svg.

通过在ebfore元素里增加`url`的`content`.

把SVG代码直接内联在CSS的url()方法中

这个逻辑里可能之所以会根据行的数量来决定要渲染的长度, 应该是为了确保不会点到后面的空行, 结果触发的是这行的内容?

这个的做法是不是弄成自动收缩这几行更好?

mermaid的图也有同样的问题.

### 分析lineHeight
首先, 我的配置中默认`lineHeight`是设置为`0`的, 所以走的下面这个逻辑

> Use 0 to automatically compute the line height from the font size.
> Values between 0 and 8 will be used as a multiplier with the font size.
> Values greater than or equal to 8 will be used as effective values.

根据这个, 所以跟`font size`应该是成成比例的.

``` js
    const lineHeight = vscode.workspace.getConfiguration("editor").get("lineHeight", 0);
    // https://github.com/microsoft/vscode/blob/45aafeb326d0d3d56cbc9e2932f87e368dbf652d/src/vs/editor/common/config/fontInfo.ts#L54
    if (lineHeight === 0) {
        state.lineHeight = Math.round(process.platform == "darwin" ? 1.5 * state.fontSize : 1.35 * state.fontSize);
    } else if (lineHeight < 8) {
        state.lineHeight = 8;
    }
```

的确, 发现这段代码这里`lineHeight`只得到了2, 大概是这里写错了. 可能作者不是`darwin`平台的, 所以感知不到.



## 多行的`latex`除了按行数整比放大代码之外, 是否有正常的方式折叠到那几行?

* collapse
* folding


* FoldingRange
* registerFoldingRangeProvider
  
Todo:怎么找不到`javascript`如何使用`FoldingRangeProvider`的呢...

使用方式类似下面这样:
``` javascript
context.subscriptions.push(vscode.languages.registerFoldingRangeProvider('markdown', {
		provideFoldingRanges: (document, context, _) => {
		console.log(document.languageId);
		console.log("try to folding range");
		let ranges = []
		const temp = new vscode.FoldingRange(1, 50, vscode.FoldingRangeKind.Comment);
		ranges.push(temp);
		return ranges
	}}));
```

`context.subscriptions.push`如何理解?

是不是这个只是用在当我即将触发`folding`的时候, 才会进入这个函数逻辑? 确实, 比如我设置`kind`为`Comment`的时候, 触发`Fold all comments`就能够把我标记的范围给圈起来.

所以自动圈, 应该不是这个函数


## fold/unfolder API?
目前根据这个[\[folding\] API for programmatically folding lines · Issue \#37682 · microsoft/vscode](https://github.com/microsoft/vscode/issues/37682)来看似乎是不存在的.

根据[Built\-in Commands \| Visual Studio Code Extension API](https://code.visualstudio.com/api/references/commands)这篇里的`editor.fold`

发现还是文档里没写清楚参数的数据结构, 搜代码样例从这里找到的[autofold/extension\.ts at c721aa8b616060624b107cbdd44c44ec4827adec · TigerGold59/autofold](https://github.com/TigerGold59/autofold/blob/c721aa8b616060624b107cbdd44c44ec4827adec/src/extension.ts)

实际使用方式是类似这种
``` js
vscode.commands.executeCommand('editor.fold', {"selectionLines": [1]});
```

## wrod wrap折叠后的后面几行开头无渲染效果

比如`quote`的特殊符号仅第一行有.

## 修复[When the cursor is in the line of heading, should it show '\#' at the begining of the line? · Issue \#10 · Sean10/markless](https://github.com/Sean10/markless/issues/10)

2组rangeBehavior目前没有办法合并.

当处在中间时, 没有办法让左侧的识别到右侧的参与.

左侧的隐藏, 右侧的需要放大

* 鼠标在这行上时, 希望所有效果全部取消. 直到`cursor`离开该行
* 目前效果: 
    * 右侧的渲染: 鼠标在行末, 取消了放大效果. 鼠标在heading的左侧离开时, 重新恢复放大效果
    * 左侧的渲染: 鼠标在行末甚至`heading`期间, 依旧不显示
      * 指针直到接近`# `的范围, 才取消隐藏.
* 考虑增加一个覆盖的操作, 当鼠标出现在这个范围内, 取消所有的装饰器. 但是条件必须是这个触发效果在最后. 从而覆盖掉另外俩装饰效果.

考虑咨询下这个的效果[兄弟萌，让我们在 Vscode 里放烟花吧 \- 51CTO\.COM](https://developer.51cto.com/art/202106/668897.htm)

似乎他对这个range使用比较熟练.

如果我这个后加, 应该就有效了把?

> A TextEditorDecorationType is a disposable which can be removed/releaded by calling its dispose-method
好像有清理的逻辑?

我先试试现在这套代码的pop逻辑?pop 不行, 我需要的是清理掉对应的decoration. 而不是现在的这种.

可以考虑增加一个清理掉所有的css的装饰, 在鼠标在上面时触发, 离开时, 这个效果被清理掉.

如果用dispose, 那所有的装饰器的变量也释放了, 如果要用这个, 那就必须要把每个heading 维护一个单独的装饰器, 否则就一起被释放了.


所以最好是有一个disable的方法, 比如从队列里去掉这个. 可能维护一张map, 或者这个列表里就自带了行号的属性, 这样就能够比较当前行是不是其中一个了.

官方的2个方法:[Remove decoration type · Issue \#22 · microsoft/vscode\-extension\-samples](https://github.com/microsoft/vscode-extension-samples/issues/22)
> option1 to nuke all uses of a certain decoration type across all editors is decType.dispose()
> option2 to nuke all uses of a certain decoration type in a single editor instance is editor.setDecorations(decType, [])

好像`memoize`这个函数还做了缓存加速.


好像代码里用的是offset, 而不是Range. 这样的话的确没有`line`属性了.

这是原生的`decorationRanges`的限制吗? 之前好像没看到这种要求. `Range`噢, 2种构造函数而已.

todo:成功了. 不过好像`enlarge`那个, 因为指针一移动, 又加回去了的样子? 这个明天再修一下.

## heading的字体放大, 似乎其实也可以接着放大. 好像行的高度, 会跟着我的指定的字体大小变大而变大

所以并不是我一开始理解的字体大小默认设置14, 这个行就只能容纳14的情况.

## heading的隐藏存在一个问题, 凡是在标题上触发的其他模式的渲染, 依旧会存在渲染错的问题, 清理可能需要考虑取消所有该位置的渲染.

是通过反查? 还是通过map出来之后, 检查?

## `block quote`存在2个渲染问题
### 位置, 依旧是按照未隐藏`#`来的

可能需要梳理一下, 这种怎么让多种模式能够复用一套清理逻辑?

怎么获取到对应的字的宽度呢? 毕竟不是以类型来显示`node`, 而是反过来拆解的.

是否可以引入关联性呢? 毕竟本身是存在`children`的概念的, 只是在子节点的时候, 不知道怎么访问到父亲节点的内容.

如果增加父亲节点的访问链接, 那就可以迭代父节点, 是否存在其他属性需要渲染了 ? 这样的话, 就可以按顺序一个个处理了? 这个好像其实就是迭代顺序? 只是后接收到的处理的数据依旧是原始数据, 而不是上级已经处理过后的数据?

目前是按照`visit`的顺序. 如果让每次visit之后, 使用的数据也进行更新呢? 这样的话, 其实会在原始的数据里增加好几个链接? 

#### 思路
* 修改visit之后, 使用的node数据来源? 或者增加基础属性? 比如font等?
* 修改数据结构? 从map改为链表关系? map之间其实是可以营造链表效果的.
* 暂时沿用之前那个修改`listItem`的思路, 直接在visite的时候, 把上层的`depth`赋给子节点.
#### 暂时沿用之前那个修改`listItem`的思路, 直接在visite的时候, 把上层的`depth`赋给子节点.

目前存在的问题: 位置是移动了, 但是好像还有一层外圈没跟着移动? 怀疑不是`inlineCode`的装饰.  的确, 注释掉这个处理之后, 还存在. 怀疑是其他插件的. 确实, `markdown_all_in_one`里的...

那如果我解决了这个问题, 可能还需要考虑那边如何关闭? 毕竟挺多快捷键依赖那个的...

先试试吧.

另外, 这里如果成功了, 还需要考虑`inlineCode`在进入该行时, 自动被消除的操作.

另外, 似乎大小也并没有跟着变...

优先尝试解决border的`size`的问题吧?

[Border do not match correctly if the font size if too big or small · Issue \#33 · leodevbro/vscode\-blockman](https://github.com/leodevbro/vscode-blockman/issues/33)
根据这个来看, 好像是根据`lineHeight`来的? 虽然`fontSize`大的时候好像把`lineHeight`也撑大了, 但是看起来`border`没有变. 根据这个的意思, 好像可以把`fontSize`调大, 然后影响border. 不过这块有点不理解, 为啥目前`fontSize`已经大了, 并没有正常呢?

试了下, 直接在元素上加`border`属性是能够按照字体大小扩大`border`的. 所以问题就在于现在背景上显示的`border`到底绑定在哪个元素上.

奇怪, 为什么全局搜索, 在`html`的这个总元素上搜到的呢? 而且调整一点用都没有...

todo:现在的核心痛点就是怎么找到这个`border`的`css`设置. 不在默认点击找到的`div`属性里, 因为删掉该行依旧存在. 好像确实很难找到. 不知道chrome为啥有的css就找不到呢? 之后再看看

`view-overlays`? 

终于一个个找找到的`<div class="cdr ced-1-TextEditorDecorationType10-0" style="left:0px;width:36px;height:18px;"></div>`


.monaco-editor .ced-1-TextEditorDecorationType10-0

ced-1-TextEditorDecorationType10-0
奇怪, 这个class的`css`在哪?

手动逐个删除二分定位是找到了

但是在哪呢?

`vscode createTextEditorDecorationType view-overlays`

什么情况下, 装饰器会被添加到一个独立的div里呢? 为啥不复用文本的那部分呢?

`export class ViewOverlays extends ViewPart implements IVisibleLinesHost<ViewOverlayLine> {`

-> `ContentViewOverlays`

const contentViewOverlays = new ContentViewOverlays(this._context);

export class View extends ViewEventHandler {

这里构造的时候就能看到了.

		contentViewOverlays.addDynamicOverlay(new CurrentLineHighlightOverlay(this._context));
		contentViewOverlays.addDynamicOverlay(new SelectionsOverlay(this._context));
		contentViewOverlays.addDynamicOverlay(new IndentGuidesOverlay(this._context));
		contentViewOverlays.addDynamicOverlay(new DecorationsOverlay(this._context));

怀疑进到不同的渲染里了?

好像view-overlays是属于标准的?不对

一个是`view-lines` 是我之前那几个元素的. `view-overlays`是那几个Css渲染的...基本上目前只看到`border`会进这里.

Todo: 忽然想到一个猜测点. 由于html的正文始终只有一份, 会不会是如果对一个range存在多个css的装饰器, 则会把新增的装饰器添加到`view-overlays`里. 

todo: 但是理论上其实是可以做到识别用户插入的多个装饰器, 然后对Range进行拆分的吧? 拆分到比如以1个字符为单位, 然后重叠装饰器.

可以验证下, 是哪种. 这块不知道官方文档里有没有提到.

emm, `*`和`**`并没有进入到view-overlays, 依旧在view-lines

TODO: 还是说仅限于`border`这种参数才会被放到overlays?

尝试了, `outline`也被加到`overlays`里了, 而`top`虽然是同一个装饰器塞入的, 这个操作被放到了`view-lines`里. 所以初步怀疑是根据属性划分? 明明可以把`border`属性直接绑在对应的位置, 为啥非得拆到`overlays`里呢?

应该是了, 这行单纯只有`border`属性,同样也是被加在`overlays`范畴.

目前, 位置对上了. 但是因为和字体大小不匹配, 还是会存在丢失的情况.

TODO:把判定当前行进入修改的逻辑封装为函数比较好.

在`View`构造的时候, 会构造`ViewLines`和`ViewOverlays`

好像是在`registerThemingParticipant`这里触发的?

`.monaco-editor .ced-1-TextEditorDecorationType10-0`, 注册时应该是`.ced-`这个关键词

走的这个`CSSNameHelper`...这样的话, 就没

`cdr`呢? 具体的`div`上有这个`class`

在`DecorationsOverlay`的`_renderNormalDecoration`

是这条`contentViewOverlays.addDynamicOverlay(new DecorationsOverlay(this._context));`

`ViewOverlays`->`prepareRender`


`viewPart.prepareRender(renderingContext);`

好像接下来就是`ViewImpl`里的触发渲染条件调用的了.

那现在问题就是那个`border`参数是什么时候被传递进来的了? 我应该使用的是`createTextEditorDecorationType`的`state.activeEditor.setDecorations`

一时这个好像没找到到底做了什么

从这里添加`DecorationsOverlay`啥都没找到,  直接用的`const className = d.options.className!;`

`let decorations: ViewModelDecoration[] = [], decorationsLen = 0;`

`const _decorations = ctx.getDecorationsInViewport();`

TODO:好像走到了这段`_getDecorationsViewportData`, 这里是不是就是我需要知道的区分`border`和`textDecoration`的地方?

`		const renderingContext = new RenderingContext(this._context.viewLayout, viewportData, this._viewLines);`




### 大小, 也是按照字体的初始大小, 而不是渲染后的大小来的.

* 是否能够感知当前行已经渲染的字体大小等参数, 然后基于这个做二次渲染呢?

毕竟完全是存在特效重叠的情况的.

翻了下文档, `border`好像没有`size`的调整属性. 那是否这个就是限制呢? 感觉不至于. 那猜测, 可能是因为`decoration`没有命中同一个区域. 所以没能采用到那个较大的size?

根据这篇[Border do not match correctly if the font size if too big or small · Issue \#33 · leodevbro/vscode\-blockman](https://github.com/leodevbro/vscode-blockman/issues/33)跟我的现象一样


根据[Get editor line height \(px\) and character space\-width \(px\) by VSCode API · Issue \#125341 · microsoft/vscode](https://github.com/microsoft/vscode/issues/125341#issuecomment-854962933)这个可以知道目前为止, 这个`feature`官方暂时不做.


## 批量处理当前行, 去除装饰器的效果
本来像统一用

```
	const delDecorationIfCurrentLine = (node, decoration, start=0, end=0, ...others) => {
		// console.log("decoration: ", decoration, "others:", others);

		if (node.position.start.line - 1 == editor.selection.active.line) {
			delDecoration(state, decoration, editor.selection.active.line);
		} else {
			if (others.length > 0){
				addDecoration(decoration(...others), start, end);
			} else {
				addDecoration(decoration, start, end);
			}
		}
	}
```

但是遇到一个问题, 居然有的`node`没有`position`属性...

* table
* latex
* img link

## f5调试显示的效果, 和本地打包安装后不同.

不对, `1.0.17`版本正常, 之后的版本和现在开发的版本都有问题.

`1.0.18`版本是否正常, 代表了是否我这个修改的思路是正确的...

奇怪, 好像是文件的问题

 我这个文件的, node, 行号始终在400行以内显示. 而实际上我行已经到2000行了.

 用remark跑了下, 是能到2000行的呀...为啥呢?

```
node:  221 
{type: "heading", depth: 2, children: Array(1), position: {…}}
children: Array(1)
0: {type: "text", value: "有序列表下无序列表下有序列表", position: {…}}
length: 1
__proto__: Array(0)
depth: 2
position:
end: {line: 222, column: 18, offset: 7909}
start: {line: 222, column: 1, offset: 7892}
__proto__: Object
type: "heading"

```

实际上是1569行, 为啥显示222行?

另外, 似乎此时的渲染速度慢了很多, 是不是因为那个遍历的操作? 是不是得内容优化

目前来看, 这个遍历的资源开销特别大, 达到用户会感知的层级了...

 这块先优化成map再看吧.

emm, map里最后存的不是地址, 而是这个range对象, 这样的话可能不太行. 除非我把所有的Range对象都做了单例模式?

奇怪, 我做了单例, 为啥还找不到呢? 好像一定情况下照不出来是正常的. 因为如果同一行有多个渲染, 那就找不到.

是不是因为`decoration`这个不是单例, 有好几次会创建新的?

这样的话, 就会导致我以为是同一个装饰器, 但是实际上这个装饰器是新的, 里面没有之前的

其实这样也没太大问题. 至少解释不了为什么第二次进这个函数, 底下就变成undefined了. 还真是一个全新的.

基于这个遍历的方案成功是成功了, 但是cpu占用太高.

而且试验了下, `onDidChangeTextEditorSelection`是可以捕获到当前的行的. 至于更进一步的思路再想想.

考虑只处理指定行? 选中范围内会自动触发, 在临界范围内, range取消的能力. 这就不存在资源消耗了.

所以可能得放弃根据行来判断的逻辑.

好像大佬在`setDecorations`这里做了实现?

```
        if (state.config.cursorDisables) {
            ranges = ranges.filter((r) => !state.selection.intersection(r));
        }
```

好像是利用这里更新range, 直接去除存在范围内的. 这样的话, 我只需要把行添加到这里, 也能起到一样的效果. 

确实, 实现了.

关键是因为`setDecoration`这里做了一次循环遍历了, 所以导致我在那侧做的实现, 完全重复了. 出现了2重循环引起的.

但是那个`##`连带着这个一起放大的问题还是存在, 那就跟性能没关系了.


### (放弃)让索引位置与字符所在位置对应上? 不让其出现删除空格, 实际是删除了最后的文字的问题.

如何实现呢? 一种就是让由于存在被隐藏了空格的部分出现. 但是由于`heading`的特性是在于表头, 因此无法复现.

## 多现实2个'#'的问题, 意思`hide`异常

398行为什么在`node`中显示265?

为什么`content`打印出了`list`的项?

奇怪, 为什么日志里记录的行号, 并不是我当前看的这几行呢? 所以是渲染的范围有问题?

```
const ms = new vscode.MarkdownString(latexElement);
            ms.isTrusted = true;
```
初步怀疑是这块构造部分的`markdown`这里出的问题.

``` json
range = new vscode.Range(Math.max(range.start.line - 200, 0), 0, range.end.line + 200, 0);
```
好像是这里引起的, 所以按照这段的意思, 如果把我超过400行的json去掉.是不是就正确了?

好像还是不对. 还是怀疑这里的range存在问题. 

```
{type: "heading", depth: 3, children: Array(1), position: {…}}
children: Array(1)
0: {type: "text", value: "作废的折叠代码.", position: {…}}
length: 1
__proto__: Array(0)
depth: 3
position:
end: {line: 421, column: 13, offset: 11791}
start: {line: 421, column: 1, offset: 11779}

```

噢噢. 忽然意识到问题点了. 这里`construct`传入的是个相对路径, 但是我却去当做真实路径计算引起的问题.

奇怪, 为什么我必须用`node`, 而不是直接用

## 日志里记录`heading`的逻辑里会出现普通节点的行...



## 可能存在内存泄露, 由于装饰器的不断new. 清空也只是[], 并不是dispose.  有必要增加单例


## TODO:存在单行由于字体过大, 没有自动换行的问题.

目前看到的实现, 是在`vscode`进入`view-line`的渲染之前就修改了字体大小等才行. 如果等到`view-line`建完之后, wrap已经结束了

TODO: 所以需要考虑是不是有什么API可以重新指定wrap. 或者重新根据我的渲染结果重新触发`render`的`view-line`. 可能也得看下wrap的实现代码有没有对应的接口.

## sometimes, 还是会出现`#`没被隐藏的问题. 1.0.22版本. 
依旧存在'#'的复现条件, 似乎是切换Editor和切换其他文件的时候发生的.


### Illegal value for `line`

奇怪, 不是只要是整型就行了吗?
``` js
mainThreadExtensionService.ts:64 Error: Illegal value for `line`
	at l._lineAt (vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/workbench/services/extensions/node/extensionHostProcess.js:88)
	at Object.lineAt (vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/workbench/services/extensions/node/extensionHostProcess.js:88)
	at updateSelectionToLine (vscode-file://vscode-app/Users/sean10/Code/markless/out/main.js:55935)
	at setDecorations (vscode-file://vscode-app/Users/sean10/Code/markless/out/main.js:55944)
	at runNextTicks (internal/process/task_queues.js:58)
	at processTimers (internal/timers.js:494)
```

最后定位到, 实际上还是`editor`切换的时候, 全局变量没有重新更新的问题,用了错误的`editor`了. 这样行号肯定是对不上的.

### 在`heading`处的
最后发现之前加的`const editor = vscode.window.activeTextEditor;`在切换`editor`时没能切换, 所以访问错文件的line了.


## TODO:疑似图片默认不会全放大状态, 通过Paste Image现加的渲染的时候, 需要收缩一次再点开才会按照我的配置进行? 包括切换editor的时候, 切换完好像也有问题, 除了关闭重新打开好像是正常的


## TODO: 性能优化, 触控板滚动的时候, 有触发selection的变化, 或者什么的变化引起重新渲染吗?




## 增加日志动态调整记录.
### config修改的同步时间是什么时候呢?
我改完, 再切换回窗口, 我看到触发的还是原来的配置.

config好像还有`active`那几个`mermaid`的逻辑要用到.

这里的俩`registerWebviewViewProvider`这个就不注册了把

### 小模块就不考虑按模块/功能拆分日志了, 直接手动过滤筛选吧.

### TODO: support env? [Expose log level and log path to extensions · Issue \#40053 · microsoft/vscode](https://github.com/microsoft/vscode/issues/40053)


## `clearDecotaions`里的对象找不到`setDecoration`,

初步怀疑是切换到非markdown的文件上, 出现的问题. 没能把函数取消掉.


### 部分`listItem`的无序列表的那个符号怎么好像没被添加上去? 隐藏了?

初步实验, 好像关键点是这行含不含有中文?

奇怪, 看起来`decoration`还是生成了的.

在`addDecoration`内部失败的, 目前初步怀疑是`vscode`版本升级引入的问题?


# todo

TODO:可以通过`MarkdownString`来完成彻底的渲染html. 自带了这个能力

todo: 第一级有序列表无法被`verified`库解析出`ordered`的`list`属性. 所以这种得翻一下`unified`的API, 他的判断逻辑里, 如果只有一层, 那还具有列表属性吗? listItem应该也有区分前缀才对?

todo: 遍历一层下述节点时, 第一级的`node`中包含了其内的子节点的属性. 然后会再次遍历其中的子节点. 所以这里会存在一个覆盖的效果. 目前来看只有`text`层级的`position`属性可以作为一个表来判定节点之间的关联.

todo: 下属节点的遍历能力是否要进行区分? 可能没办法区分, 因为只有遍历进`paragraph`层之后才能知道其children是否是`list`还是`listItem`.

* todo: 如果我模拟的有序列表的数字并不连续, 是否还能判定为有序列表? 得看下unified代码. 如果这里可以让直接根据在每个node的属性那里就能区分出是`ordered`与否, 那就不用考虑状态机转移了. emm, 似乎根据现有代码, 可以直接利用正则手动处理. 虽然性能低一点.
    * todo: 这里出现一个新问题, node的`value`里只有去掉了列表之后的内容, 现在导致我无法正则判断了. 怎么才能从解析后的`node`反得到对应的完整行呢? 虽然有`range`可以手动找到.
    * Todo: 我似乎可以找到对应的level层级, 手动得到前面的地址, 然后通过`range`提取到值


根据上面的`spec`可知`有序列表的开始数字由第一个列表项的数字决定，而不考虑 后面的列表项。`  所以理论上`remark`应该直接解析出`list`的`ordered`属性才对?

`remark-parse`主要用的[syntax\-tree/mdast: Markdown Abstract Syntax Tree format](https://github.com/syntax-tree/mdast#list)这个库来进行的解析.

`mdast`这个库里只是文档? 那`Remark`这么多`repo`之间的关系到底是什么?

`unified`代码里没搜到`listItem`,

重新推一下

```

const parser = require('unified')()
    .use(require('remark-math'))
    .use(require('remark-parse'))
    .use(require('remark-gfm'))
    .parse;
```
所以应该在`remark-parse`或者`remark-gfm`里. emm, `remark-gfm`下载下来和`mdast`一样是空的...


`unified.js`和`remark.js`是什么关联? 如果单纯只是API聚合的话, 应该不值得赞助吧?

噢, 根据这篇[An Introduction to Unified and Remark \- Braincoke \| Security Blog](https://braincoke.fr/blog/2020/03/an-introduction-to-unified-and-remark/#syntax-trees), `remark`看起来更像是`unifed`的下属子集能力.
```
import {fromMarkdown} from 'mdast-util-from-markdown'
```
根据这个, 而这个代码里又写来自于`micromark`, 所以说明这个数据来源是这个库.

的确在这个库的代码里搜到了`listItem`.

根据`micromark`里的描述

> remark is the most popular markdown parser. It’s built on top of micromark and boasts syntax trees. For an analogy, it’s like if Babel, ESLint, and more, were one project.

不过似乎语法树的部分解析可能还是在其他的代码里的.

乍看, 核心逻辑都在对应的`compile`里的`handler`那里.


`micromark`输出的是`html`, 那之前那些节点信息, 是哪个插件的呢?

奇怪, 用`remark-parse`输出的节点结构基本对应上了

```
{
    "type": "root",
    "children": [
        {
            "type": "list",
            "ordered": true,
            "start": 1,
            "spread": false,
            "children": [
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "1",
                                    "position": {
                                        "start": {
                                            "line": 1,
                                            "column": 4,
                                            "offset": 3
                                        },
                                        "end": {
                                            "line": 1,
                                            "column": 5,
                                            "offset": 4
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 1,
                                    "column": 4,
                                    "offset": 3
                                },
                                "end": {
                                    "line": 1,
                                    "column": 5,
                                    "offset": 4
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 1,
                            "column": 1,
                            "offset": 0
                        },
                        "end": {
                            "line": 1,
                            "column": 5,
                            "offset": 4
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "2",
                                    "position": {
                                        "start": {
                                            "line": 2,
                                            "column": 4,
                                            "offset": 8
                                        },
                                        "end": {
                                            "line": 2,
                                            "column": 5,
                                            "offset": 9
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 2,
                                    "column": 4,
                                    "offset": 8
                                },
                                "end": {
                                    "line": 2,
                                    "column": 5,
                                    "offset": 9
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 2,
                            "column": 1,
                            "offset": 5
                        },
                        "end": {
                            "line": 2,
                            "column": 5,
                            "offset": 9
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "3",
                                    "position": {
                                        "start": {
                                            "line": 3,
                                            "column": 4,
                                            "offset": 13
                                        },
                                        "end": {
                                            "line": 3,
                                            "column": 5,
                                            "offset": 14
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 3,
                                    "column": 4,
                                    "offset": 13
                                },
                                "end": {
                                    "line": 3,
                                    "column": 5,
                                    "offset": 14
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 3,
                            "column": 1,
                            "offset": 10
                        },
                        "end": {
                            "line": 3,
                            "column": 5,
                            "offset": 14
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 1,
                    "column": 1,
                    "offset": 0
                },
                "end": {
                    "line": 3,
                    "column": 5,
                    "offset": 14
                }
            }
        }
    ],
    "position": {
        "start": {
            "line": 1,
            "column": 1,
            "offset": 0
        },
        "end": {
            "line": 5,
            "column": 1,
            "offset": 16
        }
    }
}

```

好像跟`html`的结构一样, 先是外部结构, `ul`还是`ol`, 然后才是内部的item渲染.

这样的话, 也得按照`html`的逻辑去渲染, 才能区分有序列表和无序列表了把.

现阶段的实现上, 是按照`node`迭代去渲染的, 所以这块当进入子节点的时候就没办法感知了.

这样的话, 最好是找到内部节点, 然后往上去遍历节点是什么结构, 然后再渲染?

那这样的话, 最好渲染一次就不是渲染一个节点, 而是一组节点的. 这样就可以在`ordered`变量存在的那层处理了.

当节点是`listItem`类型的时候, 这里存一下父节点的信息, 是不是`list`,然后`type`为`list`的`ordered`参数作为渲染用参数.


最好是在得到这个`node`信息的时候, 就遍历一下, 然后把这个属性给加到`listItem`那项里.

接下来就是在什么遍历的时候增加这个属性了.

应该哪种都行. 反正只要type匹配即可. 增加在Visit那里了

### 作废的折叠代码.
```
	// context.subscriptions.push(vscode.languages.registerFoldingRangeProvider('markdown', {
	// 	provideFoldingRanges: (document, context, _) => {
	// 	console.log(document.languageId);
	// 	console.log("try to folding range");
	// 	let ranges = []s
	// 	const temp = new vscode.FoldingRange(1, 50, vscode.FoldingRangeKind.Comment);
	// 	ranges.push(temp);
	// 	return ranges
	// }}));
	// vscode.commands.executeCommand('editor.fold', {"selectionLines": [1]});
```

### todo: 回车和backspace按键卡顿, 怀疑跟这个`markless`的实时渲染有关. 好像禁用了markless还是卡, 开了进程监控, 也没看到是哪个进程的cpu显著的高, 除了main窗口.


## todo: `markless`是如何隐藏一部分文字的? 但是实际好像又能被搜索到? 透明色?





# 数据记录

在每个大节点的访问之后, 会再进到children内进行子节点的访问.

## 无序列表下无序列表
``` json
{
    "type": "listItem",
    "spread": false,
    "checked": null,
    "children": [
        {
            "type": "paragraph",
            "children": [
                {
                    "type": "text",
                    "value": "b",
                    "position": {
                        "start": {
                            "line": 4,
                            "column": 3,
                            "offset": 13
                        },
                        "end": {
                            "line": 4,
                            "column": 4,
                            "offset": 14
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 4,
                    "column": 3,
                    "offset": 13
                },
                "end": {
                    "line": 4,
                    "column": 4,
                    "offset": 14
                }
            }
        },
        {
            "type": "list",
            "ordered": false,
            "start": null,
            "spread": false,
            "children": [
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "b.a",
                                    "position": {
                                        "start": {
                                            "line": 5,
                                            "column": 5,
                                            "offset": 19
                                        },
                                        "end": {
                                            "line": 5,
                                            "column": 8,
                                            "offset": 22
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 5,
                                    "column": 5,
                                    "offset": 19
                                },
                                "end": {
                                    "line": 5,
                                    "column": 8,
                                    "offset": 22
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 5,
                            "column": 3,
                            "offset": 17
                        },
                        "end": {
                            "line": 5,
                            "column": 8,
                            "offset": 22
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "b.b",
                                    "position": {
                                        "start": {
                                            "line": 6,
                                            "column": 5,
                                            "offset": 27
                                        },
                                        "end": {
                                            "line": 6,
                                            "column": 8,
                                            "offset": 30
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 6,
                                    "column": 5,
                                    "offset": 27
                                },
                                "end": {
                                    "line": 6,
                                    "column": 8,
                                    "offset": 30
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 6,
                            "column": 3,
                            "offset": 25
                        },
                        "end": {
                            "line": 6,
                            "column": 8,
                            "offset": 30
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "b.c",
                                    "position": {
                                        "start": {
                                            "line": 7,
                                            "column": 5,
                                            "offset": 35
                                        },
                                        "end": {
                                            "line": 7,
                                            "column": 8,
                                            "offset": 38
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 7,
                                    "column": 5,
                                    "offset": 35
                                },
                                "end": {
                                    "line": 7,
                                    "column": 8,
                                    "offset": 38
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 7,
                            "column": 3,
                            "offset": 33
                        },
                        "end": {
                            "line": 7,
                            "column": 8,
                            "offset": 38
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 5,
                    "column": 3,
                    "offset": 17
                },
                "end": {
                    "line": 7,
                    "column": 8,
                    "offset": 38
                }
            }
        }
    ],
    "position": {
        "start": {
            "line": 4,
            "column": 1,
            "offset": 11
        },
        "end": {
            "line": 7,
            "column": 8,
            "offset": 38
        }
    }
}

```
## 无序列表下有序列表

``` json
{
    "type": "listItem",
    "spread": false,
    "checked": null,
    "children": [
        {
            "type": "paragraph",
            "children": [
                {
                    "type": "text",
                    "value": "o.1",
                    "position": {
                        "start": {
                            "line": 17,
                            "column": 3,
                            "offset": 93
                        },
                        "end": {
                            "line": 17,
                            "column": 6,
                            "offset": 96
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 17,
                    "column": 3,
                    "offset": 93
                },
                "end": {
                    "line": 17,
                    "column": 6,
                    "offset": 96
                }
            }
        },
        {
            "type": "list",
            "ordered": true,
            "start": 1,
            "spread": false,
            "children": [
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "o.1.1",
                                    "position": {
                                        "start": {
                                            "line": 18,
                                            "column": 6,
                                            "offset": 102
                                        },
                                        "end": {
                                            "line": 18,
                                            "column": 11,
                                            "offset": 107
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 18,
                                    "column": 6,
                                    "offset": 102
                                },
                                "end": {
                                    "line": 18,
                                    "column": 11,
                                    "offset": 107
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 18,
                            "column": 3,
                            "offset": 99
                        },
                        "end": {
                            "line": 18,
                            "column": 11,
                            "offset": 107
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "o.1.2",
                                    "position": {
                                        "start": {
                                            "line": 19,
                                            "column": 6,
                                            "offset": 113
                                        },
                                        "end": {
                                            "line": 19,
                                            "column": 11,
                                            "offset": 118
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 19,
                                    "column": 6,
                                    "offset": 113
                                },
                                "end": {
                                    "line": 19,
                                    "column": 11,
                                    "offset": 118
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 19,
                            "column": 3,
                            "offset": 110
                        },
                        "end": {
                            "line": 19,
                            "column": 11,
                            "offset": 118
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "o.1.3",
                                    "position": {
                                        "start": {
                                            "line": 20,
                                            "column": 6,
                                            "offset": 124
                                        },
                                        "end": {
                                            "line": 20,
                                            "column": 11,
                                            "offset": 129
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 20,
                                    "column": 6,
                                    "offset": 124
                                },
                                "end": {
                                    "line": 20,
                                    "column": 11,
                                    "offset": 129
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 20,
                            "column": 3,
                            "offset": 121
                        },
                        "end": {
                            "line": 20,
                            "column": 11,
                            "offset": 129
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 18,
                    "column": 3,
                    "offset": 99
                },
                "end": {
                    "line": 20,
                    "column": 11,
                    "offset": 129
                }
            }
        }
    ],
    "position": {
        "start": {
            "line": 17,
            "column": 1,
            "offset": 91
        },
        "end": {
            "line": 20,
            "column": 11,
            "offset": 129
        }
    }
}
```

## 有序列表下有序列表
``` json
{
    "type": "listItem",
    "spread": false,
    "checked": null,
    "children": [
        {
            "type": "paragraph",
            "children": [
                {
                    "type": "text",
                    "value": "4",
                    "position": {
                        "start": {
                            "line": 12,
                            "column": 4,
                            "offset": 58
                        },
                        "end": {
                            "line": 12,
                            "column": 5,
                            "offset": 59
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 12,
                    "column": 4,
                    "offset": 58
                },
                "end": {
                    "line": 12,
                    "column": 5,
                    "offset": 59
                }
            }
        },
        {
            "type": "list",
            "ordered": true,
            "start": 1,
            "spread": false,
            "children": [
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "4.1",
                                    "position": {
                                        "start": {
                                            "line": 13,
                                            "column": 7,
                                            "offset": 66
                                        },
                                        "end": {
                                            "line": 13,
                                            "column": 10,
                                            "offset": 69
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 13,
                                    "column": 7,
                                    "offset": 66
                                },
                                "end": {
                                    "line": 13,
                                    "column": 10,
                                    "offset": 69
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 13,
                            "column": 4,
                            "offset": 63
                        },
                        "end": {
                            "line": 13,
                            "column": 10,
                            "offset": 69
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "4.2",
                                    "position": {
                                        "start": {
                                            "line": 14,
                                            "column": 7,
                                            "offset": 76
                                        },
                                        "end": {
                                            "line": 14,
                                            "column": 10,
                                            "offset": 79
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 14,
                                    "column": 7,
                                    "offset": 76
                                },
                                "end": {
                                    "line": 14,
                                    "column": 10,
                                    "offset": 79
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 14,
                            "column": 4,
                            "offset": 73
                        },
                        "end": {
                            "line": 14,
                            "column": 10,
                            "offset": 79
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "4.3",
                                    "position": {
                                        "start": {
                                            "line": 15,
                                            "column": 7,
                                            "offset": 86
                                        },
                                        "end": {
                                            "line": 15,
                                            "column": 10,
                                            "offset": 89
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 15,
                                    "column": 7,
                                    "offset": 86
                                },
                                "end": {
                                    "line": 15,
                                    "column": 10,
                                    "offset": 89
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 15,
                            "column": 4,
                            "offset": 83
                        },
                        "end": {
                            "line": 15,
                            "column": 10,
                            "offset": 89
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 13,
                    "column": 4,
                    "offset": 63
                },
                "end": {
                    "line": 15,
                    "column": 10,
                    "offset": 89
                }
            }
        }
    ],
    "position": {
        "start": {
            "line": 12,
            "column": 1,
            "offset": 55
        },
        "end": {
            "line": 15,
            "column": 10,
            "offset": 89
        }
    }
}
```

## 有序列表下无序列表下有序列表
``` json
{
    "type": "listItem",
    "spread": false,
    "checked": null,
    "children": [
        {
            "type": "paragraph",
            "children": [
                {
                    "type": "text",
                    "value": "p.1",
                    "position": {
                        "start": {
                            "line": 22,
                            "column": 3,
                            "offset": 133
                        },
                        "end": {
                            "line": 22,
                            "column": 6,
                            "offset": 136
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 22,
                    "column": 3,
                    "offset": 133
                },
                "end": {
                    "line": 22,
                    "column": 6,
                    "offset": 136
                }
            }
        },
        {
            "type": "list",
            "ordered": true,
            "start": 1,
            "spread": false,
            "children": [
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "p.1.1",
                                    "position": {
                                        "start": {
                                            "line": 23,
                                            "column": 6,
                                            "offset": 142
                                        },
                                        "end": {
                                            "line": 23,
                                            "column": 11,
                                            "offset": 147
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 23,
                                    "column": 6,
                                    "offset": 142
                                },
                                "end": {
                                    "line": 23,
                                    "column": 11,
                                    "offset": 147
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 23,
                            "column": 3,
                            "offset": 139
                        },
                        "end": {
                            "line": 23,
                            "column": 11,
                            "offset": 147
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "p.1.1.1",
                                    "position": {
                                        "start": {
                                            "line": 24,
                                            "column": 8,
                                            "offset": 155
                                        },
                                        "end": {
                                            "line": 24,
                                            "column": 15,
                                            "offset": 162
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 24,
                                    "column": 8,
                                    "offset": 155
                                },
                                "end": {
                                    "line": 24,
                                    "column": 15,
                                    "offset": 162
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 24,
                            "column": 5,
                            "offset": 152
                        },
                        "end": {
                            "line": 24,
                            "column": 15,
                            "offset": 162
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "p.1.1.2",
                                    "position": {
                                        "start": {
                                            "line": 25,
                                            "column": 8,
                                            "offset": 170
                                        },
                                        "end": {
                                            "line": 25,
                                            "column": 15,
                                            "offset": 177
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 25,
                                    "column": 8,
                                    "offset": 170
                                },
                                "end": {
                                    "line": 25,
                                    "column": 15,
                                    "offset": 177
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 25,
                            "column": 5,
                            "offset": 167
                        },
                        "end": {
                            "line": 25,
                            "column": 15,
                            "offset": 177
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "p.1.1.3",
                                    "position": {
                                        "start": {
                                            "line": 26,
                                            "column": 8,
                                            "offset": 185
                                        },
                                        "end": {
                                            "line": 26,
                                            "column": 15,
                                            "offset": 192
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 26,
                                    "column": 8,
                                    "offset": 185
                                },
                                "end": {
                                    "line": 26,
                                    "column": 15,
                                    "offset": 192
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 26,
                            "column": 5,
                            "offset": 182
                        },
                        "end": {
                            "line": 26,
                            "column": 15,
                            "offset": 192
                        }
                    }
                },
                {
                    "type": "listItem",
                    "spread": false,
                    "checked": null,
                    "children": [
                        {
                            "type": "paragraph",
                            "children": [
                                {
                                    "type": "text",
                                    "value": "p.1.2",
                                    "position": {
                                        "start": {
                                            "line": 27,
                                            "column": 6,
                                            "offset": 198
                                        },
                                        "end": {
                                            "line": 27,
                                            "column": 11,
                                            "offset": 203
                                        }
                                    }
                                }
                            ],
                            "position": {
                                "start": {
                                    "line": 27,
                                    "column": 6,
                                    "offset": 198
                                },
                                "end": {
                                    "line": 27,
                                    "column": 11,
                                    "offset": 203
                                }
                            }
                        }
                    ],
                    "position": {
                        "start": {
                            "line": 27,
                            "column": 3,
                            "offset": 195
                        },
                        "end": {
                            "line": 27,
                            "column": 11,
                            "offset": 203
                        }
                    }
                }
            ],
            "position": {
                "start": {
                    "line": 23,
                    "column": 3,
                    "offset": 139
                },
                "end": {
                    "line": 27,
                    "column": 11,
                    "offset": 203
                }
            }
        }
    ],
    "position": {
        "start": {
            "line": 22,
            "column": 1,
            "offset": 131
        },
        "end": {
            "line": 27,
            "column": 11,
            "offset": 203
        }
    }
}
```



# Reference
1. [CommonMark Spec](https://spec.commonmark.org/0.30/)
