### 序言

最近做网易云信迁移至腾讯云信，翻看了一下他们的demo及源码，发现他们的emoji功能其实实现得很简陋的

* 预览只是文字，没效果
* 只能尾部添加，不能在光标位置插入
* 如果在H5与APP端通信时，必须得双端都套用
* 如果想推送到微信公众号，不好意思，解析不了，显示文字

一番Google下来发现网上的大多都是拿着网易emoji的实现思路来显摆。。。

这里自己设计并实现了一版，上面涉及的问题均已解决

> 注：1、这里涉及到了一些偏门知识点，比如Unicode码、光标API等，相关内容可以在MDN上查到
>
> ​        2、Githut预览及源码下期给出

<br >

<br >

### 设计方案一

最开始的设计思路是既然emoji表情有对应的Unicode码，那么是不是可以直接就使用Unicode码来进行传输及显示，直接不做任何处理，基于各平台对Unicode的支持，让其自生自灭呢？想着就干，尝试一波再说。。。

看到这里如果真的动手去尝试了么估计你是真的没有好好了解一波emoji的Unicode码，只要进入官网或者各统计平台应该应该都可以看到这个东西

![emojipic](https://qiniu-app.qtshe.com/1562028654596.png)

<br >

由上图可以看出个移动端对emoji的支持都存在如此大的差异，那么PC端的差距不看也知道不忍直视了，当然可能基于好奇和专研的精神，个别还是想去尝试一波，断绝念头。。。好比如我

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		
		<style type="text/css">
			.emoji-con_span {
				display: block;
				width: 300px;
				background-color: #cee6ff;
				color: #282828;
				border-radius: 4px;
				padding: 8px 12px;
				box-sizing: border-box;
			}
		</style>
	</head>
	<body>
		<span class="emoji-con_span">emoji：😂😍😘</span>
	</body>
</html>
```


你可以将如上代码运行于各浏览器，你可能看到这个效果

1. chrome <br >
![emoji1](https://cdn.linker.cc/iyunting2019-07-02/21/1562073532905.png )
3. Firefox <br >
![emoji2](https://cdn.linker.cc/iyunting2019-07-02/21/1562073630419.png )
3. IE <br >
![emoji2](https://cdn.linker.cc/iyunting2019-07-02/21/1562073838904.png )
4. Edge
![emoji4](https://cdn.linker.cc/iyunting2019-07-02/21/1562074000582.png )<br >
5. Android
![emoji5](https://cdn.linker.cc/iyunting2019-07-02/21/1562074787893.jpg )<br >

由上可以看出各端的差距主要在移动端和IE之间，特别是移动端和PC端的差距，几乎改头换面了，这不说产品，自己这关很明显都过不了，况且IE非edge模式下还是黑白色。所以这个设计方案很明显被pass掉了

<br >

<br >

### 设计方案二

方案一的失败之处很明显，PC端各平台兼容性差而且样式和移动端相比low的不行，最好的显示效果很明显是iOS端的emoji表情图标。基于此既然我们不能统一使用Unicode码来显示，但是我们却可以使用它来传输和存储（不论云信、还是数据库都能解析），所以我们唯一需要解决的问题就是显示而已，既然如此，那么设计思路就很清晰了

不论是各端怎么传输，还是数据库存储都直接使用Unicode来进行，APP端直接使用系统自带的emoji支持来显示，PC端则使用切图将Unicode正则匹配替换来显示，即只需要做一个字典来进行正则替换就OK

有兴趣不防撸撸代码尝试一下：

```vue
<template>
  <div id="chat" class="chat_page_con">
    
    <!-- 使用富文本输入框模式 -->
    <div>Ctrl + Enter 发送消息：</div>
    <div 
      id="charInput" 
      @click="saveRangeLocal"
      @focus="saveRangeLocal" 
      @keyup="inputSend"
      @input="saveRangeLocal" 
      class="chatframe_input_con scrollbar" 
      contenteditable="true">
    </div>

    <!-- 表情选择器 -->
    <div class="chatframe-icon">
      <el-popover
        placement="bottom-start"
        width="400"
        trigger="click">
        <el-tabs tab-position="bottom" value="Emotions" class="emoji_tabs_box">
          <el-tab-pane 
            v-for="(emojiCon, emojiKey, eInd) in emojiIcon" 
            :key="emojiKey"
            :label="emojiKey" 
            :name="emojiKey"
          >

            <!-- 这里的label icon不能放到json配置文件中，因为icon放到配置文件中后无法渲染出来 -->
            <!-- 这里很low可以自己修改，试用图片来替换，使用同样的图片加载方法保存在配置中 -->
            <span v-if="eInd == 0"  slot="label" class="iconfont emoji_pane_tab">&#xe612;</span>
            <span v-if="eInd == 1"  slot="label" class="iconfont emoji_pane_tab">&#xe609;</span>
            <span v-if="eInd == 2"  slot="label" class="iconfont emoji_pane_tab">&#xe62d;</span>
            <span v-if="eInd == 3"  slot="label" class="iconfont emoji_pane_tab">&#xe63c;</span>
            <img 
              v-for="(emoItem, emoInd) in emojiCon" 
              :key="emoInd" 
              :src="getIconPic(emoItem.unicode)" 
              alt="X"
              @click="sendEmojiIcon(emoItem.unicode)"
              class="chat_emoji_item">
          </el-tab-pane>
        </el-tabs>
        <el-button 
          class="iconfont open_emoji_icon" 
          type="text" 
          slot="reference">&#xe612;
        </el-button>
      </el-popover>
    </div>

    <!-- 显示内容区 -->
    <div>发送的消息：</div>
    <div class="chatframe-text text_emoji" v-html="changeEmojiCon(sendValue)"></div>
  </div>
</template>
```

<br >

页面代码设计相对简单，可以根据自己需求选取，重点是接下来的JS方法解读（只列出个别重要一点的方法做解释）

1. **定义data存储字段**

   ```javascript
   data () {
     return {
       emojiIcon: emojiData.icon,    // 导入的emoji表情配置文件内容
       emojiPath: new Map(),         // emoji表情地址map对象,
       inputRange: '',               // 光标
       sendValue: '',                // 发出的内容
     }
   }
   ```

   这其中涉及到一个字段为inputRange（光标），这里解释一下，因为插入emoji表情时需要动态的往用户编辑的当前光标处去插入emoji，所以不管用户是输入还是点击都需要获取一下当前光标，然后将其记录下来，当点击emoji表情是就将其插入到记录的光标处

   <br >

2. **记录光标**

   ```javascript
   // 延时记录光标到位置
   saveRangeLocal () {
     setTimeout(() => {
       this.inputRange = window.getSelection().getRangeAt(0)
     }, 0)
   }
   ```

   获取光标保存时并不能直接就实时获取，否则会所以报错或undefined，所以这里使用宏任务来进行延时获取

   <br >

3. **插入表情到富文本**

   ```javascript
   // 点击表情，将表情添加到输入框
   sendEmojiIcon (code) {
     let inputNode = document.getElementById('charInput')
     let html = "<img src='"+ this.getIconPic(code) +"' unicode = '" + code + "' alt='' >"
     let sel = window.getSelection()
     let range = this.inputRange
     let el = document.createElement("div")
     let frag = document.createDocumentFragment(), node, lastNode
   
     if (!inputNode) {
       return
     }
   
     if (!range) {
       inputNode.focus()
       range = window.getSelection().getRangeAt(0)
     }
   
     range.deleteContents()
     el.innerHTML = html
   
     while ((node = el.firstChild)) {
       lastNode = frag.appendChild(node)
     }
     range.insertNode(frag)
     
     if (lastNode) {
       range = range.cloneRange()
       range.setStartAfter(lastNode)
       range.collapse(true)
       sel.removeAllRanges()
       sel.addRange(range)
     }
   }
   ```

   输入框使用的是输入框使用富文本模式，所以点击emoji表情时都是传入Unicode值，字典匹配，插入对应的表情图片，再插入的image标签上添加一个Unicode属性，用于解析时字典对比替换

   <br >

4. **图片Emoji转换及发送消息**

   ```javascript
   // 将输入框中的图片替换为emoji表情
   formatInputCon () {
     let inputValue = document.getElementById('charInput').innerHTML
   
     inputValue = inputValue.replace(/<img.*?(?:>|\/>)/gi, (val) => {
       let unicode = val.match(/unicode=[\'\"]?([^\'\"]*)[\'\"]?/i)[1]
       let icon = this.emojiIcon
       let iPic = ''
   
       // 遍历查找Unicode表情
       for (const key in icon) {
         if (icon.hasOwnProperty(key)) {
           const iType = icon[key]
           let flag = false
   
           for (let index = 0; index < iType.length; index++) {
             const element = iType[index]
   
             if (element.unicode == unicode) {
               iPic = element.emoji
               flag = true
               break
             }
           }
   
           if (flag) {break}
         }
       }
   
       return iPic
     })
   
     return inputValue
   }
   ```

   ```javascript
   // 发送消息
   inputSend (e) {
     if ((e.ctrlKey && e.keyCode == 13) || (e.ctrlKey && e.keyCode == 108)) {
       // 提交的时候使用正则替换，将br换位换行符，不能使用innertext转，有兼容性问题
       this.sendValue = this.formatInputCon().replace(/<br>/g, '\r\n')
     } 
   }
   ```

   发送消息之前需要将输入框中的Emoji图片转换为相应的Emoji的Unicode值，同时由于使用的是富文本编辑器，所以拿到的内容中换行都是以‘br’标签来实现的，转换文本时需要替换为’\r\n‘

   <br >

5. **Emoji到图片转换**

   ```javascript
   // 将emoji表情转换为图片
   changeEmojiCon (str) {
     let patt = /[\ud800-\udbff][\udc00-\udfff]/g    // 检测utf16字符正则
   
     str = str.replace(patt, (char) => {
       let H, L, code
       
       if (char.length === 2) {
         H = char.charCodeAt(0)   // 取出高位
         L = char.charCodeAt(1)   // 取出低位
         code = (H - 0xD800) * 0x400 + 0x10000 + L - 0xDC00   // 转换算法
         return "&#" + code + ";"
       } else {
         return char
       }
     })
   
     str = str.replace(/&#{1}[0-9]+;{1}/ig, (a) => {
       let unicode = a.replace(/^&#{1}/ig, '')
       
       unicode = unicode.replace(/;{1}$/ig, '')
       unicode = 'U+' + (parseFloat(unicode).toString(16).toUpperCase())
   
       return "<img src='"+ this.getIconPic(unicode) +"'/>"
     })
   
     return str
   }
   ```

   拿到消息体时需要将其中的Emoji的Unicode码值转换为图片，最后以InnerHTML的方式插入到显示器

