<template>
	<view>
		<view class="content" style="position: relative;">
			<!-- #ifdef APP -->
			<cover-view class="headTop">
				<cover-view class="flex align-center padding-lr" @click="openMap()" v-if="isDw&&!ordershow&&!driverShow"
					style="margin-top: 70rpx;height: 80rpx;">
					<cover-view style="color: #333333;"> {{city?city:'暂无地址'}} </cover-view>
					<cover-image src="../../static/image/right.png"
						style="width: 14rpx;height: 22rpx;margin-left: 8rpx;">
					</cover-image>
				</cover-view>
				<cover-view class="padding-lr" v-else @click="openFest()"
					style="margin-top: 80rpx;height:65rpx;width: 80rpx;">
					<cover-image src="../../static/image/icon_back.png"
						style="width:18rpx;height: 28rpx;margin-left: 8rpx;">
					</cover-image>
				</cover-view>
			</cover-view>
			<!-- #endif -->
			<!-- #ifndef APP -->
			<view class="headTop">
				<view class="flex align-center" @click="openMap()" v-if="isDw&&!ordershow&&!driverShow">
					<view> {{city}} </view>
					<image src="../../static/image/right.png" style="width: 14rpx;height: 22rpx;margin-left: 8rpx;">
					</image>
				</view>
				<view v-else @click="openFest()">
					<image src="../../static/image/icon_back.png" style="width:18rpx;height: 28rpx;margin-left: 8rpx;">
					</image>
				</view>
			</view>
			<!-- #endif -->

			<!-- #ifndef MP-WEIXIN  -->
			<view class="page-body">
				<view class="page-section page-section-gap">
					<map id="map" style="width: 100%;" :style="!driverShow?'height: 55vh':'height: 75vh'"
						:latitude="latitude" :longitude="longitude" :markers="covers" :show-location="true"
						:polyline="polyline" :polygons="polylist">
						<!-- <cover-view style="position: fixed;top: 345px;right: 30px;" @click="onToGetLocation()">
							<image class="img-map1" src="../../static/image/dw.png">
							</image>
						</cover-view> -->
					</map>
				</view>
			</view>

			<!-- #endif -->

			<!-- #ifdef MP-WEIXIN -->
			<view class="page-body">
				<view class="page-section page-section-gap">
					<map id="map" style="width: 100%; " :style="!driverShow?'height: 55vh':'height: 75vh'"
						:latitude="latitude" :longitude="longitude" :markers="covers" :show-location="true"
						:customCallout="customCallout" :polygons="polylist" :polyline="polyline">
						<!-- <cover-view style="position: fixed;bottom: 430rpx;right: 60rpx;" @click="onToGetLocation()">
							<image class="img-map1" src="../../static/image/dw.png">
							</image>
						</cover-view> -->
					</map>
				</view>
			</view>
			<!-- #endif -->


			<view class="controls-title">
				<view class="orderb" v-if="haveorder.length!=0&&!driverShow" @click="goOrerDer()">
					<view>您当前有一个正在进行的订单</view>
					<view class="flex align-center">
						<view class="text-sm">立即查看</view>
						<image src="../../static/image/go.png" style="width: 14rpx;height: 24rpx;margin-left: 5rpx;">
						</image>
					</view>
				</view>

				<view v-if="popupshow">
					<view class="controls-box">
						<view class="controls-tabs">
							<view v-for="(item,index) in classifyLsit" :key="index" v-if="index<4"
								@tap="change(index,item)" class="atabs_ds">
								<view>{{item.name}}</view>
								<view :class="{btna:current == index}"></view>
							</view>
						</view>

						<view style="color: #999999;" class="flex align-center justify-center margin-bottom-sm"
							v-if="current==1" @click="godaijia()">
							<view v-if="daijiao.length==0">替朋友叫代驾</view>
							<view v-else>{{daijiao.name}} {{daijiao.phone}}</view>

							<image src="../../static/image/go.png"
								style="width: 14rpx;height: 21rpx;margin-left: 10rpx;">
							</image>
						</view>

						<view style="color: #999999;" class="flex align-center justify-center margin-bottom"
							v-if="current==2" @click="openTime()">
							<view v-if="!yuyueTime">选择预约时间</view>
							<view v-else style="color: #333333;">{{yuyueTime}}</view>
							<image src="../../static/image/go.png"
								style="width: 14rpx;height: 21rpx;margin-left: 10rpx;">
							</image>
						</view>
						<view class="addbox " :class="current!=0&&addlist?'bgs':''" @click="changeAdd(current,1)">
							<view class="add_cont">
								<view class="green"></view>
								<view class="" v-if="addlist.length==0">正在获取上车点...</view>
								<view class="add_tit" v-else>
									<view class="flex align-center">
										从<view class="text-cut" style="color: #346EF6;max-width: 420rpx;">
											{{addlist.address}}
										</view>出发
									</view>
									<view class="text-sm" style="color: #999999;font-weight: 500;margin-top: 5rpx;"
										v-if="riderNumber">
										附近有{{riderNumber}}个司机
									</view>
								</view>
							</view>
							<view>
								<image src="../../static/image/go.png" style="width: 16rpx;height: 25rpx;">
								</image>
							</view>
						</view>

						<view class="addbox bg" @click="changeAdd(current,2)">
							<view class="add_cont">
								<view class="orgin"></view>
								<view class="add_tit" v-if="addlists.length==0">输入代驾目的地</view>
								<view class="add_tit" v-else>{{addlists.address}}</view>
							</view>
							<view>
								<image src="../../static/image/go.png" style="width: 16rpx;height: 25rpx;">
								</image>
							</view>
						</view>
						<!-- <view class="text-center text-lg margin-tb" v-if="current==2">预估<text class="text-bold"
								style="font-size: 48rpx;">18</text>元</view> -->
					</view>

					<view class="margin-lr">
						<swiper class="swiper" autoplay="1500" :indicator-dots="true" :circular='true'
							indicator-active-color="#ffffff" indicator-color="#cccccc">
							<swiper-item class="swiper-wrap" v-for="(item,index) in bannerList" :key='index'>
								<image :src="item.imageUrl" style="width: 100%;border-radius: 8upx;height: 161upx;"
									@click="goWeb(item.url)" mode="aspectFill"></image>
							</swiper-item>
						</swiper>
					</view>
				</view>

				<view v-if="ordershow">
					<view class="controls-box">
						<view class="padding ">
							<view class="text-lg text-bold text-center">您好，预估里程时长{{basicPrice.duration}}分钟</view>
							<view class="text-center text-lg margin-top">预估
								<text class="text-bold" style="font-size: 48rpx;">{{basicPrice.allMoney}}</text>元
							</view>

							<view class="flex align-center justify-between " style="width: 50%;margin: 0 auto;">
								<view class="flex align-center justify-center margin-top" @click="openmingxi">
									<view class="text-sm" style="color: #999999;">查看明细</view>
									<image src="../../static/image/go.png"
										style="width: 10rpx;height: 15rpx;margin-left: 8rpx;">
									</image>
								</view>
								<view class="flex align-center justify-center margin-top" @click="openRemk">
									<view class="text-sm" style="color: #999999;">备注</view>
									<image src="../../static/image/go.png"
										style="width: 14rpx;height: 21rpx;margin-left: 8rpx;">
									</image>
								</view>
							</view>
							<view class="bg flex align-center justify-between margin-top" v-if="current==1"
								style="border-radius: 16rpx;padding: 20rpx 45rpx;">
								<view class="flex align-center" @click="godaijia">
									<view style="color: #666666;">{{daijiao.phone}}</view>
									<image src="../../static/image/go.png"
										style="width: 14rpx;height: 21rpx;margin-left: 8rpx;">
									</image>
								</view>
								<view class="line"></view>
								<view class="flex align-center" @click="selectShow = true">
									<view style="color: #666666;">{{payUsername}}</view>
									<image src="../../static/image/go.png"
										style="width: 14rpx;height: 21rpx;margin-left: 8rpx;">
									</image>
								</view>
							</view>
						</view>
					</view>

					<view class="flex align-center justify-end margin-lr margin-top-xs" @click="openYouhui">
						{{-couponName?-couponName:'使用优惠劵'}}
						<image src="../../static/image/right.png" style="width: 14rpx;height: 24rpx;margin-left: 5rpx;">
						</image>
					</view>

					<view class="flex align-center justify-center padding-top">
						<view class="flex align-center " style="font-size: 26upx;">
							<image v-if="showAgree" @tap="isShowAgree"
								src="https://api.shengqianxiong.com.cn/img/20201112/669aa8946cfb4ebdb459d57193c0ebca.png"
								style="width: 32upx;height: 32upx;"></image>
							<image v-else @tap="isShowAgree"
								src="https://api.shengqianxiong.com.cn/img/20201112/1e9102fc891f4d86a13c7b2ba6921cba.png"
								style="width: 32upx;height: 32upx;"></image>
							<view class="margin-left-xs">我已阅读并同意<text style="color: #346EF6;"
									@click="goWeb('/my/setting/sijixieyi')">《代驾服务协议》</text></view>
						</view>
					</view>
					<view class="btn" @click="payorder()">呼叫代驾</view>
					<view class="text-center margin-top-sm" @click="openFest()">取消呼叫</view>
				</view>

			</view>

			<view class="margin-lr margin-top" style="background: #FFFFFF;border-radius: 24rpx;" v-if="driverShow">
				<view class=" flex align-start justify-between " style="padding: 40rpx 30rpx 40rpx;">
					<view>
						<view class="text-bold" style="font-size: 40rpx;">正在为您全力寻找司机</view>
						<view class="text-lg margin-top-xs flex align-center ">已等待
							<smh-timer ref="timer" @timing="timing" :auto="true"></smh-timer>
							<!-- <text style="color: #346EF6;">00:02</text> -->
						</view>
					</view>
					<view class="qxdj" @click="qxshow = true">取消代驾</view>
				</view>
			</view>


			<!-- 新人红包弹框 -->
			<cover-view class="hongbao" v-if="HBShow">
				<cover-view style="width: 52%;margin: 0 auto;position: relative;">
					<cover-view @click="HBShow=false"
						style="position: absolute;right: -10rpx;top: -10rpx;font-size: 32rpx;font-weight: bold;z-index: 999;">X
					</cover-view>
					<cover-image src="../../static/image/hb_bg.png" class="hb_img" @click="takemoney()"></cover-image>
				</cover-view>
			</cover-view>
			<!-- 当前正在进行弹框 -->
			<cover-view class="hongbao" v-if="hoveorderShow">
				<cover-view style="width: 60%;margin: 0 auto;position: relative;">
					<!-- <cover-view @click.stop="hoveorderShow=false"
						style="position: absolute;right: 0rpx;top: 0rpx;font-size: 32rpx;font-weight: bold;z-index: 999;">X
					</cover-view> -->
					<cover-image src="../../static/image/hoveorderbg.png" class="hoveorderimg"
						@click="goOrerDer()"></cover-image>
				</cover-view>
			</cover-view>
			<view v-else></view>
		</view>

		<u-popup v-model="payshow" mode="center" border-radius="14" width="560rpx" :closeable="true">
			<view class="padding text-center">
				<image src="../../static/image/yue.png" style="width: 209rpx;height: 191rpx;"></image>
				<view class="margin-top" style="font-size: 38rpx;font-weight: bold;">
					<view>您的账户余额不足</view>
					<view>请先去充值</view>
				</view>
				<view class="czbtn">去充值</view>
			</view>
		</u-popup>

		<!-- #ifdef APP -->
		<cover-view v-if="qxshow" class="flex align-center justify-center" @click="closeHaveorder(2)"
			style="background: rgba(0, 0, 0, 0.2);width: 100%;height: 100vh;position: absolute;top: 0;left: 0;z-index: 9;">
			<cover-view class="" style="background: #FFFFFF;border-radius: 24rpx;width: 580rpx;">
				<cover-view class="padding text-center">
					<cover-image src="../../static/image/qxdj.png" style="width: 406rpx;height: 181rpx;"></cover-image>
					<cover-view class="margin-top-sm" style="font-size: 32rpx;font-weight: bold;">
						取消后重新呼叫可能会耽
					</cover-view>
					<cover-view class="margin-top-xs" style="font-size: 32rpx;font-weight: bold;">
						误您的出行时间，是否要取消？
					</cover-view>
					<cover-view class="flex justify-between margin-top-60">
						<cover-view class="btns" @click="qxshow=false">暂不取消</cover-view>
						<cover-view class="btnss" @click="bindorderOff">确认取消</cover-view>
					</cover-view>
				</cover-view>
			</cover-view>
		</cover-view>
		<cover-view v-else></cover-view>
		<!-- #endif -->

		<!-- #ifndef APP -->
		<u-popup v-model="qxshow" mode="center" border-radius="14" width="580rpx" :closeable="true">
			<view class="padding text-center">
				<image src="../../static/image/qxdj.png" style="width: 406rpx;height: 181rpx;"></image>
				<view class="margin-top-sm" style="font-size: 32rpx;font-weight: bold;">
					取消后重新呼叫可能会耽
					误您的出行时间，是否要取消？
				</view>
				<view class="flex justify-between margin-top-60">
					<view class="btns" @click="qxshow=false">暂不取消</view>
					<view class="btnss" @click="bindorderOff">确认取消</view>
				</view>
			</view>
		</u-popup>
		<!-- #endif -->

		<!-- #ifdef APP -->
		<cover-view v-if="moneyShow" class="flex align-center justify-center" @click="closeHaveorder(1)"
			style="background: rgba(0, 0, 0, 0.2);width: 100%;height: 100vh;position: absolute;top: 0;left: 0;z-index: 9;">
			<cover-view style="width: 520rpx;background: #FFFFFF;border-radius: 24rpx;position: relative;z-index: 99;">
				<cover-view class="text-lg text-bold text-center margin-top">价格明细</cover-view>
				<cover-view class="margin-tb" style="padding-left: 100rpx;">
					<cover-view class="margin-bottom-xs">总费用：￥{{basicPrice.allMoney}}</cover-view>
					<cover-view class="margin-bottom-xs">总公里数：{{basicPrice.distance}}km</cover-view>
					<cover-view class="margin-bottom-xs">基础公里数：{{basicPrice.kilometer}}km</cover-view>
					<cover-view class="margin-bottom-xs">基础起步价：￥{{basicPrice.initMoney}}</cover-view>
					<cover-view class="margin-bottom-xs">超出部分公里数：{{basicPrice.metre}}km</cover-view>
					<cover-view class="margin-bottom-xs">超出部分每公里价格：￥{{basicPrice.unitOverMoney}}</cover-view>
					<cover-view class="margin-bottom-xs">超出部分收费总金额：￥{{basicPrice.otherMoney}}</cover-view>
					<cover-view class="margin-bottom-xs" v-if="basicPrice.isNight==1">当前是夜间时段 </cover-view>
					<cover-view class="margin-bottom-xs" v-if="basicPrice.isBeforeDawn==1">当前是凌晨时段 </cover-view>
					<cover-view class="margin-bottom-xs">预估总时长费：￥{{basicPrice.durationAllMoney}}</cover-view>
					<cover-view>每分钟时长费：￥{{basicPrice.durationMoney}}</cover-view>
				</cover-view>
				<cover-view class="margin-bottom flex align-center justify-center">
					<cover-image @click.stop="closeHaveorder(1)"
						src="https://daijia.xianmaxiong.com/file/uploadPath/2023/07/24/5f4da4be9181f414b57ad4b57dd90ffa.png"
						style="width: 50rpx;height: 50rpx;"></cover-image>
				</cover-view>
			</cover-view>
		</cover-view>
		<cover-view v-else></cover-view>
		<!-- #endif -->


		<!-- #ifndef APP -->
		<u-popup v-model=" moneyShow" mode="center" width="520rpx" border-radius="24" :closeable="true">
			<view class="padding">
				<view class="text-lg text-bold text-center ">价格明细</view>
				<view class="margin-top" style="padding-left: 100rpx;">
					<view class="margin-bottom-xs">总费用：￥{{basicPrice.allMoney}}</view>
					<view class="margin-bottom-xs">总公里数：{{basicPrice.distance}}km</view>
					<view class="margin-bottom-xs">基础公里数：{{basicPrice.kilometer}}km</view>
					<view class="margin-bottom-xs">基础起步价：￥{{basicPrice.initMoney}}</view>
					<view class="margin-bottom-xs">超出部分公里数：{{basicPrice.metre}}km</view>
					<view class="margin-bottom-xs">超出部分每公里价格：￥{{basicPrice.unitOverMoney}}</view>
					<view class="margin-bottom-xs">超出部分收费总金额：￥{{basicPrice.otherMoney}}</view>
					<view class="margin-bottom-xs" v-if="basicPrice.isNight==1">当前是夜间时段 </view>
					<view class="margin-bottom-xs" v-if="basicPrice.isBeforeDawn==1">当前是凌晨时段 </view>
					<view class="margin-bottom-xs">预估总时长费：￥{{basicPrice.durationAllMoney}}</view>
					<view>每分钟时长费：￥{{basicPrice.durationMoney}}</view>
				</view>

			</view>
		</u-popup>
		<!-- #endif -->
		<u-popup v-model="youhuiShow" mode="bottom" border-radius="14" height="650rpx" :closeable="true">
			<view style="padding: 30rpx 30rpx 50rpx 30rpx;background: #F5F8FC;">
				<view class="text-lg text-bold text-center">优惠劵</view>
				<view class="flex align-center justify-end">
					<view class="deteJuan" @click="deteYouhuj">不使用优惠劵</view>
				</view>
				<scroll-view :scroll-top="scrollTop" scroll-y="true" class="scroll-Y" @scrolltoupper="upper"
					@scrolltolower="lower" @scroll="scroll">
					<view class="money_box" v-for="(item,index) in youhuiList" :key="index">
						<view class="box_tit">
							<view class="money_name">{{item.couponName}}</view>
							<view class="money_price"><text>￥</text>{{item.money}}</view>
						</view>
						<view class="money_data" v-if="item.expirationTime">有效期至{{item.expirationTime}}
						</view>
						<view class="money_data" v-else>永久有效</view>
						<view class="money_line">
							<u-line direction="row" color="#E6E6E6" border-style="dashed" />
						</view>
						<view class="box_bottom">
							<view class="money_use">满{{item.minMoney}}元可使用</view>
							<view class="money_btn">
								<view class="lj_use" @click="useryouhui(item)">
									立即使用
								</view>
							</view>
						</view>
					</view>
				</scroll-view>

			</view>
		</u-popup>
		<u-popup v-model="remkShow" mode="bottom" border-radius="14" :closeable="true">
			<view style="padding: 30rpx 30rpx 50rpx 30rpx;">
				<view class="text-lg text-bold text-center">备注</view>
				<view class="margin-top-xl" style="background: #F5F8FC;padding: 20rpx 25rpx;border-radius: 24rpx;">
					<u-input v-model="remarks" type="textarea" maxlength="100" placeholder="请输入备注(备注车型或者要求驾照,100字以内)"
						height="240" />
				</view>
				<view class="remkbtn" @click="remkShow = false">确定</view>
			</view>
		</u-popup>



		<!-- 支付选择 -->
		<u-select v-model="selectShow" mode="single-column" :list="list" @confirm="confirmPay"></u-select>

		<u-picker v-model="Timeshow" mode="time" :params="params" @confirm="confirm"></u-picker>
	</view>
