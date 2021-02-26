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
        interval: 40,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* Inspired by http://strml.net/
* 你好啊，我是秘小乐 hi，I am Tye, aka Lil-happy
* 这是我的自我介绍 This is my introduction
* 我是一名前端工程师 I am a front-end engineer
* 话不多说，现在开始！Let us begin now！
*/

/* 首先给所有元素加上过渡效果 First add a transition effect to all elements */
* {
  transition: all .3s;
}
/* 白色背景太单调了，我们来点背景  Add background */
html {
  color: rgb(255,255,255); background: rgb(28,27,39);
}
/* 文字离边框太近了 Add padding */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* 代码高亮 Code highlighting */
.token.selector{ color: rgb(164,220,63); }
.token.property{ color: rgb(107,217,238); }
.token.punctuation{ color: rgb(255,255,255); }
.token.function{ color: rgb(107,217,238); }
.token.comment{ color: rgb(115,111,94); }

/* 加点 3D 效果呗 Add 3D to make it look good */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
          transform: rotateY(10deg) translateZ(-100px) ;
}

/* 接下来我给自己准备一个编辑器 And now, I'll prepare an editor */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 48vw; height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 好了，我开始写我的经历了 Okey！ I'm going to start */


`,
          `
/* 好像差点什么 It's seems that no one likes an ugly format
 * 对了，这是 Markdown 格式的，我需要变成对阅读者更友好的格式 Let me make it beautiful
 * 简单，用开源工具翻译成 HTML 就行了  Okey！
 */
`
          ,
          `
/* 再对 HTML 加点样式  Finally, add some styles */
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
        fullMarkdown: `About me
----

* 电话 TEL: (+86)15931896736
* Email: duantengyuan@huami.com
* 定位: 北京市海淀区中关村壹号B2楼7F-106 
* Location: 106, 7th B2, No.1 Zhongguancun,Beijing 
* [Map](https://huami.feishu.cn/sheets/shtcnGXaE6aoGpubRaKzsBEuBqq)
* 部门: 大数据及云平台部 - Web应用部 - 小程序组 
* Department: Bigdata & Cloud - Web Application R&D - Applet Group


What's my job?
----

1. Zepp活力PAI小程序 (PAI mini-program)
2. Zepp健康习惯养成小程序 (Habit mini-program)
2. Zepp健康小程序 (Zepp mini-program)

Experience
----

* 2020 年 3 月作为前端开发工程师加入华米。
* Joined Huami as a front-end engineer in March 2020.
* 2021年是我在公司的第 2 年
* 2021 is my second year in the company. 
* 快乐工作，共同进步是我的工作口号
* Happy work and common progress are my work slogans. 

> end

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

<style scoped>
  #app {
    font-family: Helvetica, 'Hiragino Sans GB', 'Microsoft Yahei', '微软雅黑', Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }
</style>
