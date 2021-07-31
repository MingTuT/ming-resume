<template>
  <div id="app">
		<!-- 此按钮用来调试样式 -->
		<!-- <button type="button" @click="skip">跳过</button> -->
    <style-editor ref="styleEditor" :code="currentStyle"/>
		<resume-editor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"/>
	</div>
</template>

<script>
import StyleEditor from './components/StyleEditor.vue'
import ResumeEditor from './components/ResumeEditor'

export default {
  name: 'App',
  components: {
    StyleEditor,
		ResumeEditor
  },
  data() {
  	return {
		img: require('/public/user.png'),
		interval: 20,	// 文字出现的间隔时间
		currentStyle: '',	// 动态展示的样式
		enableHtml: false,
		timer: '',
		fullStyle: [
			`/*
* Hello，我是 Ming
* 准备找工作了，所以想写一份属于自己的简历
* 因此做了这个动态简历 ~(￣▽￣)~*
*/

/* 首先给所有元素加上过渡效果 */
* {
  transition: all .3s;
}
/* 白色背景太单调了，我们来点背景 */
html {
  color: rgb(222,222,222); background: rgb(0,43,54);
}
/* 文字离边框太近了 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* 代码高亮 */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* 加点 3D 效果呗 */
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

/* 接下来我给自己准备一个编辑器 */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 48vw; height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 好了，我开始写简历了 */

`,
`/* 这个简历好像差点什么
 * 对了，这是 Markdown 格式的，我需要变成对 HR 更友好的格式
 * 简单，用开源工具翻译成 HTML 就行了
 */`,
 `/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
	background: rgb(250,250,250)
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 4px solid rgb(204,204,204);
	padding-bottom: 10px;
  margin: 1em 0 .5em;
	width: 140px
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
  background: #f1f1f1;
	border-left: 5px solid #c7c7c7;
}
`
		],
		currentMarkdown: '',
		fullMarkdown: `<img src="user.png" > 林明辉
----

现在就读于 广东技术师范大学 ，是一名软件工程专业的大四学生

<img src="iconfont/skills.png" > 掌握技能
----

* Vue.js
* Uni-app
* Bootstrap + JQuery
* Node.js

<img src="project.png" > 项目经验
----

1. 仿蘑菇街购物商城
2. 仿豆瓣电影
3. 个人学习簿
4. 动态简历

* 证书
	- 软件水平考试 - 程序员
	- 大学英语四级

<img src="link.png" > 链接
----

* [邮箱] 409025631@qq.com
* [GitHub] https://github.com/MingTuT

> 热衷于学习新技术，做事认真，对前端有浓厚的兴趣。   
> 我希望能够加入一个以技术驱动为导向，技术氛围浓厚，有发展空间的互联网公司   
> 希望借此机会为贵司贡献自身所长
`
	}
  },
  created() {
  	this.makeResume()
  },
  methods: {
	  async makeResume() {
		  await this.progressivelyShowStyle(0)
		  await this.progressivelyShowResume()
		  await this.progressivelyShowStyle(1)
		  await this.showHtml()
		  await this.progressivelyShowStyle(2)
	  },
	  progressivelyShowStyle(n) {
		  return new Promise((resolve, reject) => {
			  let interval = this.interval
			  let showStyle = (async function() {
				  let style = this.fullStyle[n]
				  if(!style) { return }
				  // 计算前 n 个 style 的字符总数
				  let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
				  let prefixLength = length - style.length
				  if(this.currentStyle.length < length) {
					  let l = this.currentStyle.length - prefixLength
					  let char = style.substring(l, l + 1) || ' '
					  this.currentStyle += char
					  if(style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
						  this.$nextTick(() => this.$refs.styleEditor.goBottom())
					  }
					  this.timer = setTimeout(showStyle, interval)
				  }else {
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
				  if(this.currentMarkdown.length < length) {
					  this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
					  let preChar = this.currentMarkdown[this.currentMarkdown.length - 2]
					  if(preChar === '\n' && this.$refs.resumeEditor) {
						  this.$nextTick(() => this.$refs.resumeEditor.goBottom())
					  }
					  this.timer = setTimeout(showResume, interval)
				  }else {
					  resolve()
				  }
			  }
			  showResume()
		  })
	  },
	  showHtml() {
		  return new Promise((resolve, reject) => {
			  this.enableHtml = true
			  resolve()
		  })
	  },
		async skip() {
			clearTimeout(this.timer)
			await this.showHtml()
			this.currentStyle = this.fullStyle.reduce((pre, curr) => pre + curr)
			this.currentMarkdown = this.fullMarkdown
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
