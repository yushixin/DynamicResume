<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
    
  </div>
</template>

<script>
import StyleEditor from './components/StyleEditor'
import ResumeEditor from './components/ResumeEditor'
import './assets/reset.css'

export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 10,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* 你好 我是于士心 isaac yu
* 点子是在网上看到的，就想做一个，丰富一下自己的简历
* 那么现在让我们开始
*/
* {
  -webkit-transition: all .3s;
  transition: all .3s;
}
/* 加个背景颜色看着比较舒服 */
html {
  color: rgb(222,222,222);
  background: #3b3b3b; 
}
/* 调整一下布局 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* 这里用了网上找的代码高亮 */
.token.selector{ color: rgb(230,28,100); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }
/* 现在在右边加一个编辑器 */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 48vw; height: 90vh; 
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 好的，接下来是我的自我介绍 */
`,
          `
 * 这是 Markdown 格式的
 * 接下来用开源工具翻译成 HTML 
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;            
  content: counters(section, ".") " ";  
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `于士心 -- 计算机科学技术学院 -- 黑龙江大学2018年毕业生

----
我是一年前开始关注前端技术，目前已经经过一年的学习;2018年毕业现在正在找 实习/全职 的工作;
----
* 熟练掌握原生javascript
* 熟练掌握HTML(5)/CSS(3)
* 了解移动端web app开发、响应式设计（rem + 弹性盒模型）
* 熟练掌握前端框架VUE、了解reactJS
* 了解github，有在项目中使用github做版本控制工具的经验
* 熟悉php，熟悉CI框架
* 了解基本数据结构与算法，具有一定的软件工程意识

项目经验
----
猫言猫语webapp
猫言猫语是一个前后端分离项目，前端VUE，EXPRESS做的中间层 , 后台PHP;
使用技术：CSS HTML JavaScript jQUery PHP vue-cli框架 CI框架 Express;

在项目中独立完成登录、注册页面的编写。使用前端vue、中间层EXPRESS、后端三层结构。
    主要使用css、html完成静态页面的编写，JQ完成页面逻辑，PHP完成后端。
    使用vue-router完成页面间的跳转，使用axios、request完成前端页面、中间层、后端CI间的数据交互。
与项目成员共同完成了后台管理页面的编写，后台管理页面
    使用PHP框架。其中我负责后台管理页面的动态管理功能模块，该模块包括：主页专题的推送，用户文章的管理。

链接
----
* [我的GitHub](https://github.com/yushixin)
* [我的简书](http://www.jianshu.com/u/0efe075b3616)
> Email: wysx95@163.com 
> TEL: 18545577154
`
      }
    },
    created() {
      this.makeResume()
    },
    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              console.log(prevChar)
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
html {
  min-height: 100vh;
}
* {
  -webkit-transition: all .2s;
  transition: .2s;
}
</style>
