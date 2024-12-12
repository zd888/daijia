<template>
	<view style="padding-bottom: 90rpx;">
		<view class="box">
			<view class="flex align-center ">
				<view class="titls" v-if="order.indentState==0">待接单</view>
				<view class="titls" v-if="order.indentState==1">已接单</view>
				<view class="titls" v-if="order.indentState==2">进行中</view>
				<view class="titls" v-if="order.indentState==3">待支付</view>
				<view class="titls" v-if="order.indentState==4">已完成</view>
				<view class="titls" v-if="order.indentState==5">已取消</view>
				<view class="bb" v-if="order.indentType==1"> 即时代驾</view>
				<view class="bb" v-if="order.indentType==2">朋友代叫</view>
				<view class="bb" v-if="order.indentType==3">预约代驾</view>
			</view>
			<view class="margin-top-xs" style="color: #999999;" v-if="order.indentState==0">等待附近司机接单...</view>
			<view class="margin-top-xs" style="color: #999999;" v-if="order.indentState==1">司机正在努力赶来</view>
			<view class="margin-top-xs" style="color: #999999;" v-if="order.indentState==5">订单已取消，再来一单</view>
			<view class="margin-top-xs" style="color: #999999;" v-if="order.indentState==2">订单进行中</view>
			<view class="margin-top-xs" style="color: #999999;" v-if="order.indentState==3">已到达终点，请及时支付</view>
			<view class="margin-top-xs" style="color: #999999;" v-if="order.indentState==4">订单已完成</view>
		</view>


		<view class="boxa">
			<view class="flex align-center justify-between padding-lr">
				<view class="flex align-center">
					<image src="../static/data.png" style="width: 34rpx;height: 38rpx;"></image>
					<view class="margin-left-sm" v-if="order.indentType==1"> 即时代驾</view>
					<view class="margin-left-sm" v-if="order.indentType==2">朋友代叫</view>
					<view class="margin-left-sm" v-if="order.indentType==3">预约代驾</view>
				</view>
				<!-- <view>时间:16:00到达</view> -->
			</view>
			<view class="margin-tb-sm" style="width: 100%;height:1rpx;background: #F2F2F2;"></view>
			<view class="addbox bgs margin-lr margin-bottom"
				@click="openMap(order.shipAddressLatitude,order.shipAddressLongitude,order.shipAddress)">
				<view class="add_cont">
					<view class="green"></view>
					<view class="add_tit">
						<view>{{order.shipAddress}}</view>
					</view>
				</view>
				<view>
					<u-icon name="map-fill" color="#1FC657" size="45"></u-icon>
				</view>
			</view>
			<view class="addbox bg margin-lr"
				@click="openMap(order.deliveryAddressLatitude,order.deliveryAddressLongitude,order.deliveryAddress)">
				<view class="add_cont">
					<view class="orgin"></view>
					<view class="add_tit">{{order.deliveryAddress}}</view>
				</view>
				<view>
					<u-icon name="map-fill" color="#FBAC04" size="45"></u-icon>
				</view>
			</view>
			<view class="margin-tb-sm" style="width: 100%;height:1rpx;background: #F2F2F2;"></view>
			<view class="flex align-center justify-between padding-lr ">
				<view class="flex align-center">
					<u-icon name="map-fill" color="#333333" size="45"></u-icon>
					<view class="margin-left-xs">公里数<text style="color: #3699FF;">{{order.distance}}km</text></view>
				</view>
				<view class="flex align-center">
					<text v-if="order.indentState!=4">预估</text>
					<text v-else>实付</text>
					<view style="color: #FF2020;font-size: 32rpx;">
						<text>¥</text>
						<text class="text-bold" style="font-size:38rpx;"
							v-if="order.indentState==3||order.indentState==4">{{order.indentMoney}}</text>
						<text class="text-bold" style="font-size: 38rpx;" v-else>{{order.estimateMoney}}</text>
					</view>
				</view>
			</view>
		</view>

		<view class="box flex align-center" v-if="order.indentState!=0&&order.indentState!=5&&order.riderNickName"
			@click="goriderMap()">
			<image :src="order.riderAvatar?order.riderAvatar:'../../static/image/logo.png'"
				style="width: 90rpx;height: 90rpx;border-radius: 50%;"></image>
			<view class="margin-left-sm flex align-center justify-between" style="width: 80%;">
				<view>
					<view class="titls">{{order.riderNickName}}</view>
					<view class="margin-top-xs" style="color: #999999;font-size: 26rpx;">驾龄:{{order.drivingYear}}年
					</view>
				</view>
				<view @click.stop="bindPhone">
					<image src="../static/phone.png" style="width: 42rpx;height: 52rpx;"></image>
				</view>
			</view>
		</view>
		<view class="box" v-if="order.indentState==4&&order.evaluateMessage">
			<view class="titls margin-bottom-sm">评价</view>
			<!-- <view class="" v-if="order.evaluate==0">满意</view>
			<view class="" v-if="order.evaluate==1">不满意</view> -->
			<view class="margin-top-xs">{{order.evaluateMessage}}</view>
		</view>

		<view class="box " style="color: #999999;">
			<view class="titls">订单信息</view>

			<view class="margin-tb" v-if="order.remarks">
				<view>备注</view>
				<view class="margin-top-xs">{{order.remarks}}</view>
			</view>
			<view v-if="order.indentType==2">
				<view class="flex align-center justify-between margin-tb">
					<view>付款对象</view>
					<view v-if="order.payUser==0">下单人</view>
					<view v-if="order.payUser==1">乘车人</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>乘车人</view>
					<view>{{order.customName}}</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>乘车人电话</view>
					<view>{{order.customPhones}}</view>
				</view>
			</view>
			<view v-if="order.indentType==3">
				<view class="flex align-center justify-between margin-tb">
					<view>预约时间</view>
					<view>{{order.hopeTime}}</view>
				</view>
			</view>
			<view v-if="order.indentState==3||order.indentState==4">
				<view class="flex align-center justify-between margin-tb">
					<view>总公里数</view>
					<view>{{order.moneyJson.distance}}km</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>基础公里数</view>
					<view>{{order.moneyJson.kilometer}}km</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>基础起步价</view>
					<view>￥{{order.moneyJson.initMoney}}</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>超出部分公里数</view>
					<view>{{order.moneyJson.metre}}km</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>超出部分每公里价格</view>
					<view>￥{{order.moneyJson.unitOverMoney}}</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>超出部分收费总金额</view>
					<view>￥{{order.moneyJson.otherMoney}}</view>
				</view>
				<view class="flex align-center justify-between margin-tb">
					<view>总时长费</view>
					<view>￥{{order.moneyJson.durationAllMoney}}</view>
				</view>
				<view class="flex align-center justify-between margin-tb" v-if="order.couponMoney">
					<view>优惠劵</view>
					<view>-￥{{order.couponMoney}}</view>
				</view>
			</view>

			<view class="flex align-center justify-between margin-tb">
				<view>订单号码</view>
				<view @click.stop="copyClick(order.indentNumber)">{{order.indentNumber}}
					<image src="../static/copy.png" style="width:28rpx;height: 28rpx;"></image>
				</view>
			</view>
			<view class="flex align-center justify-between ">
				<view>下单时间</view>
				<view>{{order.createTime}}</view>
			</view>
			<view class="flex align-center justify-between margin-top" v-if="order.modeOfPayment">
				<view>支付方式</view>
				<view v-if="order.modeOfPayment==1">微信支付</view>
				<view v-if="order.modeOfPayment==2">支付宝支付</view>
				<view v-if="order.modeOfPayment==0">余额支付</view>
			</view>
		</view>

		<view class="tabber"
			v-if="order.indentState==0||order.indentState==3||order.indentState==4||order.indentState==1">
			<view class="btn" v-if="order.indentState==0" @click="bindorderOff()">取消订单</view>
			<!-- <view class="btn2">抵达起点</view> -->
			<view class="btn1 margin-left" v-if="order.indentState!=3" @click="backindex()">再来一单</view>
			<view class="btn1 " v-if="order.indentState==3" @click="orderpay()">立即支付</view>

			<view class="btn1 margin-left" v-if="order.indentState == 4 &&!order.evaluateMessage"
				@click="bindcomment()">去评价</view>
			<view class="btn1 margin-left" v-if="order.indentState==4" @click="bindtousu()">去投诉</view>
		</view>

		<u-popup v-model="showpay" mode="bottom">
			<view class="popup_pay">
				<view class=" margin-top padding-lr radius">
					<view style="padding: 0 20upx;margin-top: 36rpx;">
						<view
							style="display: flex;height: 100upx;align-items: center;padding: 20upx 0;justify-content: center;"
							v-for="(item,index) in openLists" :key='index'>
							<image :src="item.image" style="width: 55upx;height: 55upx;border-radius: 50upx;">
							</image>
							<view style="font-size: 30upx;margin-left: 20upx;width: 70%;">
								{{item.name}} <text @click="goconz" class="margin-left text-sm" style="color:#3699FF;"
									v-if="item.id==3">去充值</text>
							</view>
							<radio-group name="openWay" style="margin-left: 45rpx;" @tap='selectWay(item)'>
								<label class="tui-radio">
									<radio color="#3699FF" :checked="openWay == item.id ? true : false" />
								</label>
							</radio-group>
						</view>
					</view>
				</view>
				<view class="pay_btns" @click="pay()">确认支付</view>
			</view>
		</u-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				indentNumber: '',
				order: [],
				showpay: false,
				openWay: 1,
				openLists: [],
				creditScore: 0,
				AllcreditScore: 0,
			}
		},
		onLoad(option) {
			if (option.indentNumber) {
				this.indentNumber = option.indentNumber
				this.getDetail()
			}

			// #ifdef MP-WEIXIN
			this.openLists = [{
				id: 1,
				image: '../../static/image/icon_weixin.png',
				name: '微信'
			}, {
				id: 3,
				image: '../../static/image/lingian.png',
				name: '余额'
			}]
			// #endif
			// #ifdef H5
			let ua = navigator.userAgent.toLowerCase();
			if (ua.indexOf('micromessenger') !== -1) {
				//公众号是否自动登录  416
				this.$Request.get('/app/common/type/416').then(res => {
					if (res.data && res.data.value && res.data.value == '是') {
						this.openLists = [{
							id: 2,
							image: '../../static/image/zhifubao.png',
							name: '支付宝'
						}, {
							id: 1,
							image: '../../static/image/icon_weixin.png',
							name: '微信'
						}, {
							id: 3,
							image: '../../static/image/lingian.png',
							name: '余额'
						}];
						this.openWay = 3;
					} else {
						this.openLists = [{
							id: 2,
							image: '../../static/image/zhifubao.png',
							name: '支付宝'
						}, {
							id: 3,
							image: '../../static/image/lingian.png',
							name: '余额'
						}];
						this.openWay = 3;
					}
				})
			} else {
				this.openLists = [{
					id: 2,
					image: '../../static/image/zhifubao.png',
					name: '支付宝'
				}, {
					id: 3,
					image: '../../static/image/lingian.png',
					name: '余额'
				}];
				this.openWay = 3;
			}
			// #endif
			// #ifdef APP-PLUS
			this.openLists = [{
					id: 1,
					image: '../../static/image/icon_weixin.png',
					name: '微信'
				},
				{
					id: 2,
					image: '../../static/image/zhifubao.png',
					name: '支付宝'
				}, {
					id: 3,
					image: '../../static/image/lingian.png',
					name: '余额'
				}
			]
			this.openWay = 3;
			// #endif
			let that = this
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

		},
		methods: {
			goconz() {
				uni.navigateTo({
					url: '/my/wallet/Txmoney'
				})
			},
			copyClick(copy) {
				uni.setClipboardData({
					data: copy,
					success: function(res) {
						uni.getClipboardData({
							success: function(res) {
								uni.showToast({
									title: "复制成功",
									icon: 'none',
								});
							},
						});
					},
				});
			},
			// 一键导航
			openMap(latitude, longitude, name) {
				uni.openLocation({
					latitude: latitude - 0, //要去的纬度-地址       
					longitude: longitude - 0, //要去的经度-地址
					name: name, //地址名称
					address: name, //详细地址名称
					success: function() {
						console.log('导航成功');
					},
					fail: function(error) {
						console.log(error)
					}
				});
			},
			goriderMap() { //查看司机位置
				uni.navigateTo({
					url: '/my/order/orderMap?indentNumber=' + this.order.indentNumber
				})
			},
			selectWay: function(item) {
				this.openWay = item.id;
			},
			orderpay() {
				this.showpay = true
			},
			bindPhone() {
				uni.makePhoneCall({
					phoneNumber: this.order.riderPhone //仅为示例
				});
			},
			getDetail() {
				let data = {
					indentNumber: this.indentNumber
				}
				this.$Request.getT('/app/indent/userIndentMessage', data).then(res => {
					if (res.code === 0) {
						this.order = res.data
						if (this.order.moneyJson) {
							let moneyJson = JSON.parse(this.order.moneyJson)
							this.order.moneyJson = moneyJson
						}

						if (this.order.customPhone) {
							let mobile = this.order.customPhone
							this.order.customPhones = mobile.substring(0, 3) + "****" + mobile.substring(7)
						}
					}
					uni.hideLoading()

				});
			},
			backindex() {
				uni.switchTab({
					url: '/pages/index/index'
				})
			},
			// 取消订单
			bindorderOff() {
				let content = ''
				if (this.order.indentState == 1) {
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
							this.$Request.postT('/app/indent/userCancleIndent?indentNumber=' + this
									.indentNumber)
								.then(res => {
									if (res.code == 0) {
										uni.showToast({
											title: '订单取消成功'
										});
										this.getDetail()
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
			pay() {
				if (this.openWay == 0) {
					this.$queue.showToast('请选择支付方式!')
					return;
				}
				if (this.openWay == 1) {
					console.log('微信')
					// #ifdef APP-PLUS
					// 微信APP支付 根据订单id获取支付信息
					let data = {
						indentId: this.order.indentId
					}
					this.$Request.postT("/app/wxPay/payAppOrder", data).then(res => {
						// console.log(JSON.stringify(res), '微信支付信息')
						this.isCheckPay(res.code, 'wxpay', JSON.stringify(res.data));
					});
					// #endif
					// #ifdef MP-WEIXIN
					let that = this
					let data = {
						indentId: that.order.indentId
					}
					that.$Request.postT("/app/wxPay/wxPayJsApiOrder", data).then(res => {
						if (res.code == 0) {
							uni.requestPayment({
								provider: 'wxpay',
								timeStamp: res.data.timestamp,
								nonceStr: res.data.noncestr,
								package: res.data.package,
								signType: res.data.signType,
								paySign: res.data.sign,
								success: function(res) {
									console.log(res)
									uni.hideLoading();
									uni.showToast({
										title: '实名认证提交成功',
										icon: 'none'
									});
									setTimeout(function() {
										uni.navigateBack(1)
									}, 1000)
								},
								fail: function(err) {
									console.log('fail:' + JSON.stringify(err));
									uni.hideLoading();
									that.$queue.showToast('支付失败');
								}
							});
						} else {
							uni.showToast({
								title: res.msg,
								icon: 'none'
							})
						}
					});
					// #endif

					// #ifdef H5
					// 微信h5支付 根据订单id获取支付信息
					let ua = navigator.userAgent.toLowerCase();
					if (ua.indexOf('micromessenger') !== -1) { //微信里打开
						let data = {
							indentId: this.order.indentId
						}
						this.$Request.postT('/app/wxPay/wxPayMpOrder', data).then(res => {
							if (res.code == 0) {
								console.log()
								this.callPay(res.data);
							} else {
								uni.showToast({
									icon: 'none',
									title: '支付失败!'
								});
							}
						});
					}
					// #endif
				} else if (this.openWay == 2) {
					// APP支付宝支付
					// #ifdef H5
					// this.form.payType = 5
					let data = {
						indentId: this.order.indentId
					}
					this.$Request.postT('/app/aliPay/payH5Order', data).then(
						res => {
							if (res.code == 0) {
								const div = document.createElement('div')
								div.innerHTML = res.data //此处form就是后台返回接收到的数据
								document.body.appendChild(div)
								document.forms[0].submit()
							} else {
								uni.showToast({
									icon: 'none',
									title: '支付失败!'
								});
							}
						});
					// #endif

					// #ifdef APP
					// this.form.payType = 4
					let data = {
						indentId: this.order.indentId
					}
					this.$Request.postT("/app/aliPay/payAppOrder", data).then(res => {
						console.log(JSON.stringify(res), '支付宝支付信息')
						this.isCheckPay(res.code, 'alipay', res.data)
					});
					// #endif
				} else if (this.openWay == 3) {
					this.showpay = false
					uni.showModal({
						title: '温馨提示',
						content: '确定使用余额支付？',
						showCancel: true,
						cancelText: '取消',
						confirmText: '确认',
						success: res => {
							if (res.confirm) {
								uni.showLoading({
									title: '支付中...'
								})
								this.$Request.postT('/app/userMoney/moneyPayIndent?indentId=' + this
										.order.indentId)
									.then(res => {
										if (res.code == 0) {
											uni.showToast({
												title: '订单支付成功'
											});
											setTimeout(function() {
												uni.navigateBack()
											}, 1000)

										} else {
											uni.hideLoading();
											uni.showModal({
												showCancel: false,
												title: '提示',
												content: res.msg
											});
										}
									});
							}
						}
					});

				}
			},
			callPay: function(response) {
				if (typeof WeixinJSBridge === "undefined") {
					if (document.addEventListener) {
						document.addEventListener('WeixinJSBridgeReady', this.onBridgeReady(response), false);
					} else if (document.attachEvent) {
						document.attachEvent('WeixinJSBridgeReady', this.onBridgeReady(response));
						document.attachEvent('onWeixinJSBridgeReady', this.onBridgeReady(response));
					}
				} else {
					this.onBridgeReady(response);
				}
			},
			onBridgeReady: function(response) {
				let that = this;
				// if (!response.package) {
				// 	return;
				// }
				console.log(response, 111111)
				WeixinJSBridge.invoke(
					'getBrandWCPayRequest', {
						"appId": response.appid, //公众号名称，由商户传入
						"timeStamp": response.timestamp, //时间戳，自1970年以来的秒数
						"nonceStr": response.noncestr, //随机串
						"package": response.package,
						"signType": response.signType, //微信签名方式：
						"paySign": response.sign //微信签名
					},
					function(res) {
						console.log(res, '/*-/*-/*-')
						if (res.err_msg === "get_brand_wcpay_request:ok") {
							// 使用以上方式判断前端返回,微信团队郑重提示：
							//res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
							uni.showLoading({
								title: '支付成功'
							});
							setTimeout(function() {
								uni.hideLoading();
								uni.navigateBack()
							}, 1000);
						} else {
							uni.hideLoading();
						}
						WeixinJSBridge.log(response.err_msg);
					}
				);
			},
			isCheckPay(status, name, order) {
				if (status == 0) {
					this.setPayment(name, order);
				} else {
					uni.hideLoading();
					uni.showToast({
						title: '支付信息有误',
						icon: 'none'
					});
				}
			},
			setPayment(name, order) {
				console.log(name, '*-*-*', order)
				uni.requestPayment({
					provider: name,
					orderInfo: order, //微信、支付宝订单数据
					success: function(res) {
						console.log(res)
						uni.hideLoading();
						uni.showLoading({
							title: '支付成功'
						});
						setTimeout(function() {
							uni.navigateBack()
						}, 1000);
					},
					fail: function(err) {
						console.log(err)
						uni.hideLoading();
					},
					complete() {
						uni.hideLoading();
					}
				});
			},
		}
	}
</script>

<style lang="less">
	page {
		background: #F5F5F5;
	}

	.bg {
		background: #FFFFFF;
	}

	.boxa {
		background: #FFFFFF;
		border-radius: 24rpx;
		margin: 30rpx;
		padding: 30rpx 0;
	}

	.box {
		background: #FFFFFF;
		border-radius: 24rpx;
		padding: 30rpx;
		margin: 30rpx;

	}

	.titls {
		font-size: 32rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #333333;
	}

	.bb {
		width: 110rpx;
		height: 38rpx;
		// background: #49A5FF;
		// opacity: 0.32;
		background-color: rgba(73, 165, 255, 0.3);
		border-radius: 4rpx;
		font-size: 24rpx;
		font-family: PingFang SC;
		font-weight: 500;
		color: #359CFF;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-left: 10rpx;
	}

	.addbox {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 30rpx;
		height: 110rpx;
		border-radius: 16rpx;

	}

	.add_cont {
		display: flex;
		align-items: center;

	}

	.add_tit {
		font-size: 30rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #333333;
	}

	.bg {
		background: #F5F5F5;
	}

	.bgs {
		background: #F5F8FF;

	}

	.green {
		width: 16rpx;
		height: 16rpx;
		background: #1FC657;
		border-radius: 50%;
		margin-right: 20rpx;
	}

	.orgin {
		width: 16rpx;
		height: 16rpx;
		background: #FBAC04;
		border-radius: 50%;
		margin-right: 20rpx;
	}

	.tabber {
		background: #FFFFFF;
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		z-index: 9;
		display: flex;
		align-items: center;
		justify-content: end;
		padding: 10rpx 30rpx;

		.btn {
			flex: 1;
			// width: 330rpx;
			height: 78rpx;
			border: 1px solid #A0A0A0;
			border-radius: 16rpx;
			font-size: 28rpx;
			font-family: PingFang SC;
			font-weight: bold;
			color: #9C9C9C;
			display: flex;
			align-items: center;
			justify-content: center;
		}

		.btn1 {
			// width: 330rpx;
			flex: 1;
			height: 78rpx;
			background: #346EF6;
			border-radius: 16rpx;
			font-size: 28rpx;
			font-family: PingFang SC;
			font-weight: bold;
			color: #FFFFFF;
			display: flex;
			align-items: center;
			justify-content: center;
		}

		.btn2 {
			flex: 1;
			// width: 330rpx;
			height: 78rpx;
			background: #3699FF;
			border-radius: 16rpx;
			font-size: 28rpx;
			font-family: PingFang SC;
			font-weight: bold;
			color: #FFFFFF;
			display: flex;
			align-items: center;
			justify-content: center;
		}
	}

	/* 支付弹框 */
	.popup_pay {
		width: 100%;
	}

	.pay_btns {
		width: 90%;
		margin: 0 auto 40rpx;
		text-align: center;
		background: #3699FF;
		height: 80rpx;
		border-radius: 16rpx;
		color: #ffffff;
		line-height: 80rpx;
		margin-top: 20rpx;
	}
</style>