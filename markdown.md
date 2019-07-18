# Markdown示例

---

## 标题H2

### 标题H3

#### 标题H4

##### 标题H5

###### 标题H6

### 字符效果和横线等

```markdown
## 标题H2

### 标题H3

#### 标题H4

##### 标题H5

###### 标题H6

### 字符效果和横线等
```

---

## 强调内容

适合显示重要的提示信息，语法为 !> 内容。

!> 适合显示重要的提示信息，语法为 !> 内容，可以和其他 **Markdown** 语法混用。

?> 普通的提示信息，比如写 TODO 或者参考内容等。

```markdown

!> 适合显示重要的提示信息，语法为 !> 内容，可以和其他 **Markdown** 语法混用。

?> 普通的提示信息，比如写 TODO 或者参考内容等。
```

### 引用

> 引用文本

```markdown
> 引用文本
```

--

~~删除线~~ <s>删除线（开启识别HTML标签时）</s>

*斜体字*      _斜体字_

**粗体**  __粗体__

***粗斜体*** ___粗斜体___

上标：X<sub>2</sub>，下标：O<sup>2</sup>

```markdown

~~删除线~~ <s>删除线（开启识别HTML标签时）</s>

*斜体字*      _斜体字_

**粗体**  __粗体__

***粗斜体*** ___粗斜体___

上标：X<sub>2</sub>，下标：O<sup>2</sup>
```

### 分隔线

***

---

```markdwown
***

---
```

### 锚点与链接

