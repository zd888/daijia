<template>
	<view>
		<map id="map" style="width: 100%; height: 700px;" :latitude="latitude" :longitude="longitude" :markers="covers"
			:show-location="true" :polyline="polyline">
		</map>

		<!-- <cover-view class="controls-title">
			<cover-view class="tabs_box">
				<cover-image class="pay_img" src="../static/avatar.png"></cover-image>
				<cover-view class="flex flex-sub margin-left-sm flex-direction justify-between">
					<cover-view class="pay_name">骑手预计{{rider.mDateTime[1]}}送达</cover-view>
					<cover-view class="text-gray margin-top" style="margin-top: 5rpx;">
						距离您{{rider.aDouble}}
					</cover-view>
				</cover-view>
				<cover-view class="flex">
					<cover-image class="pay_img1 margin-right" @click="goNav" src="../../../static/image/order/im.png"></cover-image>
					<cover-image class="pay_img1" @click="call" src="../static/phones.png"></cover-image>
				</cover-view>
			</cover-view>

			<cover-view class="tabs_bottom">
				<cover-view class="pay_btn" @click="call">
					<cover-image class="pay_img" style="width: 40rpx;height: 40rpx;"
						src="../../../static/image/icon_number.png"></cover-image>
					<cover-view class="pay_tit">电话骑手</cover-view>
				</cover-view>
				<cover-view class="pay_btn" @click="goNav">
					<cover-image class="pay_img" style="width: 40rpx;height: 40rpx;"
						src="../../../static/image/my/8.png"></cover-image>
					<cover-view class="pay_tit">联系客服</cover-view>
				</cover-view>
			</cover-view>
		</cover-view>
 -->
	</view>
</template>

