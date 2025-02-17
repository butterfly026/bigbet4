<template>
	<view :id="elId" class="v-tabs">
		<scroll-view id="scrollContainer" class="tabs_content" :scroll-x="scroll" :scroll-left="scroll ? scrollLeft : 0"
			:scroll-with-animation="scroll">
			<view class="v-tabs__container" :style="{
				display: scroll ? 'inline-flex' : 'flex',
				whiteSpace: scroll ? 'nowrap' : 'normal',
				background: bgColor, height,padding
			}">
				<view class="v-tabs__container-item" v-for="(v, i) in tabs" :key="i" :style="{
					color: current == i ? activeColor : color,
					fontSize: current == i ? fontSize : fontSize,
					fontWeight: bold && current == i ? 'bold' : '',
					justifyContent: !scroll ? 'center' : '',
					flex: scroll ? '' : 1,
					padding: paddingItem
				}" @click="change(i, v)">
					{{ field ? v[field] : v }}
				</view>
				<view v-if="!pills" class="v-tabs__container-line" :style="{
					background: lineColor,
					width: lineWidth + 'px',
					height: lineHeight,
					borderRadius: lineRadius,
					left: lineLeft + 'px',
					transform: `translateX(-${lineWidth / 2}px)`
				}"></view>
				<view v-else class="v-tabs__container-pills" :style="{
					background: pillsColor,
					borderRadius: pillsBorderRadius,
					left: pillsLeft + 'px',
					width: currentWidth + 'px',
					height
				}"></view>
			</view>
		</scroll-view>
		<view class="v-tabs__placeholder" :style="{ height: fixed ? height : '0', padding }"></view>
		<view class="tab-icon" :style="{ height: height }" @tap="handleOpenPopup" v-if="isShowMore">
			<image class="tab-icon-img" src="../image/tab-menu.png" mode="widthFix"></image>
		</view>
	</view>
</template>

<script>
/**
 * v-tabs
 * @property {Number} value 选中的下标
 * @property {Array} tabs tabs 列表
 * @property {String} bgColor = '#fff' 背景颜色
 * @property {String} color = '#333' 默认颜色
 * @property {String} activeColor = '#2979ff' 选中文字颜色
 * @property {String} fontSize = '28rpx' 默认文字大小
 * @property {String} activeFontSize = '28rpx' 选中文字大小
 * @property {Boolean} bold = [true | false] 选中文字是否加粗
 * @property {Boolean} scroll = [true | false] 是否滚动
 * @property {String} height = '60rpx' tab 的高度
 * @property {String} lineHeight = '10rpx' 下划线的高度
 * @property {String} lineColor = '#2979ff' 下划线的颜色
 * @property {Number} lineScale = 0.5 下划线的宽度缩放比例
 * @property {String} lineRadius = '10rpx' 下划线圆角
 * @property {Boolean} pills = [true | false] 是否胶囊样式
 * @property {String} pillsColor = '#2979ff' 胶囊背景色
 * @property {String} pillsBorderRadius = '10rpx' 胶囊圆角大小
 * @property {String} field 如果是对象，显示的键名
 * @property {Boolean} fixed = [true | false] 是否固定
 * @property {String} paddingItem = '0 22rpx' 选项的边距
 *
 * @event {Function(current)} change 改变标签触发
 */