</template>

<script>
	// 引入SDK核心类，地图组件 
	import QQMapWX from '@/components/qqmap-wx-jssdk1.2/qqmap-wx-jssdk.js'
	export default {
		data() {
			return {
				is_focus: false,
				remarks: '', //需求备注
				HBShow: false,

				id: 0, // 使用 marker点击事件 需要填写id
				title: 'map',
				latitude: '',
				longitude: '',
				// iconPath: '../../static/image/qi.png',
				qqmapsdk: {}, // 腾讯地图小程序的SDK
				covers: [], // 地图上面的标记点
				to: { // 目的地坐标
					latitude: '',
					longitude: '',
				},
				polyline: [], //路线
				polylines: '',
				polylist: [],

				classifyLsit: [{
					id: 1,
					name: '即时代驾'
				}, {
					id: 2,
					name: '朋友代叫'
				}, {
					id: 3,
					name: '预约代驾'
				}],
				current: 0,
				value0: '',
				value1: '',
				value2: '',
				value3: '',
				type: 'text',
				clearable: false,
				riderNumber: 0,


				// 用户红包
				newUserFlag: 2,

				token: '',
				bannerList: [],
				noticeList: [],


				tuiguang: '',
				tuiguangImg: '',
				arr: [],
				showModal1: true,
				invitationCode: '',

				city: '',
				showAgree: false, //协议是否选择
				payshow: false,
				qxshow: false,
				isDw: true,
				yuyueTime: '',
				Timeshow: false,
				params: {
					year: true,
					month: true,
					day: true,
					hour: true,
					minute: true,
					second: false
				},

				addlist: [], //出发地址
				addressId: '',
				addlists: [], //目的地
				addressIds: "",
				popupshow: true,
				ordershow: false,
				basicPrice: [], //基础价格
				driverShow: false,

				moneyShow: false, //价格明细
				haveorder: [], //正在进行订单
				youhuiShow: false, //优惠劵弹框
				daijiao: [], //代叫信息

				selectShow: false, //下单人切换
				list: [{
						value: '1',
						label: '我来支付'
					},
					{
						value: '2',
						label: '对方支付'
					}
				],
				payUser: 0,
				payUsername: '我来支付',
				youhuiList: [], //优惠劵
				couponName: '',
				couponId: '',
				scrollTop: 0,
				old: {
					scrollTop: 0
				},
				timer: '',
				hoveorderShow: false, //正在进行订单
				isFist: true,
				key: '',

				remkShow: false, //备注弹框
				remarks: '',
				district: '',
				province: '',

			}
		},
		onLoad(e) {
			let that = this
			// #ifdef MP-WEIXIN
			if (e.scene) {
				const scene = decodeURIComponent(e.scene);
				that.$queue.setData('inviterCode', scene.split(',')[0]);
			}
			// #endif

			// 获取邀请码保存到本地
			if (e.invitation) {
				that.$queue.setData('inviterCode', e.invitation);
			}
			// this.invitationCode = this.$queue.getData('invitationCode');
			that.$Request.getT('/app/common/type/128').then(res => {
				if (res.code == 0 && res.data && res.data.value) {
					that.key = res.data.value
					// 实例化API核心类
					that.qqmapsdk = new QQMapWX({
						key: that.key // 自己申请的key值
					})
				}
			});



			uni.getLocation({
				type: 'gcj02',
				geocode: true, //设置该参数为true可直接获取经纬度及城市信息
				success: function(res) {
					console.log(res, '地理位置')
					that.latitude = res.latitude
					that.longitude = res.longitude
					uni.setStorageSync('latitude', res.latitude)
					uni.setStorageSync('longitude', res.longitude)

					// #ifdef APP-PLUS
					that.city = res.address.city
					uni.setStorageSync('city', res.address.city)
					that.selectCity(that.longitude, that.latitude);

					// #endif

					// #ifdef H5
					that.selectCity(that.longitude, that.latitude);
					// #endif

					// #ifdef MP-WEIXIN
					let add
					uni.request({
						url: 'https://apis.map.qq.com/ws/geocoder/v1/?location=' + that.latitude +
							',' + that.longitude + '&key=' + that.key,
						success(re) {
							// console.log(re, '---------89')
							if (re.statusCode === 200) {
								if (re.data.status == 121) {
									that.selectCity(that.longitude, that.latitude);
								} else {
									let citydata = re.data.result.address_component.city
									console.log("获取城市名称成功", citydata)
									that.city = citydata ? citydata : '未知'
									that.district = re.data.result.address_component.district
									that.province = re.data.result.address_component.province
									that.getOpencity()
									that.getquyuList()
									uni.setStorageSync('city', citydata)
									add = {
										// address: res.data.province + res.data.city + res.data.district, //地址
										address: re.data.result.address_reference.landmark_l2
											.title,
										lng: that.longitude, //地址经度
										lat: that.latitude, //地址维度
										addressId: '',
										addressDefault: '',
										county: re.data.result.address_component.district,
										province: re.data.result.address_component.province,
										city: re.data.result.address_component.city
									}
									that.addlist = add
									that.covers = [{
										id: 1,
										latitude: that.latitude,
										longitude: that.longitude,
										iconPath: '../../static/image/qi.png',
										width: 32,
										height: 44,
										callout: { //自定义标记点上方的气泡窗口 点击有效
											content: re.data.result.address_reference
												.landmark_l2
												.title, //文本
											color: '#333333', //文字颜色
											fontSize: 12, //文本大小
											padding: 10, //附近留白
											borderRadius: 6, //边框圆角
											bgColor: '#FFFFFF', //背景颜色
											display: 'ALWAYS', //常显
										},
									}]
								}
							} else {
								console.log("获取信息失败，请重试！")
							}
						}
					});
					// #endif
				},
				fail: function() {
					console.log('获取地址失败')
					uni.startLocationUpdate({
						type: 'gcj02', //gcj02  wgs84
						isHighAccuracy: true, //开启高精度定位(！！！必需)
						geocode: true,
						success: res => {
							uni.onLocationChange(function(res2) {
								console.log('实时纬度：' + res2.latitude);
								console.log('实时经度：' + res2.longitude);
								that.latitude = res2.latitude
								that.longitude = res2.longitude
								uni.setStorageSync('latitude', that.latitude)
								uni.setStorageSync('longitude', that.longitude)
								that.selectCity(that.longitude, that.latitude);

							});
						},
						fail: err => {
							// clearInterval(this.ordertimer)
							console.error('开启小程序接收位置消息失败：', err)
						},
						complete: msg => {
							// clearInterval(this.ordertimer)
							console.log('调用开启小程序接收位置消息 API 完成')
						}
					});
				}
			})
		},
		onHide() {
			clearInterval(this.timer)
		},
		onShow() {
			this.getBannerList()

			this.token = uni.getStorageSync('token')
			if (this.token) {
				this.timer = setInterval(() => {
					this.gethaveOrder() //获取正在进行订单k
				}, 3000);

				this.getuserinfo()
				if (this.isFist) {
					this.isFist = false
					this.gethaveOrders()
				}


				this.getZiZhi() //获取分享文案  图片
				if (uni.getStorageSync('addressId')) {
					this.addressId = uni.getStorageSync('addressId')
					this.getcfd()
				}
				if (uni.getStorageSync('addressIds')) {
					this.addressIds = uni.getStorageSync('addressIds')
					this.getzd()
				}

				if (uni.getStorageSync('daijiao')) {
					this.daijiao = uni.getStorageSync('daijiao')
				}

				this.$Request.getT('/app/common/type/307').then(res => { //用户端订单状态通知 307
					if (res.code == 0) {
						if (res.data && res.data.value) {
							this.arr.push(res.data.value)
						}
					}
				})
				if (this.showModal1) {
					// #ifdef MP-WEIXIN
					this.openMsg()
					// #endif
				}
			} else {
				clearInterval(this.timer)
			}

		},
		onReady() {
			this.map = uni.createMapContext("map", this)
		},
		onShareAppMessage(res) { //发送给朋友
			return {
				title: this.tuiguang,
				path: '/pages/index/index?invitation=' + this.invitationCode,
				imageUrl: this.tuiguangImg,
			}
		},
		onShareTimeline(res) { //分享到朋友圈
			return {
				title: this.tuiguang,
				path: '/pages/index/index?invitation=' + this.invitationCode,
				imageUrl: this.tuiguangImg,
			}
		},
		methods: {
			// 获取区域范围
			getquyuList() {
				let data = {
					city: this.city,
					// district: this.district,
					province: this.province,
				}
				this.$Request.get("/app/banArea/getBanAreaList", data).then(res => {
					if (res.code == 0) {
						let arr = []
						if (res.data.records && res.data.records.length > 0) {
							res.data.records.forEach((item, index) => {
								let dataList = [];
								item.points.forEach((ite, ind) => {
									dataList.push({
										latitude: ite.lat,
										longitude: ite.lng
									})
								})
								let ps = {
									points: dataList,
									strokeColor: "#0186FF",
									strokeWidth: 3,
									fillOpacity: 0.4,
								};
								arr.push(ps)

							});

							this.polylist = arr

						}

					}
				});
			},
			getOpencity() {
				this.$Request.getT('/app/cityInfo/isOpenCity?cityName=' + this.district).then(res => {
					if (res.code === 0) {
						if (!res.data) {
							uni.showToast({
								title: '当前城市未开通服务',
								icon: 'none',
								duration: 2000
							})
						}
					}
				});
			},
			openRemk() {
				this.remkShow = true
			},
			closeHaveorder(index) {
				console.log('99999999999')
				if (index == 1) {
					this.moneyShow = false
				} else if (index == 2) {
					this.qxshow = false
				}

			},
			openYouhui() {
				if (this.youhuiList.length == 0) {
					uni.showToast({
						title: '当前暂无可使用的优惠劵',
						icon: 'none'
					})
					return
				}
				this.youhuiShow = true
			},
			openMap() {
				console.log('--44444444444')
				let that = this
				uni.chooseLocation({
					success: function(res) {
						console.log('位置名称：' + res.name);
						console.log('详细地址：' + res.address);
						console.log('纬度：' + res.latitude);
						console.log('经度：' + res.longitude);
						that.latitude = res.latitude
						that.longitude = res.longitude
						that.selectCity(that.longitude, that.latitude);


					}
				});
			},
			confirmPay(e) {
				// console.log(e,'-------')
				if (e[0].value == 1) {
					this.payUser = 0
				} else if (e[0].value == 2) {
					this.payUser = 1
				}
				this.payUsername = e[0].label
			},
			goOrerDer() {
				uni.navigateTo({
					url: '/my/order/pay?indentNumber=' + this.haveorder.indentNumber
				})
				this.hoveorderShow = false
			},
			openFest() { //返回首页下单
				console.log('.........222222')
				this.addlists = []
				this.covers = []
				this.polyline = []
				this.ordershow = false
				this.popupshow = true
				this.driverShow = false
				this.selectCity(this.longitude, this.latitude);
				this.gethaveOrder()
			},
			openmingxi() { //查看价格明细
				this.moneyShow = true
			},


			confirm(e) {
				console.log(e)
				this.yuyueTime = e.month + '月' + e.day + '日' + ' ' + e.hour + ':' + e.minute
				this.hopeTime = e.year + '-' + e.month + '-' + e.day + ' ' + e.hour + ':' + e.minute + ':00'
			},
			openTime() {
				this.Timeshow = true
			},
			godaijia() {
				uni.navigateTo({
					url: '/my/daijia/index'
				})
			},
			changeAdd(current, index) {
				if (this.token) {
					if (this.current == 1) {
						if (this.daijiao.length == 0) {
							uni.showToast({
								title: '请填写代叫信息',
								icon: 'none'
							})
							return
						}
					}
					if (this.current == 2) {
						if (!this.yuyueTime) {
							uni.showToast({
								title: '请选择预约时间',
								icon: 'none'
							})
							return
						}
					}
					uni.navigateTo({
						url: '/my/address/index?type=' + current + '&index=' + index
					})
					this.hoveorderShow = false
				} else {
					uni.navigateTo({
						url: '/pages/my/register'
					})
				}

			},
			isShowAgree() {
				//是否选择协议
				this.showAgree = !this.showAgree;
			},
			// 分享文案和图片
			getZiZhi() {
				this.$Request.getT('/app/common/type/276').then(res => {
					if (res.code === 0) {
						this.tuiguang = res.data.value;
					}
				});
				this.$Request.getT('/app/common/type/277').then(res => {
					if (res.code === 0) {
						this.tuiguangImg = res.data.value;
					}
				});
			},
			goWeb(url) {
				// #ifdef MP-WEIXIN
				if (uni.getStorageSync('sendMsg')) {
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
				// console.log(url.indexOf('/pages/') !== -1)
				// return
				if (url.indexOf('/pages/') !== -1 || url.indexOf('/pageA/') !== -1 || url.indexOf('/my/') !== -1) {
					uni.navigateTo({
						url
					});
				} else {
					//#ifndef H5
					uni.navigateTo({
						url: '/pages/index/webView?url=' + url
					});
					//#endif
					//#ifdef H5
					window.location.href = url;
					//#endif
				}
			},

			// 获取轮播图
			getBannerList() {
				this.$Request.get("/app/banner/selectBannerList?classify=1&state=1").then(res => {
					if (res.code == 0) {
						this.bannerList = res.data
					}
				});
			},

			//右下角定位按钮的点击事件
			onToGetLocation() {
				let mapContext = uni.createMapContext('map');
				mapContext.moveToLocation(); //moveToLocation将地图中心移动到当前定位点，需要配合map组件的show-location使用
			},
			selectCity(longitude, latitude) {
				let data
				this.$Request.getT('/app/Login/selectCity?lat=' + latitude + '&lng=' + longitude).then(res => {
					if (res.code === 0) {
						this.city = res.data.city ? res.data.city : res.data.province
						this.district = res.data.district
						this.province = res.data.province
						this.getOpencity()
						this.getquyuList()

						uni.setStorageSync('city', res.data.city)
						this.covers = [{
							id: 1,
							latitude: latitude,
							longitude: longitude,
							iconPath: '../../static/image/qi.png',
							width: 32,
							height: 44,
							callout: { //自定义标记点上方的气泡窗口 点击有效
								content: res.data.address, //文本
								color: '#333333', //文字颜色
								fontSize: 10, //文本大小
								padding: 10, //附近留白
								borderRadius: 6, //边框圆角
								bgColor: '#FFFFFF', //背景颜色
								display: 'ALWAYS', //常显
							},
						}]
						data = {
							address: res.data.address,
							lng: longitude, //地址经度
							lat: latitude, //地址维度
							addressId: '',
							addressDefault: '',
							county: res.data.district
						}
						this.addlist = data
						// console.log('----')
					}
				});
			},
			// 查询附近司机
			getAdd(lng, lat) {
				this.$Request.getT('/app/indent/findVicinityRider?lng=' + lng + '&lat=' + lat).then(res => {
					if (res.code == 0) {
						this.riderNumber = res.data.count
					}
				});
			},
			//获取出发地
			getcfd() {
				this.$Request.getT('/app/address/getAddressInfo?addressId=' + this.addressId).then(res => {
					if (res.code == 0 && res.data) {
						this.addlist = res.data

						this.covers = [{
							id: 1,
							latitude: this.addlist.lat,
							longitude: this.addlist.lng,
							iconPath: '../../static/image/qi.png',
							width: 32,
							height: 44,
							callout: { //自定义标记点上方的气泡窗口 点击有效
								content: this.addlist.address, //文本
								color: '#333333', //文字颜色
								fontSize: 10, //文本大小
								padding: 10, //附近留白
								borderRadius: 6, //边框圆角
								bgColor: '#FFFFFF', //背景颜色
								display: 'ALWAYS', //常显
							},
						}]
						this.getAdd(res.data.lng, res.data.lat)
						uni.removeStorageSync('addressId')
						// console.log(this.addlists || this.addlist, this.addlists && this.addlis)
						// return
						if (this.addlists && this.addlists.lng && this.addlist && this.addlist.lng) {
							this.orderPay()
						}
						// uni.getStorageSync('addressId')
					} else {
						this.addlist = []
					}
				});
			},
			//获取目的地
			getzd() {
				this.$Request.getT('/app/address/getAddressInfo?addressId=' + this.addressIds).then(res => {
					if (res.code == 0 && res.data) {
						this.addlists = res.data

						this.to.latitude = res.data.addressLatitude
						this.to.longitude = res.data.addressLongitude
						// console.log(this.addlist,'cccccccccccccc')
						// return
						this.covers = [{
							id: 1,
							latitude: this.addlist.lat,
							longitude: this.addlist.lng,
							iconPath: '../../static/image/qi.png',
							width: 32,
							height: 44,
							callout: { //自定义标记点上方的气泡窗口 点击有效
								content: this.addlist.address, //文本
								color: '#333333', //文字颜色
								fontSize: 10, //文本大小
								padding: 10, //附近留白
								borderRadius: 6, //边框圆角
								bgColor: '#FFFFFF', //背景颜色
								display: 'ALWAYS', //常显
							},
						}, {
							id: 2,
							latitude: res.data.addressLatitude,
							longitude: res.data.addressLongitude,
							iconPath: '../../static/image/zd.png',
							width: 32,
							height: 44,
							callout: { //自定义标记点上方的气泡窗口 点击有效
								content: res.data.address, //文本
								color: '#333333', //文字颜色
								fontSize: 10, //文本大小
								padding: 10, //附近留白
								borderRadius: 6, //边框圆角
								bgColor: '#FFFFFF', //背景颜色
								display: 'ALWAYS', //常显
							},
						}]
						uni.removeStorageSync('addressIds')
						if (this.addlists && this.addlists.lng && this.addlist && this.addlist.lng) {
							this.orderPay()
						}

					} else {
						this.addlists = []
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
			// 计算基础价格
			orderPay() {
				let data = {
					goLng: this.addlist.lng,
					goLat: this.addlist.lat,
					toLng: this.addlists.lng,
					toLat: this.addlists.lat,
					redPacketId: '',
					goCity: this.addlist.county, //出发市
					toCity: this.addlists.county, //目的市
				}
				this.$Request.getT('/app/indent/basicsMoney', data).then(res => {
					if (res.code == 0) {
						this.basicPrice = res.data
						this.polylines = res.data.polyline
						this.gethaveOrder()
						this.popupshow = false
						this.ordershow = true
						this.routePlanning()
						this.getyouhui() //获取优惠劵

					} else {
						uni.showToast({
							title: res.msg,
							icon: 'none'
						})
					}
				});
			},
			//下单
			payorder() {
				if (!this.showAgree) {
					uni.showToast({
						title: '请同意代驾服务协议',
						icon: 'none'
					})
					return
				}

				let data
				let indentType = this.classifyLsit[this.current].id
				data = {
					indentType: indentType,
					shipAddress: this.addlist.address,
					shipAddressLongitude: this.addlist.lng,
					shipAddressLatitude: this.addlist.lat,
					deliveryAddress: this.addlists.address,
					deliveryAddressLongitude: this.addlists.lng,
					deliveryAddressLatitude: this.addlists.lat,
					deliveryAddressId: this.addlists.addressId,
					shipAddressId: this.addlist.addressId,
					shipDistrict: this.addlist.county,
					shipProvince: this.addlist.province,
					shipCity: this.addlist.city,
					remarks: '',
					couponId: this.couponId,
					itemCodeFlag: 1,
					payUser: this.payUser,
					type: '',
					remarks: this.remarks
				}
				if (indentType == 2) {
					data.customPhone = this.daijiao.phone
					data.customName = this.daijiao.name
				}
				if (indentType == 3) {
					data.hopeTime = this.hopeTime
				}
				this.$Request.postJson('/app/indent/addIndent', data).then(res => {
					if (res.code == 0) {

						this.ordershow = false
						this.driverShow = true
						this.isShowAgree = false
						this.gethaveOrder()
						uni.removeStorageSync('daijiao')
						this.daijiao = []
						this.yuyueTime = ''
					} else {
						uni.showToast({
							title: res.msg,
							icon: 'none',
							duration: 1500
						})
					}
				});
			},
			// 查询是否有正在进行的订单
			gethaveOrders() {
				this.$Request.getT('/app/indent/getMyHaveOrder').then(res => {
					if (res.code == 0 && res.data) {
						this.haveorder = res.data
						this.hoveorderShow = true
					} else {
						this.haveorder = []
					}
				});
			},
			gethaveOrder() {
				this.$Request.getT('/app/indent/getMyHaveOrder').then(res => {
					if (res.code == 0 && res.data) {
						this.haveorder = res.data
						// console.log('查询订单', this.haveorder && this.haveorder.riderUserId && this.driverShow)
						if (this.haveorder && this.haveorder.riderUserId && this.driverShow) {
							uni.navigateTo({
								url: '/my/order/pay?indentNumber=' + this.haveorder.indentNumber
							})
							clearInterval(this.timer)
							this.openFest()
						}
					} else {
						this.haveorder = []
					}
				});
			},
			// 取消订单
			bindorderOff() {
				let data = {
					indentNumber: this.haveorder.indentNumber
				}
				this.$Request.postT('/app/indent/userCancleIndent', data).then(res => {
					if (res.code == 0) {
						uni.showToast({
							title: '订单取消成功'
						});
						this.qxshow = false
						this.driverShow = false

						this.openFest()
					} else {
						uni.hideLoading();
						uni.showModal({
							showCancel: false,
							title: '订单失败',
							content: res.msg
						});
					}
				});
			},
			timing(second) {
				//实时返回计时时间
				// console.log(second)
			},
			reset() {
				//重置计时器
				this.$refs.timer.reset()
			},
			start() {
				//手动开启计时器
				this.$refs.timer.start()
			},
			clear() {
				//停止计时器
				this.$refs.timer.clear()
			},
			change(index, item) {
				this.current = index;

			},
			//清空优惠劵
			deteYouhuj() {
				this.couponId = ''
				this.couponName = ''
				this.youhuiShow = false
			},
			//使用优惠劵
			useryouhui(item) {
				this.couponId = item.id
				this.couponName = item.money
				this.youhuiShow = false
			},
			// 获取用户信息
			getuserinfo() {
				this.$Request.getT('/app/userinfo/findUserInfoById').then(res => {
					if (res.code == 0 && res.data) {
						if (res.data.newUserFlag) {
							this.newUserFlag = res.data.newUserFlag
							// console.log(this.newUserFlag)
							if (this.newUserFlag == 1) {
								//是否开启新人优惠劵
								this.$Request.getT('/app/common/type/354').then(res => {
									if (res.code == 0) {
										if (res.data && res.data.value && res.data.value == '是') {
											this.HBShow = true
										}
									}
								});
							} else {
								this.HBShow = false
							}
						}
					}
				});
			},

			getyouhui() { //获取优惠劵
				let indentType = this.classifyLsit[this.current].id
				let data = {
					indentType: indentType,
					estimateMoney: this.basicPrice.allMoney
				}
				this.$Request.getT('/app/couponUser/getMyOrderCouponList', data).then(res => {
					if (res.code === 0) {
						this.youhuiList = res.data.records
					} else {
						uni.showToast({
							title: res.msg,
							icon: "none"
						})
					}
				});
			},
			// 红包
			takemoney() {
				this.$Request.getT('/app/userinfo/newUserCoupon').then(res => {
					if (res.code === 0) {
						this.HBShow = false
						uni.showToast({
							title: '领取成功',
							icon: 'none',
							duration: 3000
						})
						this.getuserinfo()
						// setTimeout(function() {
						// 	uni.navigateTo({
						// 		url: '/pages/my/hongbao/hongbao'
						// 	})
						// }, 100)
					} else {
						uni.showToast({
							title: res.msg,
							icon: "none"
						})
						this.newUserFlag = ''
					}
				});
			},
			upper: function(e) {
				// console.log(e)
			},
			lower: function(e) {
				// console.log(e)
			},
			scroll: function(e) {
				// console.log(e)
				this.old.scrollTop = e.detail.scrollTop
			},
			// 开启订阅消息
			openMsg() {
				console.log('订阅消息')
				var that = this
				uni.getSetting({
					withSubscriptions: true, //是否获取用户订阅消息的订阅状态，默认false不返回
					success(ret) {
						console.log(ret.subscriptionsSetting, '------------------')
						// if (ret.subscriptionsSetting.itemSettings && Object.keys(ret.subscriptionsSetting.itemSettings).length == 2) {
						if (ret.subscriptionsSetting.itemSettings) {
							uni.setStorageSync('sendMsg', true)
							uni.openSetting({ // 打开设置页 
								success(rea) {
									console.log(rea.authSetting)
								}
							});
						} else { // 用户没有点击“总是保持以上，不再询问”则每次都会调起订阅消息
							console.log(99999)
							uni.setStorageSync('sendMsg', false)
							uni.showModal({
								title: '提示',
								content: '为了更好的体验,请绑定消息推送',
								confirmText: '确定',
								cancelText: '取消',
								success: function(res) {
									if (res.confirm) {
										console.log(that.arr)
										wx.requestSubscribeMessage({
											tmplIds: that.arr,
											success(re) {
												console.log(JSON.stringify(re),
													'++++++++++++++')
												var datas = JSON.stringify(re);
												if (datas.indexOf("accept") != -1) {
													console.log(re)
													// uni.setStorageSync('sendMsg', true)
												}
											},
											fail: (res) => {
												console.log(res)
											}
										})
										// uni.setStorageSync('sendMsg', true)
										console.log('确认')
										that.showModal1 = false
									} else if (res.cancel) {
										console.log('取消')
										// uni.setStorageSync('sendMsg', false)
										that.showModal1 = true
									}
								}
							})
						}
					}
				})
			},
		}
	}
</script>

<style lang="less">
	/deep/.swiper {
		height: 160rpx !important;
	}

	.content {
		width: 100%;
		// position: fixed;
		// top: 0upx;
		// left: 0upx;
		// right: 0upx;
		// bottom: 0upx;

	}

	.controls-title {
		width: 100%;
		// height: 55vh;
		background: #F5F8FC;
		// box-shadow: 0px -12px 40px 0px rgba(192, 220, 245, 0.6);
		border-radius: 55upx 55upx 0 0;
		/* #ifdef MP-WEIXIN */
		padding-bottom: 20rpx;
		/* #endif */
	}

	.orderb {
		display: flex;
		align-items: center;
		justify-content: space-between;
		background: #FFFFFF;
		margin: 30rpx 30rpx 0 30rpx;
		border-radius: 24rpx;
		padding: 30rpx;
	}

	.controls-box {
		background: #FFFFFF;
		margin: 30upx 30rpx 20rpx;
		border-radius: 32upx;
		padding-bottom: 1rpx;
	}

	/* tab选项卡 */
	.controls-tabs {
		/* width: 100%; */
		height: 90rpx;
		display: flex;
		font-size: 28rpx;
		overflow-x: auto;
		justify-content: space-between;
		align-items: center;
		margin-top: 10rpx;
		padding: 0 30rpx;
	}

	.atabs_ds {
		text-align: center;
		height: 55rpx;
	}

	.btna {
		background: #346EF6;
		width: 40rpx;
		height: 6rpx;
		/* margin: 9rpx 10rpx 10rpx 29rpx; */
		margin: 5rpx auto;
		border-radius: 55rpx;

	}

	.addbox {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin: 0 30rpx 30rpx;
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

	/* 红包样式 */
	.hongbao {
		width: 100%;
		/* height: 100px; */
		/* background: #007AFF; */
		position: fixed;
		top: 24%;
		/* bottom: 50%; */
		left: 0rpx;
		right: 0rpx;
		/* display: none; */
	}

	.hb_img {
		width: 100%;
		height: 435rpx;
	}

	.hb_btn {
		width: 60%;
		height: 72rpx;
		position: absolute;
		top: 315rpx;
		left: 80rpx;
	}

	.hoveorderimg {
		width: 100%;
		height: 535rpx;
	}

	.img-map1 {
		width: 80rpx;
		height: 80rpx;
	}

	.headTop {
		width: 100%;
		background: #FFFFFF;
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 999;
		/* #ifdef MP-WEIXIN */
		padding: 120rpx 30rpx 25rpx;
		/* #endif */
		/* #ifdef H5 */
		padding: 40rpx 30rpx 25rpx;
		/* #endif */
		/* #ifdef APP */

		padding: 0rpx 0rpx;
		/* #endif */
		display: flex;
		align-items: center;

	}

	.line {
		width: 1rpx;
		height: 63rpx;
		background: #E6E6E6;
	}

	.btn {
		width: 686rpx;
		height: 98rpx;
		margin: 0 auto;
		background: #346EF6;
		border-radius: 16rpx;
		font-size: 32rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #FFFFFF;
		display: flex;
		align-items: center;
		justify-content: center;
		margin: 40rpx auto;
	}

	.czbtn {
		width: 480rpx;
		height: 80rpx;
		background: #346EF6;
		border-radius: 40rpx;
		font-size: 28rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #FFFFFF;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-top: 50rpx;
	}

	.qxdj {
		width: 180rpx;
		height: 70rpx;
		border: 2rpx solid #CCCCCC;
		border-radius: 16rpx;
		font-size: 28rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #666666;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.btns {
		width: 250rpx;
		height: 80rpx;
		border: 2rpx solid #999999;
		border-radius: 40rpx;
		font-size: 28rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #999999;
		display: flex;
		align-items: center;
		justify-content: center;
		line-height: 80rpx;
	}

	.btnss {
		width: 250rpx;
		height: 80rpx;
		background: #346EF6;
		border-radius: 40rpx;
		font-size: 28rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #FFFFFF;
		display: flex;
		align-items: center;
		justify-content: center;
		line-height: 80rpx;
	}

	//优惠劵
	.scroll-Y {
		height: 580rpx;
	}

	.money_box {
		// height: 300rpx;
		// line-height: 300rpx;
		// text-align: center;
		// font-size: 36rpx;
		width: 100%;
		margin: 0 auto;
		background: #ffffff;
		border-radius: 14upx;
		height: 220rpx;
		margin-bottom: 20upx;
		margin-top: 20rpx;
	}

	.box_tit {
		width: 90%;
		margin: 0 auto;
		height: 80upx;
		display: flex;
	}

	.money_name {
		flex: 1;
		display: flex;
		justify-content: left;
		align-items: center;
		font-size: 27rpx;
		font-weight: bold;
		letter-spacing: 2upx;
	}

	.money_price {
		flex: 1;
		display: flex;
		justify-content: flex-end;
		align-items: center;
		font-size: 48upx;
		font-weight: bold;
		color: red;

		text {
			font-size: 30rpx;
		}
	}

	.money_data {
		color: #999999;
		font-size: 24rpx;
		width: 90%;
		margin: 0 auto;
		margin-top: -8upx;
	}

	.u-line {
		width: 90% !important;
		border-bottom-width: 6upx !important;
		margin: 0 auto !important;
		margin-top: 22upx !important;
		margin-bottom: 22upx !important;
	}

	.box_bottom {
		width: 90%;
		margin: 0 auto;
		display: flex;
		height: 40upx;
	}

	.money_use {
		flex: 1;
		color: #999999;
		font-size: 24rpx;
		display: flex;
		justify-content: left;
		align-items: center;
	}

	.lj_use {
		width: 150rpx;
		border: 2rpx solid #FF130A;
		color: #FF130A;
		text-align: center;
		line-height: 48rpx;
		border-radius: 40rpx;
		font-size: 23rpx;
	}

	.deteJuan {
		width: 190rpx;
		border: 2rpx solid #FF130A;
		color: #FF130A;
		text-align: center;
		line-height: 48rpx;
		border-radius: 40rpx;
		font-size: 23rpx;
		margin: 20rpx 0;
	}

	.remkbtn {
		width: 100%;
		height: 80rpx;
		background: #346EF6;
		border-radius: 40rpx;
		font-size: 28rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #FFFFFF;
		display: flex;
		align-items: center;
		justify-content: center;
		line-height: 80rpx;
		margin-top: 40rpx;
	}
</style>