<template>
	<view class="comp-container">
		<slot></slot>
		<view id="hk-video" class="hk-video">
		</view>
		<!-- 顶部显示监控点名称 -->
		<view class="name-view" :class="{show:showOptBar}">
			{{curCameraName}}
		</view>
		<!-- 底部操作 -->
		<view class="control-view" :class="{show:showOptBar}">
			<view class="left">
				<!-- 停止播放 -->
				<image class="icon stop-icon" :src="stopIcon" mode="widthFix" @click="handleStop"></image>
			</view>
			<view class="right">
				<!-- 声音 -->
				<image class="icon sound-icon" :src="hasSound ? soundIcon:muteIcon" mode="widthFix"
					@click="handleSound"></image>

				<!-- 全屏 -->
				<image class="icon full-icon" :src="fullScreenIcon" mode="widthFix" @click="singleFullScreen"></image>
			</view>

		</view>
	</view>
</template>
<script>
	// const IS_MOVE_DEVICE = document.body.clientWidth < 992 // 是否移动设备
	import stopIcon from './images/stop.png'
	import fullScreenIcon from './images/full_screen.png'
	import soundIcon from './images/sound.png'
	import muteIcon from './images/mute.png'
	export default {
		props: {
			//默认分屏，2:2x2布局
			currentSplit: {
				type: Number,
				default: 2
			},
			//最大分屏，4:4x4布局
			maxSplit: {
				type: Number,
				default: 4
			},
			jsBasePath: {
				type: String,
				default: '/static/lib'
			},
			//当前处于预览状态的窗口信息，包含窗口号，监控点编码
			previewWindList: {
				type: Array,
				default: () => []
			}
		},
		data() {
			return {
				stopIcon,
				fullScreenIcon,
				soundIcon,
				muteIcon,
				player: null,
				curWindIndex: 0, //当前窗口索引1开始
				curPlayWinds: [], //当前正在播放的窗口
				showOptBar: false, //是否显示操作栏
				interval: null,
				isChangeLayout: false,
				curCameraName: '', //当前窗口设备名称
				hasSound: false, //是否有声音


			}
		},
		watch: {

		},
		computed: {

		},
		mounted() {
			this.init()
			this.onWindowEvent()
		},
		methods: {
			init() {
				//实例化
				this.player = new window.JSPlugin({
					szId: 'hk-video',
					szBasePath: this.jsBasePath,
					iMaxSplit: this.maxSplit || 4,
					iCurrentSplit: this.currentSplit || 1,
					openDebug: true,
					oStyle: {
						borderSelect: '#FFCC00',
						background: '#4C4C4C'
					}
				})
				// 事件回调绑定
				this.player.JS_SetWindowControlCallback({
					windowEventSelect: iWndIndex => { //插件选中窗口回调
						if (this.curWindIndex == iWndIndex && this.curPlayWinds.find(item => item
								.windowIndex === iWndIndex) && !this.interval && !this
							.isChangeLayout) { //第二次点击窗口号，且当前窗口处于播放状态显示操作栏
							this.showOptBar = true
							this.curCameraName = this.getCurCameraName()
							this.interval = setTimeout(() => {
								this.showOptBar = false
								this.interval = null
							}, 5000)
						} else if (this.curWindIndex != iWndIndex) {
							this.curWindIndex = iWndIndex
							this.$emit('windowChange', this.curWindIndex)
						}
					},
					pluginErrorHandler: (iWndIndex, iErrorCode, oError) => { //插件错误回调
						console.log('pluginError callback: ', iWndIndex, iErrorCode, oError);
						this.$emit('pluginError', iErrorCode)
					},
					windowEventOver: function(iWndIndex) { //鼠标移过回调
						//console.log(iWndIndex);
					},
					windowEventOut: function(iWndIndex) { //鼠标移出回调
						//console.log(iWndIndex);
					},
					windowEventUp: function(iWndIndex) { //鼠标mouseup事件回调
						//console.log(iWndIndex);
					},
					windowFullCcreenChange: function(bFull) { //全屏切换回调
						console.log('fullScreen callback: ', bFull);
					},
					firstFrameDisplay: function(iWndIndex, iWidth, iHeight) { //首帧显示回调
						console.log('firstFrame loaded callback: ', iWndIndex, iWidth, iHeight);
					},
					performanceLack: function() { //性能不足回调
						console.log('performanceLack callback: ');
					}
				});
			},
			//窗口事件处理
			onWindowEvent() {
				// 设置播放容器的宽高并监听窗口大小变化
				window.addEventListener('resize', () => {
					this.player.JS_Resize()
				})

			},
			getCurCameraName() {
				let target = this.curPlayWinds.find(item => item.windowIndex === this.curWindIndex)
				if (target) return target.name
				return ''
			},

			/** 预览
			 * @param url:流媒体url
			 * @param windowIndex:窗口号
			 * @param config:播放配置
			 * @param cameraName:设备（监控点）名称
			 */
			startPreview(url, cameraName = '', windowIndex = this.curWindIndex, config = {}) {
				return new Promise((resolve, reject) => {
					this.player.JS_Play(url, {
						playURL: url,
						mode: 1, //解码类型：0=普通模式; 1=高级模式 默认为0,h5高级模式才能播放
						...config
					}, windowIndex).then(() => {
						//console.log('play success')
						//保存当前播放的窗口号
						if (!this.curPlayWinds.find(v => v.windowIndex === windowIndex)) {
							this.curPlayWinds.push({
								windowIndex,
								name: cameraName
							})
							this.$emit('previewChange', this.curPlayWinds)
						}
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})

			},
			//停止某个窗口播放
			stopOnePlay(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_Stop(windowIndex).then(() => {
						this.curPlayWinds = this.curPlayWinds.filter(item => item.windowIndex !==
							windowIndex)
						this.showOptBar = false;
						this.interval = null
						this.$emit('previewChange', this.curPlayWinds)
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//停止所有窗口播放
			stopAllPlay() {
				return new Promise((resolve, reject) => {
					this.player.JS_StopRealPlayAll().then(() => {
						this.curPlayWinds = []
						this.showOptBar = false;
						this.interval = null
						this.$emit('previewChange', this.curPlayWinds)
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//开启声音
			openSound(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_OpenSound(windowIndex).then(() => {
						//	console.log('open success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//关闭声音
			closeSound(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_CloseSound(windowIndex).then(() => {
						//console.log('close succes')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			/** 设置音量
			 * @param volumn:音量大小，范围1~100，默认50
			 * @param windowIndex:窗口号，默认当前窗口
			 */
			setVolume(volumn = 50, windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_SetVolume(volumn, windowIndex).then(() => {
						//console.log('setVolume succes')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			/** 录像回放
			 * @param url:流媒体url
			 * @param windowIndex:窗口号
			 * @param config:播放配置
			 * @param startTime :回放开始时间，格式类型：2021-06-29T00:00:00Z
			 * @param endTime:回放结束时间，格式类型：2021-06-29T00:00:00Z
			 */
			startPlayBack(url, startTime, endTime, windowIndex = this.curWindIndex, config = {}) {
				return new Promise((resolve, reject) => {
					this.player.JS_Play(url, {
						playURL: url,
						mode: 1, //解码类型：0=普通模式; 1=高级模式 默认为0,h5高级模式才能播放
						...config
					}, windowIndex, startTime, endTime).then(() => {
						//console.log('play success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//暂停回放
			pausePlayBack(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_Pause().then(() => {
						//console.log('pause success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//恢复播放
			resumePlayBack(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_Resume(windowIndex).then(() => {
						//console.log('resume success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//回放快放
			fastPlayBack(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_Fast(windowIndex).then(() => {
						//console.log('fast success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//慢放
			slowPlayBack(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_Slow(windowIndex).then(() => {
						//console.log('slow success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			/** 录像
			 * @param fileName:文件名，可不带后缀，默认为.mp4
			 * @param idstType:录像文件类型,2-ps 5-mp4 
			 * @param windowIndex:窗口号
			 */
			startSave(fileName, idstType = 5, windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					if (!fileName) {
						fileName = new Date().getTime() + '_videotape'
					}
					this.player.JS_StartSaveEx(windowIndex, fileName, idstType).then(() => {
						resolve(windowIndex)
					}).catch(e => {
						reject(e)
					})
				})

			},
			//停止录像并保存
			stopSave(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_StopSave(windowIndex).then(() => {
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})

			},
			/** 抓图
			 * @param fileName:图片名
			 * @param fileType:图片类型，'JPEG','BMP'
			 * @param windowIndex:窗口号
			 */
			capture(fileName, fileType = 'JPEG', windowIndex = this.curWindIndex, callback = () => {}) {

				return new Promise((resolve, reject) => {
					if (!fileName) {
						fileName = new Date().getTime() + "_snap"
					}
					this.player.JS_CapturePicture(windowIndex, fileName, fileType, res => {
						if (res) {
							//base64
							this.saveImg(res, fileName)
							resolve()
						}
					}).catch(e => {
						reject(e)
					})
				})

			},
			//开始对讲szTalkUrl:对讲URL,需要https域名下才支持
			startTalk(szTalkUrl) {
				return new Promise((resolve, reject) => {
					this.player.JS_StartTalk(szTalkUrl).then(() => {
						resolve()
					}, e => {
						reject(e)
					})
				})

			},
			//停止对讲
			stopTalk() {
				return new Promise((resolve, reject) => {
					this.player.JS_StopTalk().then(() => {
						//console.log('stopTalk success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//设置对讲音量(0-100)
			setTalkVolume(nVolume) {
				return new Promise((resolve, reject) => {
					this.player.JS_TalkSetVolume(nVolume).then(() => {
						//console.log('setTalkVolume success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//分屏
			setLayout(splitNum) {
				this.isChangeLayout = true;
				this.player.JS_ArrangeWindow(splitNum).then(() => {}).catch(e => {
					console.log(e)
				}).finally(() => {
					setTimeout(() => {
						this.isChangeLayout = false
					}, 200)
				})
			},
			//是否全屏
			isFullScreen(is = true) {
				this.player.JS_FullScreenDisplay(is).then(() => {
					//console.log('fullscreen success')
				}).catch(e => {
					console.log(e)
					reject(e)
				})
			},
			//单窗口全屏
			singleFullScreen(windowIndex = this.curWindIndex) {
				this.player.JS_FullScreenSingle(windowIndex).then(() => {
					//console.log('singlefullscreen success')
				}).catch(e => {
					console.log(e)
					reject(e)
				})
			},
			//设置窗口大小
			setWindResize(width, height) {
				return new Promise((resolve, reject) => {
					this.player.JS_Resize(width, height).then(() => {
						//console.log('resize success')
						resolve()
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//获取音视频信息
			getVideoInfo(windowIndex = this.curWindIndex) {
				return new Promise((resolve, reject) => {
					this.player.JS_GetVideoInfo(windowIndex).then((e) => {
						//console.log('videoinfo success')
						resolve(e)
					}).catch(e => {
						console.log(e)
						reject(e)
					})
				})
			},
			//操作栏上停止按钮点击停止播放
			handleStop() {
				this.stopOnePlay()
			},
			//保存抓图图片
			saveImg(bold, fileName) {
				let aLink = document.createElement('a')
				let blob = this.base64ToBlob(bold)
				let evt = document.createEvent('HTMLEvents')
				evt.initEvent('click', true, true) //
				aLink.download = fileName
				aLink.href = URL.createObjectURL(blob)
				aLink.click()
			},
			// base64转blob
			base64ToBlob(code) {
				let parts = code.split(';base64,')
				let contentType = parts[0].split(':')[1]
				let raw = window.atob(parts[1]) // 解码base64得到二进制字符串
				let rawLength = raw.length
				let uInt8Array = new Uint8Array(rawLength) // 创建8位无符号整数值的类型化数组
				for (let i = 0; i < rawLength; ++i) {
					uInt8Array[i] = raw.charCodeAt(i) // 数组接收二进制字符串
				}
				return new Blob([uInt8Array], {
					type: contentType
				})
			},
			//声音处理
			handleSound() {
				if (this.hasSound) { //静音
					this.closeSound()
				} else {
					this.openSound()
				}
				this.hasSound = !this.hasSound
			}
		}
	}
</script>


<style scoped lang="scss">
	.comp-container {
		width: 100%;
		height: 100%;
		position: relative;
		overflow: hidden;
	}

	/*
	*组件大小由父元素决定
	*/
	.hk-video {
		width: 100%;
		height: 100%;
		position: relative;
	}

	.control-view {
		position: absolute;
		z-index: 998;
		width: 100%;
		height: 70rpx;
		bottom: 0;
		background-color: #000;
		color: #fff;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding-right: 30rpx;
		box-sizing: border-box;
		transition: 0.3s all;
		transform: translateY(100%);

		.right {
			display: flex;
			align-items: center;
		}

		.icon {
			margin-left: 30rpx;
			width: 34rpx;
			height: auto;

			&.sound-icon {
				width: 40rpx;
			}
		}

		&.show {
			transform: translateY(0);
		}
	}

	.name-view {
		position: absolute;
		z-index: 998;
		width: 100%;
		height: 40rpx;
		top: 0;
		background-color: #000;

		padding-left: 30rpx;
		box-sizing: border-box;
		transition: 0.3s all;
		transform: translateY(-100%);
		font-size: 24rpx;
		line-height: 40rpx;
		color: #fff;
		text-align: left;

		&.show {
			transform: translateY(0);
		}
	}
</style>