[普通链接](https://www.mdeditor.com/)  
[普通链接带标题](https://www.mdeditor.com/ "普通链接带标题")  
直接链接：<https://www.mdeditor.com>  
[anchor-id]: https://www.mdeditor.com/  
[mailto:test.test@gmail.com](mailto:test.test@gmail.com)  
邮箱地址自动链接 test.test@gmail.com  www@vip.qq.com  

```markdown
[普通链接](https://www.mdeditor.com/)  
[普通链接带标题](https://www.mdeditor.com/ "普通链接带标题")  
直接链接：<https://www.mdeditor.com>  
[anchor-id]: https://www.mdeditor.com/  
[mailto:test.test@gmail.com](mailto:test.test@gmail.com)  
邮箱地址自动链接 test.test@gmail.com  www@vip.qq.com  
```

## 代码

### 行内代码

执行命令：`npm install marked`

```markdown
执行命令：`npm install marked`
```

### 缩进风格

即缩进四个空格，也做为实现类似 `<pre>` 预格式化文本 ( Preformatted Text ) 的功能。

    <?php
        echo "Hello world!";
    ?>

预格式化文本：

    | First Header  | Second Header |
    | ------------- | ------------- |
    | Content Cell  | Content Cell  |
    | Content Cell  | Content Cell  |

```markdown
    | First Header  | Second Header |
    | ------------- | ------------- |
    | Content Cell  | Content Cell  |
    | Content Cell  | Content Cell  |
```

### JS代码

```javascript
function test() {
	console.log("Hello world!");
}
```

```html
<!DOCTYPE html>
<html>
    <body>
        <p class="text-green">Plain text</p>
    </body>
</html>
```

## 图片

图片缩放

![logo](https://docsify.js.org/_media/icon.svg ':size=50x100')
![logo](https://docsify.js.org/_media/icon.svg ':size=100')

图片加链接 (Image + Link)：

[![](https://docsify.js.org/_media/icon.svg)](https://docsify.js.org/#/zh-cn/helpers "markdown")


```markdown
图片缩放

![logo](https://docsify.js.org/_media/icon.svg ':size=50x100')
![logo](https://docsify.js.org/_media/icon.svg ':size=100')

图片加链接 (Image + Link)：

[![](https://docsify.js.org/_media/icon.svg)](https://docsify.js.org/#/zh-cn/helpers "markdown")
```

---

## 列表

### 无序列表（减号）(-)

- 列表一
- 列表二

```markdown
- 列表一
- 列表二
```

### 无序列表（星号）(*)

* 列表一
* 列表二

```markdown
* 列表一
* 列表二
```

### 无序列表（加号和嵌套）(+)

+ 列表一
    + 列表二-1
    + 列表二-2

```markdown
+ 列表一
    + 列表二-1
    + 列表二-2
```

### 有序列表(-)

1. 第一行
2. 第二行

```markdown
1. 第一行
2. 第二行
```

### GFM任务列表

- [x] GFM task list 1
- [ ] GFM task list 3
    - [ ] GFM task list 3-1

```markdown
- [x] GFM task list 1
- [ ] GFM task list 3
    - [ ] GFM task list 3-1
```

---

## 绘制表格

| 项目        | 价格   |  数量  |
| --------   | -----:  | :----:  |
| 管线        |    $1    |  234  |

First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell

| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |

| Function name | Description                    |
| ------------- | ------------------------------ |
| `help()`      | Display the help window.       |

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| zebra stripes | are neat        |    $1 |

| Item      | Value |
| --------- | -----:|
| Pipe      |    $1 |

```markdown

| 项目        | 价格   |  数量  |
| --------   | -----:  | :----:  |
| 管线        |    $1    |  234  |

First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell

| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |

| Function name | Description                    |
| ------------- | ------------------------------ |
| `help()`      | Display the help window.       |

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| zebra stripes | are neat        |    $1 |

| Item      | Value |
| --------- | -----:|
| Pipe      |    $1 |

```

---

## 代码高亮

内置的代码高亮工具是 [Prism]，默认支持 CSS、JavaScript 和 HTML。如果需要高亮其语言——例如 PHP——可以手动引入代码高亮插件。  
使用方法
- 在入口文件引入js即可
- 引入方法：```code-languages，code-languages参考[Prism-Supported languages]

[Prism]:https://github.com/PrismJS/prism
[Prism-Supported languages]:https://prismjs.com/index.html#supported-languages

---

## 特殊符号

&copy; &  &uml; &trade; &iexcl; &pound;
&amp; &lt; &gt; &yen; &euro; &reg; &plusmn; &para; &sect; &brvbar; &macr; &laquo; &middot;

X&sup2; Y&sup3; &frac34; &frac14;  &times;  &divide;   &raquo;

18&ordm;C  &quot;  &apos;

```markdown
&copy; &  &uml; &trade; &iexcl; &pound;
&amp; &lt; &gt; &yen; &euro; &reg; &plusmn; &para; &sect; &brvbar; &macr; &laquo; &middot;

X&sup2; Y&sup3; &frac34; &frac14;  &times;  &divide;   &raquo;

18&ordm;C  &quot;  &apos;
```

## 反斜杠

\*literal asterisks\*

```markdown
\*literal asterisks\*
```

## 公式

插件：<https://katex.org/docs/supported.html>

---

$$
\frac{n!}{k!(n-k)!} = \binom{n}{k}
$$

$$
f(x) = \int_{-\infty}^\infty \hat f(\xi)\,e^{2 \pi x}\,d\xi
$$

$$ x^2+y^2 = 1 $$  

$$ a^2 + b^2 = c^2 $$

$$ f(x) = \displaystyle\sum_{n=0}^{n=100} n(n+1) $$

$$\sin(\alpha)^{\theta}=\sum_{i=0}^{n}(x^i + \cos(f))$$

```markdown
# 以下的内容前后需要$$包裹起来

\frac{n!}{k!(n-k)!} = \binom{n}{k}

f(x) = \int_{-\infty}^\infty \hat f(\xi)\,e^{2 \pi x}\,d\xi

x^2+y^2 = 1

a^2 + b^2 = c^2

f(x) = \displaystyle\sum_{n=0}^{n=100} n(n+1)

\sin(\alpha)^{\theta}=\sum_{i=0}^{n}(x^i + \cos(f))
```

---

## Emoji表情

编码格式

---

:star: :100: :cloud:

```markdown
:star: :100: :cloud:
```

---

非编码格式

🌹🍀🍎💰📱🌙🍁🍂🍃🌷💎🔪🔫🏀⚽⚡👄👍🔥

😀😁😂😃😄😅😆😉😊😋😎😍😘😗😙😚☺😇😐😑😶😏😣😥😮😯😪😫😴😌😛😜😝😒😓😔😕😲😷😖😞😟😤😢😭😦😧😨😬😰😱😳😵😡😠

👦👧👨👩👴👵👶👱👮👲👳👷👸💂🎅👰👼💆💇🙍🙎🙅🙆💁🙋🙇🙌🙏👤👥🚶🏃👯💃👫👬👭💏💑👪

💪👈👉☝👆👇✌✋👌👍👎✊👊👋👏👐✍

👣👀👂👃👅👄💋👓👔👕👖👗👘👙👚👛👜👝🎒💼👞👟👠👡👢👑👒🎩🎓💄💅💍🌂

📱📲📶📳📴☎📞📟📠

♻🏧🚮🚰♿🚹🚺🚻🚼🚾⚠🚸⛔🚫🚳🚭🚯🚱🚷🔞💈

🙈🙉🙊🐵🐒🐶🐕🐩🐺🐱😺😸😹😻😼😽🙀😿😾🐈🐯🐅🐆🐴🐎🐮🐂🐃🐄🐷🐖🐗🐽🐏🐑🐐🐪🐫🐘🐭🐁🐀🐹🐰🐇🐻🐨🐼🐾🐔🐓🐣🐤🐥🐦🐧🐸🐊🐢🐍🐲🐉🐳🐋🐬🐟🐠🐡🐙🐚🐌🐛🐜🐝🐞🦋

💐🌸💮🌹🌺🌻🌼🌷🌱🌲🌳🌴🌵🌾🌿🍀🍁🍂🍃

🌍🌎🌏🌐🌑🌒🌓🌔🌕🌖🌗🌘🌙🌚🌛🌜☀🌝🌞⭐🌟🌠☁⛅☔⚡❄🔥💧🌊

🍇🍈🍉🍊🍋🍌🍍🍎🍏🍐🍑🍒🍓🍅🍆🌽🍄🌰🍞🍖🍗🍔🍟🍕🍳🍲🍱🍘🍙🍚🍛🍜🍝🍠🍢🍣🍤🍥🍡🍦🍧🍨🍩🍪🎂🍰🍫🍬🍭🍮🍯🍼☕🍵🍶🍷🍸🍹🍺🍻🍴

🎪🎭🎨🎰🚣🛀🎫🏆⚽⚾🏀🏈🏉🎾🎱🎳⛳🎣🎽🎿🏂🏄🏇🏊🚴🚵🎯🎮🎲🎷🎸🎺🎻🎬

😈👿👹👺💀☠👻👽👾💣

🌋🗻🏠🏡🏢🏣🏤🏥🏦🏨🏩🏪🏫🏬🏭🏯🏰💒🗼🗽⛪⛲🌁🌃🌆🌇🌉🌌🎠🎡🎢🚂🚃🚄🚅🚆🚇🚈🚉🚊🚝🚞🚋🚌🚍🚎🚏🚐🚑🚒🚓🚔🚕🚖🚗🚘🚚🚛🚜🚲⛽🚨🚥🚦🚧⚓⛵🚤🚢✈💺🚁🚟🚠🚡🚀🎑🗿🛂🛃🛄🛅

💌💎🔪💈🚪🚽🚿🛁⌛⏳⌚⏰🎈🎉🎊🎎🎏🎐🎀🎁📯📻📱📲☎📞📟📠🔋🔌💻💽💾💿📀🎥📺📷📹📼🔍🔎🔬🔭📡💡🔦🏮📔📕📖📗📘📙📚📓📃📜📄📰📑🔖💰💴💵💶💷💸💳✉📧📨📩📤📥📦📫📪📬📭📮✏✒📝📁📂📅📆📇📈📉📊📋📌📍📎📏📐✂🔒🔓🔏🔐🔑🔨🔫🔧🔩🔗💉💊🚬🔮🚩🎌💦💨

♠♥♦♣🀄🎴🔇🔈🔉🔊📢📣💤💢💬💭♨🌀🔔🔕✡✝🔯📛🔰🔱⭕✅☑✔✖❌❎➕➖➗➰➿〽✳✴❇‼⁉❓❔❕❗©®™🎦🔅🔆💯🔠🔡🔢🔣🔤🅰🆎🅱🆑🆒🆓ℹ🆔Ⓜ🆕🆖🅾🆗🅿🆘🆙🆚🈁🈂🈷🈶🈯🉐🈹🈚🈲🉑🈸🈴🈳㊗㊙🈺🈵▪▫◻◼◽◾⬛⬜🔶🔷🔸🔹🔺🔻💠🔲🔳⚪⚫🔴🔵

🐁🐂🐅🐇🐉🐍🐎🐐🐒🐓🐕🐖

♈♉♊♋♌♍♎♏♐♑♒♓⛎

🕛🕧🕐🕜🕑🕝🕒🕞🕓🕟🕔🕠🕕🕡🕖🕢🕗🕣🕘🕤🕙🕥🕚🕦⌛⏳⌚⏰⏱⏲🕰

💘❤💓💔💕💖💗💙💚💛💜💝💞💟❣

💐🌸💮🌹🌺🌻🌼🌷🌱🌿🍀

🌿🍀🍁🍂🍃

🌑🌒🌓🌔🌕🌖🌗🌘🌙🌚🌛🌜🌝

🍇🍈🍉🍊🍋🍌🍍🍎🍏🍐🍑🍒🍓

💴💵💶💷💰💸💳

🚂🚃🚄🚅🚆🚇🚈🚉🚊🚝🚞🚋🚌🚍🚎🚏🚐🚑🚒🚓🚔🚕🚖🚗🚘🚚🚛🚜🚲⛽🚨🚥🚦🚧⚓⛵🚣🚤🚢✈💺🚁🚟🚠🚡🚀

🏠🏡🏢🏣🏤🏥🏦🏨🏩🏪🏫🏬🏭🏯🏰💒🗼🗽⛪🌆🌇🌉

📱📲☎📞📟📠🔋🔌💻💽💾💿📀🎥📺📷📹📼🔍🔎🔬🔭📡📔📕📖📗📘📙📚📓📃📜📄📰📑🔖💳✉📧📨📩📤📥📦📫📪📬📭📮✏✒📝📁📂📅📆📇📈📉📊📋📌📍📎📏📐✂🔒🔓🔏🔐🔑

⬆↗➡↘⬇↙⬅↖↕↔↩↪⤴⤵🔃🔄🔙🔚🔛🔜🔝

---

[Marked]: https://github.com/markedjs/marked/
[Markdown]: http://daringfireball.net/projects/markdown/