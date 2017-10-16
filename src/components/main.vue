<template>
	<div id="main">
		<StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
		<ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
	</div>
</template>

<script>
	import StyleEditor from './StyleEditor'
	import ResumeEditor from './ResumeEditor'
	import './../assets/reset.css'
	export default {
		name: 'main',
		components: {
			StyleEditor,
			ResumeEditor
		},
		data() {
			return {
				interval: 5,
				currentStyle: '',
				enableHtml: false,
				fullStyle: [
					`
/*
 * 大家好，我是杨玉峰
 * 现在是找工作的好季节，好多公司都在招聘，你是不是也在准备简历呀。
 * 说做就做，我也来写一份简历！
 */
/* 首先给所有元素加上过渡效果 */
* {
  transition: all .3s;
  -webkit-transition: all .3s;
  -moz-transition: all .3s;
}
/* 白色背景太单调了，我们来点背景 */
html {
  color: rgb(222,222,222); 
  background:rgb(0,43,54);
}
/* 文字离边框太近了 */
.styleEditor {
	padding: .5em;
	border: 1px solid;
	margin: .5em;
	overflow: auto;
	width: 45vw; 
	height: 90vh;
}
/* 代码高亮 */
.token.comment{
	color:#a9a1a1;
}
.token.selector{
	color: rgb(90, 101, 11); 
}
.token.property{
	color: rgb(187,137,0); 
}
.token.punctuation{
	color: yellow; 
}
.token.function{
	color: rgb(42,161,152); 
}
.styleEditor pre{
	color: #504949;
}

/* 加点 3D 效果呗 */
html{
    perspective: 1000px;
}
.styleEditor {
    position: fixed; 
    left: 0; 
    top: 0;
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
					`
/* 这个简历好像差点什么
 * 对了，这是 Markdown 格式的，我需要变成对 HR 更友好的格式
 * 简单，用开源工具翻译成 HTML 就行了
 */
`,
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
.resumeEditor table{
	width:500px;
}
.resumeEditor table tr th,
.resumeEditor table tr td{
	border:1px solid;	
	text-align:center;
	height:50px;
	line-height:50px;
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
`
				],
				currentMarkdown: '',
				fullMarkdown: `
## 杨玉峰

出生日期 | 联系方式 | 所在地
---|--- |--- 
1996.04.23| 15871483684 | 武汉

## 工作经历

- 2016-2017：xxxx
- 2016-2017：xxxx
- 2016-2017：xxxx

## 专业技能

1. 精通html,js,css和h5的规范以及各浏览器之间的兼容性;
2. 精通jquery,bootstrap,juqery mobbile的主流框架；
3. 熟悉node.js,vue.js,angular.js;

## 教育背景

武软

## 个人评价

1. a
2. b
3. c
4. d

## 链接

[github](https://github.com/yyf1539558157?tab=repositories)
`
			}
		},
		created() {
			this.makeResume();
		},
		methods: {
			makeResume: async function() {
				//写第一段css代码
				await this.progressivelyShowStyle(0);
				//写简历
				await this.progressivelyShowResume();
				//写第二段css代码
				await this.progressivelyShowStyle(1);
				//用markdown展示简历
				await this.showResumeHtml();
				//写第三段css代码(给简历加样式)
				await this.progressivelyShowStyle(2);
			},
			//逐步显示style样式
			//有顺序的完成  同步回调
			//写css代码
			progressivelyShowStyle(n) {
				return new Promise((resolve, reject) => {
					//书写速度
					let interval = this.interval;
					//显示代码
					let showStyle = (async function() {
						//书写第n个样式代码段
						let style = this.fullStyle[n];
						if(!style) {
							return;
						}
						//计算前 n 个 style 的字符总数
						let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0);
						let prefixLength = length - style.length;
						if(this.currentStyle.length < length) {
							let l = this.currentStyle.length - prefixLength
							let char = style.substring(l, l + 1) || ' '
							this.currentStyle += char
							if(style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
								this.$nextTick(() => {
									this.$refs.styleEditor.goBottom()
								})
							}
							setTimeout(showStyle, interval)
						} else {
							resolve()
						}

					}).bind(this);
					showStyle();

				})
			},
			progressivelyShowResume() {
				return new Promise((resovle, reject) => {
					//markdown的代码长度
					let length = this.fullMarkdown.length;
					//书写速度
					let interval = this.interval;
					let showResume=()=>{
						let currMarkLen=this.currentMarkdown.length;
						if(currMarkLen<length){
							this.currentMarkdown=this.fullMarkdown.substring(0,currMarkLen+1);
							let char = this.currentMarkdown[currMarkLen]              				
              				if(char==="\n"&&this.$refs.resumeEditor){
              					console.log(char);
              					this.$nextTick(()=>{
              						this.$refs.resumeEditor.goBottom();
              					})
              				}           				
              				setTimeout(showResume, interval);
						}else{
							resovle();
						}
					}
					showResume();

				})
			},
			showResumeHtml(){
				return new Promise((resovle,reject)=>{				
					this.enableHtml=true;
					resovle();										
				})
			}
		}
	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>