<template>
  <div class="styleEditor" ref="container">
    <div class="code" v-html="codeInStyleTag"></div>
    <!-- 此div用于展示代码 -->
    <pre class="" v-html="highlightedCode"></pre>
    <!-- 此div用于展示高亮代码 -->
  </div>
</template>

<script>
  import Prism from 'prismjs'
  export default {
    name: 'Editor',
    props: ['code'],
    computed: {
      highlightedCode: function () {//给关键词外面套一个span eg:给body{} <span>body{}</span>
        return Prism.highlight(this.code, Prism.languages.css)//调用prism库 来高亮代码
      },
      codeInStyleTag: function () {
        return `<style>${this.code}</style>`
      }
    },
    methods: {
      goBottom() {
        this.$refs.container.scrollTop = 100000
      }
    }
  }

</script>

<style scoped>
  pre{
  }
  @media (max-width:500px){
    pre{
    }
  }
  .code {
    display: none;
  }
</style>
