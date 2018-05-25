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
        interval: 1,//多久执行
        currentStyle: '',//当前用户看到的左边的代码 不停的修改其
        enableHtml: false,//是否开启html写
        //一行一行把fullstyle拷给currentstyle
        fullStyle: [
          `/*
* 思路源于 http://strml.net/
* 嗨，你好啊，我是王煜程
*..........
* 啊，当前页面好空啊
* 没办法，那只能重新造一个网页了╮(╯_╰)╭
*/

/* 首先给所有元素加上过渡效果 */
* {
  transition: all .3s;
}
/* 白色背景太单调了，太low了，我们来点背景吧 */
html {
  color: rgb(222,222,222); background:#545454;
}
/* 文字离边框太近了 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 30vw; height: 90vh;
}
/* 代码颜色有点单调吧，我把代码高亮一下~ */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* 接下来我给自己准备一个编辑器用来介绍一下自己 */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 任务完成！那给新模块让个地方把！ */
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
/* 好了，我开始写简历了 */


`,
          `
/* 这个简历好像有点问题
 * 对了，只是 Markdown 格式的，我需要给它变换一下
 * 简单，用开源工具翻译成 HTML 就行了
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.div1 img{
    display:none; 
}
.div1:hover img{
    display:block; 
} 
.div1{
  margin-bottom:2em;
}
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
        fullMarkdown: `        
我的技能
--------

* HTML&CSS
* jQuery
* JavaScript
* ES6,vue
* PHP（学习中）
* nodejs(学习中)

个人项目
----

[1. 动态生成简历页面](#)


<pre>通过es6模板字符串和代码substring（），setinterval()等来将代码定时的输入在页面和编辑器中，展现出边写代码边实现效果的样式，并通过判断输出
字符串是否未回车来实现页面下移功能。</pre>



[2. 去哪儿网App](#)


<pre>基于Vue全家桶实现vue2 +vue-router+ es6 +webpack，使用iconfont矢量图标，高仿去哪儿网页面app，实现旅游网首页，旅游网站城市列表
页面开发，和旅游网站详情页面的功能。</pre>



[3. 电商旅游App](#)    


<pre>基于Vue全家桶实现vue2 +vue-router+ es6 +webpack,并使用iview UI 组件库，实现旅游详情页，游记攻略页面等的功能
后端采用PHP +CodeIgniter 实现注册，登陆，评论，购物信息，旅游信息等数据的传输。</pre>


[4. css照片墙](#)


<pre>使用css3中transform:的translate值来实现图片进入，旋转效果功能，并使用JavaScript来实现图片分散，合并，显示顺序
的逻辑效果</pre>



[5. 吃豆人游戏](#)


<pre>应用技术/框架：canvas+JavaScript，利用canvas实现画布的绘制，并利用js实现人物移动，实现食物接触判断,吃食物加分，
随机生成食物功能等。</pre>


链接
----


* [我的个人主页]（）
* [GitHub](https://github.com/gloriousch)
* [CSDN](https://blog.csdn.net/weixin_39173093)


请联系我
-------

* Tel:18846433516
* QQ:77961829
* E-mail:77961829@qq.com
`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)//等待前面一个await执行后接着执行下一个await，以此类推
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
              if (prevChar === '\n' && this.$refs.resumeEditor)
              //判断如果上一次打的字符是回车
               {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
                //$nextTick 下次HTML渲染之前 也就是下一次打打代码之前  让resumeEditor运行goBottom
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
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
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
