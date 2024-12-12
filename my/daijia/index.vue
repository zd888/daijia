<template>
	<view>
		<view class="box">
			<view class="text-bold" style="font-size: 34rpx;">乘车人</view>
			<view class="margin-top-xs">
				<u-input v-model="phone" type="number" placeholder="手机号" />
			</view>
			<view style="width: 100%;height: 1rpx;background: #E6E6E6;"></view>
			<view class="margin-top-sm">
				<u-input v-model="name" type="text" placeholder="姓名" />
			</view>
		</view>
		<view class="btn" @click="submit()">确认</view>
		<view class="box">
			<view class="text-lg text-bold">代叫服务说明</view>
			<view class="margin-top-sm" v-html="content"></view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				phone: '',
				name: '',
				content: ''
			}
		},
		onLoad() {
			this.$Request.getT('/app/common/type/352').then(res => { //代叫服务说明
				if (res.code == 0) {
					if (res.data && res.data.value) {
						this.content = res.data.value
					}
				}
			});
		},
		onShow() {
			if (uni.getStorageSync('daijiao')) {
				let data = uni.getStorageSync('daijiao')
				this.phone = data.phone
				this.name = data.name

			}
		},
		methods: {
			submit() {
				if (!this.phone) {
					uni.showToast({
						title: '请输入乘车人手机号',
						icon: 'none'
					})
					return
				}
				if (!this.name) {
					uni.showToast({
						title: '请输入乘车人姓名',
						icon: 'none'
					})
					return
				}
				let data = {
					phone: this.phone,
					name: this.name
				}
				uni.setStorageSync('daijiao', data)

				uni.navigateBack()

			}
		}
	}
</script>

<style lang="less">
	page {
		background: #F5F5F5;
	}

	.box {
		background: #FFFFFF;
		border-radius: 24rpx;
		margin: 30rpx;
		padding: 30rpx;
	}

	.btn {
		// width: 100%;
		height: 98rpx;
		background: #346EF6;
		border-radius: 16rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		margin: 30rpx;
		font-size: 32rpx;
		font-family: PingFang SC;
		font-weight: bold;
		color: #FFFFFF;
	}
</style>