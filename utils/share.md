# APP分享、微博分享、QQ分享、微信好友、朋友圈

### QQ交流群(学习干货多多) 607391225
![QQ交流群](http://qn.kemean.cn//upload/202004/14/15868301778472k7oubi6.png)

### 使用方法
```
<template>
	<button type="default" @click="onShare">APP分享</button>
</template>

<script>
	// 引入方法
	import appShare from "@/utils/share.js"
	export default {
		methods: {
			onShare(){
				let shareData = {
					shareUrl:"https://kemean.com/",
					shareTitle:"分享的标题",
					shareContent:"分享的描述",
					shareImg:"http://qn.kemean.cn//upload/202004/18/1587189024467w6xj18b1.jpg",
				};
				// 调用
				appShare(shareData,res => {
					console.log("分享成功回调",res);
				});
			}
		}
	}
</script>
```
### 插件说明
| 参数名称	     | 类型 	 | 默认值	    | 描述
| -------------- |---------- | ------------ | --------------------------------------- |
| shareUrl  	 | String	 | --    | 分享的地址`（type 为 0 时必传）`         |
| shareTitle	 | String	 | --       	| 分享的标题                               |
| shareContent	 | String	 | 分享的描述	| 分享的描述`（type 为 1 时必传）`          |
| shareImg  	 | String	 | 分享的图片	| 分享的图片`（type 为 0、2、5 时必传）`    |
| mediaUrl	     | String	 | --	        | 分享的音视频地址`（type 为 3、4 时必传）` |
| type  	     | Number	 | 参考平台默认值| 分享形式，如图文、纯文字、纯图片、音乐、视频、小程序等，具体参考下面type说明|

### type 值说明
| 值   	  | 说明 	 | 支持平台	     | 
| ------- |--------- | ------------- |
| 0   	  | 图文 	 | 微信、新浪微博 | 
| 1   	  | 纯文字 	 | 全平台支持     |
| 2   	  | 纯图片 	 | 全平台支持     |
| 3   	  | 音乐 	 | 微信、QQ       |
| 4   	  | 视频 	 | 微信、新浪微博 |
| 5   	  | 小程序 	 | 微信聊天界面   |

### 平台默认值
| 平台       | 默认值 	  |
| ---------- |--------- |
| 新浪微博   | 0 	      |
| 微信好友   | 0 	      |
| 微信朋友圈 | 0 	      |
| QQ        | 1 	      |

### 分享小程序必传参数`（type 为 5 时必传）`
注意：`小程序必须是在微信开放平台与App绑定的才行`
| 参数名称	     | 类型 	 | 默认值	    | 描述
| -------------- |---------- | ------------ | --------------------------- |
| appId     	 | String	 | --	        | 微信小程序原始id (比传)            |
| appPath   	 | String	 | --	        | 点击链接进入的页面 (比传)          |
| appWebUrl 	 | String	 | ""	        | 兼容低版本的网页链接(比传)  |
| appType   	 | Number	 | 0	        | 微信小程序版本类型，可取值： 0-正式版； 1-测试版； 2-体验版。 默认值为0 |
