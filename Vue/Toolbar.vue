<style lang="less">
	@bgGray: #333;
	.vi_toolbar {
		position: fixed;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: @bgGray;
		z-index: 99999;
		color: white;
		padding: 5px 10px;
		cursor: auto;

		.vi_close{
			position: absolute;
			right: 10px;
			font-size: 14px;
			top: 0;
			bottom: 0;
			line-height: 38px;
			width: 20px;
			text-align: center;
			cursor: pointer;
		}

		.vi_tit {
			font-weight: bold;
		}

		.vi_color {
			vertical-align: middle;
			border: none;
			appearance: textfield !important;
			outline: none;
			width: 50px;
			height: 28px;
			margin: 0;
			padding: 0;
		}

		.vi_text {
			width: 32px;
			height: 18px;
			padding: 0 2px;
			padding-right: 15px;
			text-align: right;

			color: #fff !important;
			background: #474747 !important;
			border: 1px solid #666 !important;
			appearance: textfield !important;

			&::-webkit-outer-spin-button,
			&::-webkit-inner-spin-button {
				display: none;
			}
		}

		.vi_tbItem {
			display: inline-block;
			white-space: nowrap;
			padding: 0 5px;

			* {
				vertical-align: middle;
			}

			input[type=range]{
				width: 80px;
			}

			button{
				padding: 2px 5px !important;
				width: auto !important;
				height: auto !important;
				font-size: 12px !important;
				border-radius: 3px;
				line-height: 16px;
				cursor: default;
				color: buttontext;
				background-color: buttonface;
			}
		}
		.vi_snap .vi_tbItem{
			position: relative;

			&::after{
				content: 'px';
				color: gray;
				position: absolute;
				right: 7px;
				top: 0;
				bottom: 0;
				line-height: 20px;
			}
		}

		::-webkit-color-swatch-wrapper {
			border: 1px solid @bgGray;
			background-color: @bgGray;
		}
		::-webkit-color-swatch {
			/*position: relative;*/
		}

		&.vi_toolbar_ff{
			.vi_color {
				height: 20px;
			}
		}
	}
</style>

<template>
	<div class="vi_toolbar" :class="{vi_toolbar_ff: isFF}" @mousemove.stop @touchmove.stop>
		<div class="vi_tbItem">
			<span class="vi_tit">颜色：</span>
			<input type="color" class="vi_color" v-model="$parent.bgColor"/>
		</div>

		<div class="vi_tbItem">
			<span class="vi_tit">背景透明度：</span>
			<input v-blur type="range" min=".1" max="1" step=".1" v-model="$parent.bgOpacity" />
		</div>

		<div class="vi_tbItem vi_snap">
			<div class="vi_tbItem">
				<span class="vi_tit">边界吸附：</span>
				<input type="number" min="0" max="200" class="vi_text" v-validate v-model="$parent.snapToLine"/>
			</div>

			<div class="vi_tbItem">
				<span class="vi_tit">顶点吸附：</span>
				<input type="number" min="0" max="100" class="vi_text" v-validate v-model="$parent.snapToAngle"/>
			</div>
		</div>

		<div class="vi_tbItem">
			<button @click="reset" title="重置所有设置项到默认值">重置</button>
		</div>

		<span class="vi_close" @click="$parent.showToolbar = false">X</span>
	</div>
</template>

<script>
	import {isFF} from "../utils";

	export default {
		name: "Toolbar",

		data() {
			return {
				isFF
			}
		},

		methods: {
			reset(){
				this.$parent.bgColor = '#1171cd';
				this.$parent.bgOpacity = .3;
				this.$parent.snapToLine = 50;
				this.$parent.snapToAngle = 15;
			},

			watchData() {
				let {bgColor, bgOpacity, snapToAngle, snapToLine} = this.$parent.$data;
				let dataArr = [bgColor, bgOpacity, snapToAngle, snapToLine];
				let ret = 0;
				this.$watch(function (){
					let {bgColor, bgOpacity, snapToAngle, snapToLine} = this.$parent.$data;
					let newDataArr = [bgColor, bgOpacity, snapToAngle, snapToLine];
					let result = dataArr.every(item => newDataArr.every(newItem => newItem === item) );
					if (!result) {
						// console.log('changed', bgColor, bgOpacity, snapToAngle, snapToLine);
						ret = Math.random();
						dataArr = [bgColor, bgOpacity, snapToAngle, snapToLine];
					}
					return ret;
				}, this.saveData)
			},

			saveData() {
				let {bgColor, bgOpacity, snapToAngle, snapToLine} = this.$parent.$data;
				chrome.storage.local.set({bgColor, bgOpacity, snapToAngle, snapToLine});
			},
		},

		directives: {
			validate: {
				bind(el) {
					el._validate = function (){
						el.value = +el.value || 0;

						if (el.value > +el.max || el.value < +el.min) {
							if (el.value > +el.max) {
								el.value = el.max
							} else if (el.value < +el.min) {
								el.value = el.min
							}

							let event = document.createEvent('HTMLEvents');
							event.initEvent("input", true, true); // 3个参数：事件类型，是否冒泡，是否阻止浏览器的默认行为
							el.dispatchEvent(event);
						}
					};

					el.addEventListener('change', el._validate);
				},
				unbind(el) {
					el.removeEventListener('change', el._validate);
					delete el._validate;
				}
			},
			blur: {
				bind(el) {
					el.onfocus = el.blur.bind(el);
				},
				unbind(el) {
					delete el.onfocus;
				}
			}
		},

		created() {
			this.watchData();
		}
	}
</script>