export default {
	props: {
		value: {
			type: Number,
			default: 0
		},
		tabs: {
			type: Array,
			default() {
				return []
			}
		},
		bgColor: {
			type: String,
			default: '#f7f7f7'
		},
		padding: {
			type: String,
			default: '0'
		},
		color: {
			type: String,
			default: '#646566'
		},
		activeColor: {
			type: String,
			default: '#323233'
		},
		fontSize: {
			type: String,
			default: '28rpx'
		},
		activeFontSize: {
			type: String,
			default: '32rpx'
		},
		bold: {
			type: Boolean,
			default: true
		},
		scroll: {
			type: Boolean,
			default: true
		},
		height: {
			type: String,
			default: '90rpx'
		},
		lineColor: {
			type: String,
			default: 'var(--themeBtnBg)'
		},
		lineHeight: {
			type: String,
			default: '6rpx'
		},
		lineScale: {
			type: Number,
			default: 0.6
		},
		lineRadius: {
			type: String,
			default: '10rpx'
		},
		pills: {
			type: Boolean,
			deafult: false
		},
		pillsColor: {
			type: String,
			default: '#2979ff'
		},
		pillsBorderRadius: {
			type: String,
			default: '10rpx'
		},
		field: {
			type: String,
			default: ''
		},
		fixed: {
			type: Boolean,
			default: false
		},
		paddingItem: {
			type: String,
			default: '0 22rpx'
		},
		isShowMore: {
			type: Boolean,
			default: false
		}
	},
	data() {
		return {
			elId: '',
			lineWidth: 30,
			currentWidth: 0, // 当前选项的宽度
			lineLeft: 0, // 滑块距离左侧的位置
			pillsLeft: 0, // 胶囊距离左侧的位置
			scrollLeft: 0, // 距离左边的位置
			containerWidth: 0, // 容器的宽度
			current: 0 // 当前选中项
		}
	},
	watch: {
		value(newVal) {
			this.current = newVal
			this.$nextTick(() => {
				this.getTabItemWidth()
			})
		},
		current(newVal) {
			this.$emit('input', newVal)
		},
		tabs(newVal) {
			this.$nextTick(() => {
				this.getTabItemWidth()
			})
		}
	},
	methods: {
		// 产生随机字符串
		randomString(len) {
			len = len || 32
			let $chars =
				'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678' /****默认去掉了容易混淆的字符oOLl,9gq,Vv,Uu,I1****/
			let maxPos = $chars.length
			let pwd = ''
			for (let i = 0; i < len; i++) {
				pwd += $chars.charAt(Math.floor(Math.random() * maxPos))
			}
			return pwd
		},
		// 切换事件
		change(index, items) {
			if (this.current !== index) {
				this.current = index

				this.$emit('change', { index, ...items })
			}
		},
		// 获取左移动位置
		getTabItemWidth() {
			let query = uni
				.createSelectorQuery()
				// #ifndef MP-ALIPAY
				.in(this)
			// #endif
			// 获取容器的宽度
			query
				.select(`#scrollContainer`)
				.boundingClientRect((data) => {
					if (!this.containerWidth && data) {
						this.containerWidth = data.width
					}
				})
				.exec()
			// 获取所有的 tab-item 的宽度
			query
				.selectAll('.v-tabs__container-item')
				.boundingClientRect((data) => {
					if (!data) {
						return
					}
					let lineLeft = 0
					let currentWidth = 0
					if (data) {
						for (let i = 0; i < data.length; i++) {
							if (i < this.current) {
								lineLeft += data[i].width
							} else if (i == this.current) {
								currentWidth = data[i].width
							} else {
								break
							}
						}
					}
					// 当前滑块的宽度
					this.currentWidth = currentWidth
					// 缩放后的滑块宽度
					this.lineWidth = currentWidth * this.lineScale * 1
					// 滑块作移动的位置
					this.lineLeft = lineLeft + currentWidth / 2
					// 胶囊距离左侧的位置
					this.pillsLeft = lineLeft
					// 计算滚动的距离左侧的位置
					if (this.scroll) {
						this.scrollLeft = this.lineLeft - this.containerWidth / 2
					}
				})
				.exec()
		},
		// 打开更多操作
		handleOpenPopup() {
			this.$emit('handleOpenPopup', { ...this.tabs[this.current], index: this.current })
		},
	},
	mounted() {
		this.elId = 'xfjpeter_' + this.randomString()
		this.current = this.value
		this.$nextTick(() => {
			this.getTabItemWidth()
		})
	}
}
</script>

<style lang="scss" scoped>
.v-tabs {
	width: 100%;
	box-sizing: border-box;
	overflow: hidden;
	position: relative;
	display: flex;
	z-index: 93;
	align-items: center;

	::-webkit-scrollbar {
		display: none;
	}

	.tabs_content {
		width: calc(100% - 88rpx);
	}

	&__container {
		min-width: 100%;
		position: relative;
		display: inline-flex;
		align-items: center;
		white-space: nowrap;
		overflow: hidden;

		&-item {
			display: flex;
			align-items: center;
			height: 100%;
			position: relative;
			z-index: 10;
			// padding: 0 11px;
			transition: all 0.3s;
			white-space: nowrap;
		}

		&-line {
			position: absolute;
			bottom: 0;
			transition: all 0.3s linear;
		}

		&-pills {
			position: absolute;
			transition: all 0.3s linear;
			z-index: 9;
		}
	}
}

.tab-icon {
	z-index: 94;
	width: 100upx;
	// height: 60upx;
	text-align: center;
	background-color: #fff;
	border-left: 1upx solid #f2f2f2;
	box-shadow: 10upx 0 40upx 0 rgba(0, 0, 0, 0.3);

	.tab-icon-img {
		width: 80upx;
		height: 80upx;
		margin: 6rpx auto 0;
		background-size: 100% 100%;
	}
}</style>
