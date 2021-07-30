<template>
	<div class="styleEditor" ref="container">
		<div class="code" v-html="codeInStyleTag"></div>
		<pre v-html="highlightedCode"></pre>
	</div>
</template>

<script>
	import Prism from 'prismjs'
	export default {
		name: 'Editor',
		props:['code'],
		computed: {
			codeInStyleTag() {
				return `<style>${this.code}</style>`
			},
			highlightedCode() {
				return Prism.highlight(this.code, Prism.languages.css)
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
