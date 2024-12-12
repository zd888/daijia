<template>
	<view class="order">
		<!-- #ifdef H5 -->
		<view style="position: fixed;top:0rpx;width: 100%;z-index: 999;">
		<!-- #endif -->
			<!-- #ifndef H5 -->
			<view style="position: fixed;top: 0;width: 100%;z-index: 999;">
			<!-- #endif -->
				<u-tabs :list="list" :is-scroll="true" active-color="#346EF6" inactive-color=" #666666" :bold="bold"
					:current="current" @change="change" :show-bar="true"></u-tabs>
			</view>

			<view class="ifbox">

				<view class="order_box" v-for="(item,index) in order" :key="index"
					@click="bindorderDetail(item.indentNumber)">
					<view class="order_success">
						<view v-if="item.indentType==1"> 即时代驾</view>
						<view v-if="item.indentType==2">朋友代叫</view>
						<view v-if="item.indentType==3">预约代驾</view>
						<view class="order_name" v-if="item.indentState==0">待接单</view>
						<view class="order_name" v-if="item.indentState==1">已接单</view>
						<view class="order_name" v-if="item.indentState==2">进行中</view>
						<view class="order_name" v-if="item.indentState==3">待支付</view>
						<view class="order_name" v-if="item.indentState==4">已完成</view>
						<view class="order_name" v-if="item.indentState==5">已取消</view>
					</view>
					<u-line color="#E6E6E6" />
					<view class="order_city">
						<view>下单时间：{{item.createTime}}</view>
						<view class="margin-tb">出发地点：{{item.shipAddress}}</view>
						<view>到达地点：{{item.deliveryAddress}}</view>
						<view class="margin-top" v-if="item.type==2">联系电话：18292841990</view>
						<view class="margin-top" v-if="item.type==3">预约时间：021-06-11 11:39</view>
					</view>
					<!-- <u-line color="#E6E6E6" /> -->
					<view class="order_btn">
						<view class="btn2" v-if="item.indentState==0" @tap.stop="bindorderOff(item)">取消订单</view>
						<view class="btn2" v-if="item.indentState==1&&item.isData" @tap.stop="bindorderOff(item)">
							取消订单
						</view>

						<view class="btn2 margin-left" v-if="item.indentState!=3" @click.stop="backindex()">再来一单</view>
						<view class="btn2 margin-left" v-if="item.indentState==3&&item.payUser==0"
							@click.stop="bindorderDetail(item.indentNumber)">立即支付</view>


					</view>
				</view>
				<view class="empty" v-if="order.length == 0">
					<view
						style="width: 90%; margin: 0 auto; position: fixed;top: 30%;left: 0rpx;right: 0rpx;text-align: center;">
						<image src="../../static/image/emety.png" style="width: 300rpx;height: 300rpx;"></image>
						<view style="color: #CCCCCC;">暂无内容</view>
					</view>
				</view>
			</view>
		</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [{
					name: '全部',
					id: ''
				}, {
					name: '待接单',
					id: 0
				}, {
					name: '已接单',
					id: 1
				}, {
					name: '进行中',
					id: 2
				}, {
					name: '待支付',
					id: 3
				}, {
					name: '已完成',
					id: 4
				}, {
					name: '已取消',
					id: 5
				}],
				current: 0,
				bold: false,
				page: 1,
				limit: 10,
				order: [],
				totalPage: 0,
				arr: [],
				showModal: true,
				userId: '',
				time: '',

				creditScore: 0,
				AllcreditScore: 0,
			}
		},
		onLoad() {
			// 接单成功通知 268
			//  订单完成通知 269
			let that = this;
			that.$Request.getT('/app/common/type/268').then(res => { //接单成功通知
				if (res.code == 0) {
					if (res.data && res.data.value) {
						that.arr.push(res.data.value)
					}
				}
			})
			that.$Request.getT('/app/common/type/269').then(res => { //订单完成通知
				if (res.code == 0) {
					if (res.data && res.data.value) {
						that.arr.push(res.data.value)
					}
				}
			})

			that.$Request.getT('/app/common/type/364').then(res => { // 每次取消扣除的信用分数量 364
				if (res.code == 0) {
					if (res.data && res.data.value) {
						that.creditScore = res.data.value
					}
				}
			})
			that.$Request.getT('/app/common/type/365').then(res => { // 低于多少不能接单和下单 365
				if (res.code == 0) {
					if (res.data && res.data.value) {
						that.AllcreditScore = res.data.value
					}
				}
			})
		},
		onShow() {
			// this.order = []
			let token = this.$queue.getData('token')
			this.userId = this.$queue.getData('userId')
			if (token) {
				let index = uni.getStorageSync('current')
				if (index) {
					this.current = index
					this.page = 1;
					this.orderList()
					uni.removeStorageSync('current')
				} else {
					this.current = 0
					this.page = 1;
					this.orderList()
				}

				let that = this
				that.time = setInterval(function() {
					that.orderList()
				}, 30000)
			} else {
				uni.stopPullDownRefresh();
				uni.showModal({
					title: '提示',
					content: '请先去登录',
					success: function(res) {
						if (res.confirm) {
							uni.navigateTo({
								url: '/pages/my/register'
							})
						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				});
			}

			// 	// #ifdef MP-WEIXIN
			// 	//订阅
			// 	if (!uni.getStorageSync('sendorderMsg')) {
			// 		this.openMsg()
			// 	}
			// 	// #endif
			// } else {
			// 	uni.navigateTo({
			// 		url: 'pages/my/register'
			// 	})
		},
		onHide() {
			uni.removeStorageSync('current')
			clearInterval(this.time)
		},
		onUnload() {
			uni.$off('send')
			uni.$off('current')

		},
		methods: {
			backindex() {
				uni.switchTab({
					url: '/pages/index/index'
				})
			},
			change(index) {
				console.log(index)
				this.current = index;
				this.page = 1;
				this.orderList()
			},
			// 订单获取
			orderList() {
				uni.showLoading({
					title: '加载中...',
					mask: false
				});
				let indentState = this.list[this.current].id
				this.$Request.getT('/app/indent/findUserIndent?page=' + this.page + '&limit=' + this.limit +
					'&indentState=' + indentState).then(res => {
					if (res.code === 0) {
						var newTime = new Date()
						// console.log('当前时间：' + newTime,'-------------')
						res.data.records.forEach(item => {
							if (item.indentState == 1 && item.receivingTime) {
								let receivingTime = new Date(newTime).getTime()
								let date = new Date(item.receivingTime)
								var y = date.getFullYear();
								var m = (date.getMonth() + 1) < 10 ? ("0" + (date.getMonth() + 1)) : (date
									.getMonth() + 1);
								var d = date.getDate() < 10 ? ("0" + date.getDate()) : date.getDate();
								var h = date.getHours() < 10 ? ('0' + date.getHours()) : date.getHours()
								var f = date.getMinutes() < 10 ? ('0' + date.getMinutes()) : date
									.getMinutes() + 5
								var s = date.getSeconds() < 10 ? ('0' + date.getseconds()) : date
									.getSeconds()
								var formatdate = y + '-' + m + '-' + d + ' ' + h + ":" + f + ":" + s;
								let nowTime = new Date(formatdate).getTime()
								// console.log(nowTime, '++++++++++++++++++', receivingTime)
								console.log(receivingTime, nowTime, '-------------')
								if (receivingTime <= nowTime) {
									item.isData = true
								} else {
									item.isData = false
								}
								// console.log(nowTime >= receivingTime, item.isData, '-------------')
							} else {
								item.isData = false
							}
						})
						if (this.page == 1) {
							this.order = res.data.records
						}
						if (this.page > 1) {
							if (res.data.records.length > 0) {
								this.order = this.order.concat(res.data.records)

							}
						}
						this.totalPage = res.data.pages
					}
					uni.hideLoading()
					uni.stopPullDownRefresh();

				});
			},
			// 订单详情
			bindorderDetail(indentNumber) {
				// #ifdef MP-WEIXIN
				if (uni.getStorageSync('sendorderMsg')) {
					uni.requestSubscribeMessage({
						tmplIds: this.arr,
						success(re) {
							// console.log(re,'**********')
							var datas = JSON.stringify(re);
							if (datas.indexOf("accept") != -1) {
								console.log(re)
							}
						},
						fail: (res) => {
							console.log(res)
						}
					})
				}
				// #endif
				console.log(indentNumber)
				// uni.navigateTo({
				// 	url: '/pages/order/orderDetail/detail?indentNumber=' + indentNumber
				// })
				uni.navigateTo({
					url: '/my/order/pay?indentNumber=' + indentNumber
				})
			},
			// 再来一单
			bindorder(e) {
				// #ifdef MP-WEIXIN
				if (uni.getStorageSync('sendorderMsg')) {
					uni.requestSubscribeMessage({
						tmplIds: this.arr,
						success(re) {
							// console.log(re,'**********')
							var datas = JSON.stringify(re);
							if (datas.indexOf("accept") != -1) {
								console.log(re)
							}
						},
						fail: (res) => {
							console.log(res)
						}
					})
				}
				// #endif
				console.log(e)
				let index = e.indentType
				let current = e.buyType
				if (e.indentType == 1 || e.indentType == 2) {
					uni.navigateTo({
						url: '/pages/Helpsend/Helpsend?indentNumber=' + e.indentNumber + '&index=' + index
					})
				} else if (e.indentType == 3) {
					uni.navigateTo({
						url: '/pages/Helppay/Helppay?indentNumber=' + e.indentNumber + '&index=' + index +
							'&current=' + current
					})
				} else if (e.indentType == 4) {
					uni.navigateTo({
						url: '/pages/Cityservice/Cityservice?indentNumber=' + e.indentNumber + '&index=' + index
					})
				}
			},
			// 取消订单
			bindorderOff(e) {
				// #ifdef MP-WEIXIN
				if (uni.getStorageSync('sendorderMsg')) {
					uni.requestSubscribeMessage({
						tmplIds: this.arr,
						success(re) {
							// console.log(re,'**********')
							var datas = JSON.stringify(re);
							if (datas.indexOf("accept") != -1) {
								console.log(re)
							}
						},
						fail: (res) => {
							console.log(res)
						}
					})
				}
				// #endif
				let indentNumber = e.indentNumber
				let content = ''
				if (e.indentState == 1) {
					content = '师傅接单后5分钟内可取消，取消订单将会扣除信用分' + this.creditScore + '分,信用分低于' + this.AllcreditScore +
						'分无法下单，是否取消？'
				} else {
					content = '确定取消订单？'
				}
				uni.showModal({
					title: '温馨提示',
					content: content,
					showCancel: true,
					cancelText: '取消',
					confirmText: '确认',
					success: res => {
						if (res.confirm) {
							this.$Request.postT('/app/indent/userCancleIndent?indentNumber=' + indentNumber)
								.then(res => {
									if (res.code == 0) {
										uni.showToast({
											title: '订单取消成功'
										});
										this.page = 1;
										this.orderList()
									} else {
										uni.hideLoading();
										uni.showModal({
											showCancel: false,
											title: '订单失败',
											content: res.msg
										});
									}
								});
						}
					}
				});
			},
			//确认订单
			bindconfirm(e) {
				// #ifdef MP-WEIXIN
				if (uni.getStorageSync('sendorderMsg')) {
					uni.requestSubscribeMessage({
						tmplIds: this.arr,
						success(re) {
							// console.log(re,'**********')
							var datas = JSON.stringify(re);
							if (datas.indexOf("accept") != -1) {
								console.log(re)
							}
						},
						fail: (res) => {
							console.log(res)
						}
					})
				}
				// #endif
				// console.log(e)
				let indentNumber = e.indentNumber
				console.log(indentNumber)
				this.$Request.postT('/app/indent/userDelivery?indentNumber=' + indentNumber).then(res => {
					console.log(res)
					if (res.code == 0) {
						uni.showToast({
							title: '订单确认成功'
						});
						this.page = 1;
						this.orderList()
					} else {
						uni.hideLoading();
						uni.showModal({
							showCancel: false,
							title: '订单确认失败',
							content: res.msg
						});
					}
				});
			},
			// 立即付款
			bindorderpay(e) {
				// #ifdef MP-WEIXIN
				if (uni.getStorageSync('sendorderMsg')) {
					uni.requestSubscribeMessage({
						tmplIds: this.arr,
						success(re) {
							// console.log(re,'**********')
							var datas = JSON.stringify(re);
							if (datas.indexOf("accept") != -1) {
								console.log(re)
							}
						},
						fail: (res) => {
							console.log(res)
						}
					})
				}
				// #endif
				uni.navigateTo({
					url: '/pages/order/pay/pay?indentNumber=' + e.indentNumber
				})
			},
			// 开启订阅消息
			openMsg() {
				var that = this
				wx.getSetting({
					withSubscriptions: true, //是否获取用户订阅消息的订阅状态，默认false不返回
					success(ret) {
						if (ret.subscriptionsSetting.itemSettings) {
							uni.setStorageSync('sendorderMsg', true)
							uni.openSetting({ // 打开设置页 
								success(rea) {
									console.log(rea.authSetting)
								}
							});
						} else { // 用户没有点击“总是保持以上，不再询问”则每次都会调起订阅消息
							uni.setStorageSync('sendorderMsg', false)
							uni.showModal({
								title: '提示',
								content: '为了更好的体验,请绑定消息推送',
								confirmText: '确定',
								cancelText: '取消',
								success: function(res) {
									if (res.confirm) {
										uni.requestSubscribeMessage({
											tmplIds: that.arr,
											success(re) {
												var datas = JSON.stringify(re);
												if (datas.indexOf("accept") != -1) {
													console.log(re)
												}
											},
											fail: (res) => {
												console.log(res)
											}
										})
										that.showModal = false
									} else if (res.cancel) {
										that.showModal = true
									}
								}
							})
						}
					}
				})
			}
		},
		onReachBottom: function() {
			if (this.userId) {
				if (this.page < this.totalPage) {
					this.page = this.page + 1;
					this.orderList();
				}
			}
		},
		onPullDownRefresh: function() {
			if (this.userId) {
				this.page = 1;
				this.orderList();
			}

		},
	}
</script>

<style lang="less">
	page {
		background: #F5F5F5;
	}

	.empty {
		width: 100%;
		background: #ffffff;
		height: 100vh;
	}

	.ifbox {
		/* #ifdef H5 */
		padding-top: 85rpx;
		/* #endif */
		/* #ifndef H5 */
		padding-top: 86rpx;
		/* #endif */
	}

	.order_box {
		background: #FFFFFF;
		border-radius: 24rpx;
		margin: 30rpx;

		.order_success {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 30rpx;

			.order_name {
				color: #346EF6;
			}
		}

		.order_city {
			padding: 30rpx;
			color: #666666;
		}

		.order_btn {
			padding: 30rpx;
			display: flex;
			align-items: center;
			justify-content: flex-end;

			.btn2 {
				width: 150rpx;
				height: 60rpx;
				background: #346EF6;
				border-radius: 16rpx;
				font-size: 24rpx;
				font-family: PingFang SC;
				font-weight: 500;
				color: #FFFFFF;
				display: flex;
				align-items: center;
				justify-content: center;
			}

			.btn1 {
				width: 150rpx;
				height: 60rpx;
				border: 1px solid #999999;
				border-radius: 16rpx;
				font-size: 24rpx;
				font-family: PingFang SC;
				font-weight: 500;
				color: #999999;
				display: flex;
				align-items: center;
				justify-content: center;

			}
		}
	}
</style>