<script>
	// 引入SDK核心类，地图组件
	import QQMapWX from '@/components/qqmap-wx-jssdk1.2/qqmap-wx-jssdk.js'
	export default {
		data() {
			return {
				latitude: '',
				longitude: '',
				qqmapsdk: {}, // 腾讯地图小程序的SDK
				covers: [], // 地图上面的标记点
				to: { // 目的地坐标
					latitude: '',
					longitude: '',
				},
				polyline: [], //路线
				polylines: '',

				indentNumber: '',
				riderUserId: '',
				orderDetails: {},
				rider: {},
				timer: '',

			}
		},
		onLoad(option) {
			let that = this

			// 实例化API核心类
			that.qqmapsdk = new QQMapWX({
				key: 'TMWBZ-MZJ3O-QR2WO-SDQZY-HRNWO-3ABYP' // 自己申请的key值
			})

			that.indentNumber = option.indentNumber
			that.getData()


		},
		onShow() {
			let that = this
			this.timer = setInterval(function() {
				that.getLocation()
			}, 10000)
		},
		onHide() {
			console.log(this.timer, '定时器')
			clearInterval(this.timer)
		},
		methods: {
			getData() {
				this.$Request.getT('/app/indent/userIndentMessage?indentNumber=' + this.indentNumber).then(res => {
					console.log(res)
					if (res.code == 0) {
						this.orderDetails = res.data
						this.polylines = res.data.polyline
						
						this.to.latitude = res.data.deliveryAddressLatitude
						this.to.longitude = res.data.deliveryAddressLongitude

						this.riderUserId = this.orderDetails.riderUserId
						this.getLocation()


						// console.log(this.orderDetails.avatar)
						// let marker = {
						// 	id: 1,
						// 	latitude: res.data.deliveryAddressLatitude,
						// 	longitude: res.data.deliveryAddressLongitude,
						// 	iconPath: '../static/01.png',
						// 	width: '30',
						// 	height: '30'
						// }
						// this.markers.push(marker)

						// console.log(this.markers)
					}
				});
			},
			getLocation() {
				let data = {
					riderUserId: this.riderUserId,
					// lat: this.orderDetails.deliveryAddressLatitude,
					// lng: this.orderDetails.deliveryAddressLongitude
				}
				this.$Request.getT('/timedtask/selectRiderLocation', data).then(res => {
					if (res.code === 0) {
						this.latitude = res.data.lat;
						this.longitude = res.data.lng;

						this.covers = [{
							id: 1,
							latitude: res.data.lat,
							longitude: res.data.lng,
							iconPath: '../static/avatar.png',
							width:28,
							height:28,
							// callout: { //自定义标记点上方的气泡窗口 点击有效
							// 	content:res.data.address, //文本
							// 	color: '#333333', //文字颜色
							// 	fontSize: 10, //文本大小
							// 	padding: 10, //附近留白
							// 	borderRadius: 6, //边框圆角
							// 	bgColor: '#FFFFFF', //背景颜色
							// 	display: 'ALWAYS', //常显
							// },
						}, {
							id: 2,
							latitude: this.orderDetails.deliveryAddressLatitude,
							longitude: this.orderDetails.deliveryAddressLongitude,
							iconPath: '../../static/image/zd.png',
							width: 32,
							height: 44,
							callout: { //自定义标记点上方的气泡窗口 点击有效
								content:this.orderDetails.deliveryAddress, //文本
								color: '#333333', //文字颜色
								fontSize: 10, //文本大小
								padding: 10, //附近留白
								borderRadius: 6, //边框圆角
								bgColor: '#FFFFFF', //背景颜色
								display: 'ALWAYS', //常显
							},
						}]
						this.routePlanning()
						// this.rider = res.data
						// this.rider.mDateTime = res.data.mDateTime.split(' ')
						// if (this.rider.aDouble > 1000) {
						// 	this.rider.aDouble = Number((this.rider.aDouble / 1000)).toFixed(2) + "km"
						// } else {
						// 	if (this.rider.aDouble == 0) {
						// 		this.rider.aDouble = "0m";
						// 	} else {
						// 		this.rider.aDouble = Number(this.rider.aDouble).toFixed(1) + "m";
						// 	}
						// }

						// let marker = {
						// 	id: 2,
						// 	latitude: res.data.riderLocation.lat,
						// 	longitude: res.data.riderLocation.lng,
						// 	iconPath: '../../../static/image/rider.png',
						// 	width: '30',
						// 	height: '30',
						// }
						// this.markers.push(marker)
					}
				});
			},
			// 步行路线规划
			routePlanning() {
				let that = this
				that.qqmapsdk.direction({
					mode: 'driving', // 驾车
					from: { // 起始位置(当前位置)坐标
						latitude: that.latitude,
						longitude: that.longitude
					},
					to: that.to, // 目的地位置坐标
					success(res) {
						// console.log(res, '=======')
						var coors = JSON.parse(that.polylines)
						// console.log(coors,'11111111111111')
						// var coors = res.result.routes[0].polyline
						// console.log(coors2,'11111111111111')
						var pl = [] // 点串数组
						// 坐标解压(返回的点串坐标，通过前向差分进行压缩)
						var kr = 1000000
						for (var i = 2; i < coors.length; i++) {
							coors[i] = Number(coors[i - 2]) + Number(coors[i]) / kr
						}
						// 将解压后的坐标放入点串数组pl中
						for (var i = 0; i < coors.length; i += 2) {
							pl.push({
								latitude: coors[i],
								longitude: coors[i + 1]
							})
						}
						// 设置polyline属性，将路线显示出来，将解压坐标第一个数据作为起点
						that.polyline = [{
							points: pl,
							color: '#346EF6',
							width: 4
						}]
					}
				})
			},
			call() {
				uni.makePhoneCall({
					phoneNumber: this.orderDetails.phone
				});
			},
			goNav() {
				uni.navigateTo({
					url: '/pageA/kefu/kefu'
				})
			}
		},
	}
</script>

<style>
	.controls-title {
		width: 90%;
		/* height: 220upx; */
		/* line-height: 220upx; */
		background: #FFFFFF;
		position: fixed;
		bottom: 0rpx;
		margin: 40upx;
		border-radius: 26upx;
		box-shadow: 0upx 30upx 40upx 0upx rgba(187, 170, 163, 0.20);
		/* margin: 0 40rpx; */
		margin-bottom: 50rpx;
	}

	.tabs_box {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 10rpx;
		margin: 20rpx;
	}

	.pay_tit {
		flex: 1;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}

	.pay_img {
		width: 100rpx;
		height: 100rpx;
		border-radius: 10rpx;
	}

	.pay_img1 {
		width: 60rpx;
		height: 60rpx;
		border-radius: 10rpx;
	}

	.tabs_bottom {
		margin: 0 20rpx 20rpx;
		display: flex;
	}

	.pay_btn {
		padding: 5rpx 16rpx;
		border: solid 2rpx #999999;
		margin-right: 20rpx;
		display: flex;
		align-items: center;
		border-radius: 10rpx;
	}

	.pay_name {
		font-size: 32rpx;
		font-weight: bold;
	}

	.pay_line {
		height: 2rpx;
		background-color: #afafaf;
		margin: 10rpx 0;
	}
</style>