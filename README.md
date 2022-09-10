
# vue-hk-video
海康H5插件2.0.0版本组件 uniapp


组件大小默认填充父元素(100%,100%)



**Props**

| 参数 |说明  |类型  | 默认值| 可选值 |是否必须 |
| --- | --- | --- | --- | --- | --- |
currentSplit	|初始化默认分屏布局，2表示2x2布局	|Number|	2|	——	|否
maxSplit|	最大分屏，4表示4x4布局|	Number|	4|	——|	否
jsBasePath|	引用H5player.min.js的js路径|	String	|/static/lib|	——	|是




**Event**

| 事件名 |说明  |回调参数
| --- | --- |--- |
windowChang|	选中窗口改变|	（windId） windId：窗口号，从0开始
previewChange	|当前所有预览状态的窗口集改变，新增或者关闭预览触发	|（{  name:'''//设备名称,windowIndex:1//窗口号，索引从0开始}）
pluginError	|插件启动失败|	(iErrorCode) iErrorCode:错误编码


**方法**
通过实例调用例如：this.$refs.hkvideo.setLayout(2);


| 方法名 |说明  |参数|返回值
| --- | --- |--- |--- |
startPreview|	单个窗口预览|	(url, windowIndex = this.curWindIndex, config = {}) url：流媒体url，必填，windowIndex:窗口号，默认当前窗口号，config:播放配置,默认高级模式，具体配置看开发指南|	Promise(成功后回调，下同)
stopAllPlay|	停止所有窗口播放（回放）|	无	|Promise
stopOnePlay|	停止某个窗口播放（回放）	|（windowIndex = this.curWindIndex) windowIndex：窗口号，默认当前窗口|	Promise
startPlayBack|录像回放|(url, startTime, endTime, windowIndex = this.curWindIndex, config = {}) url：流媒体url，必填，startTime :回放开始时间，格式类型：2021-06-29T00:00:00Z，必填，endTime:回放结束时间，格式类型：2021-06-29T00:00:00Z，必填，windowIndex:窗口号，默认当前窗口号，config:播放配置,默认高级模式，具体配置看开发指南|	Promise
pausePlayBack |	暂停回放|	(windowIndex = this.curWindIndex) windowIndex：窗口号，默认当前窗口|	Promise
resumePlayBack|	恢复回放	|(windowIndex = this.curWindIndex) windowIndex：窗口号，默认当前窗口	Promise
fastPlayBack	|回放快放|	(windowIndex = this.curWindIndex)  windowIndex：窗口号，默认当前窗口	Promise
slowPlayBack|	回放慢放	|(windowIndex = this.curWindIndex)  windowIndex：窗口号，默认当前窗口	Promise
isFullScreen	|是否全屏|	(is = true) is:是否全屏，boolean类型	|Promise
singleFullScreen	|单个窗口全屏|	(windowIndex = this.curWindIndex)  windowIndex：窗口号，默认当前窗口	Promise
setLayout	|设置布局|	(splitNum) splitNum：splitNum x splitNum布局，number类型，例如2表示2x2布局	|——
openSound	|开启声音|	(windowIndex = this.curWindIndex) windowIndex：窗口号，默认当前窗口	|Promise
closeSound|	关闭声音|	(windowIndex = this.curWindIndex) windowIndex：窗口号，默认当前窗口	|Promise
setVolume	|设置音量|	(volumn = 50, windowIndex = this.curWindIndex) volumn:音量大小，范围1~100，默认50 windowIndex:窗口号，默认当前窗口|	Promise
startSave	|开始本地录像	|(fileName, idstType = 5, windowIndex = this.curWindIndex) fileName:文件名，可不带后缀，默认为.mp4，不传默认当前时间戳命名，idstType:录像文件类型,2-ps 5-mp4 ，windowIndex:窗口号	|Promise，成功后回调返回窗口号windowIndex
stopSave|	停止本地录像	|(windowIndex = this.curWindIndex)  windowIndex：窗口号，默认当前窗口|	Promise
capture|	抓图	|(fileName, fileType = 'JPEG', windowIndex = this.curWindIndex, callback = () => {})fileName图片名，默认当前时间戳命名，fileType:图片类型，默认JPEG'，可选值：'BMP'，windowIndex：窗口号，默认当前窗口，callback：成功回调函数	|Promise
startTalk|	开始对讲	|(szTalkUrl) szTalkUrl:对讲URL	|Promise
stopTalk|	停止对讲|	无	|Promise
setTalkVolume	|设置对讲音量	|(nVolume) nVolume：音量(0-100)|	Promise
setWindResize|	设置窗口大小|	(width, height) width:宽，height:高	|Promise
getVideoInfo|	获取音视频信息|	(windowIndex = this.curWindIndex) windowIndex：窗口号，默认当前窗口|	Promise，成功后回调返回音视频信息



**demo**

```html
<template>
	<view class="content">
		<view class="video-box">
			<hk-video ref="hkvideo" @change="onWindChange" @previewChange="onPreviewChange"></hk-video>
		</view>
		<view class="opt-btn">
			<button @click="startPreview">预览</button>
			<button @click="setLayout(1)">1x1</button>
			<button @click="setLayout(2)">2x2</button>
			<button @click="setLayout(3)">3x3</button>
			<button @click="setLayout(4)">4x4</button>
			<button @click="setFullScreen(true)">全屏</button>
		</view>
	
	</view>
</template>
```

```javascript
<script>
	import hkVideo from '@/components/hk-video/index.vue'
	export default {
		components: {
			hkVideo,
		},
		data() {
			return {
				previewUrl: 'ws://192.xx.xx.xx:xxx/openUrl/he7HlcI',//实际对接从海康接口获取
			}
		},

		methods: {
			//预览
			startPreview() {
				this.$refs.hkvideo.startPreview(this.previewUrl, '测试设备').then(()=>{
                   console.log('预览成功')
                })
			},
           //分屏
			setLayout(layout) {
				this.$refs.hkvideo.setLayout(layout)
			},
            //是否全屏
			setFullScreen(n) {
				this.$refs.hkvideo.isFullScreen(false)
			},
           //选中窗口
			onWindChange(index) {
				console.log(index, 'windIndex')
			},
          //窗口预览改变
			onPreviewChange(e){
			 console.log(e,'previewChange')	
			}
			
		}
	}
</script>

```

