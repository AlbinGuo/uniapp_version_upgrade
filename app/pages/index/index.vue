<template>
	<view>
		<u-navbar back-text="返回" title="剑未配妥，出门已是江湖">
			<text style="border: 1px solid red;position: absolute;border-radius:4rpx;padding: 10rpx 20rpx;right:20rpx;">热更新</text>
		</u-navbar>
		<u-swiper :list="list"></u-swiper>
		<u-grid :col="menus.length">
			<u-grid-item v-for="(item,index) in menus" :key="index" @click="goPage(item.open_url)">
				<u-icon :name="item.title" :size="46"></u-icon>
				<view class="grid-text">{{item.description}}</view>
			</u-grid-item>
		</u-grid>
		<button @click="check_verson()">检测更新 - v1.0.7</button>
	</view>
</template>

<script>
	import upAPP from '@/uni_modules/uni-upgrade-center-app/utils/check-update.js'
	export default {
		data() {
			return {
				title: 'Hello',
				list: [],
				menus: []
			}
		},
		onLoad() {
			upAPP()
			this.getSwiper()
			this.getMenu()
		},
		methods: {
			check_verson(){
				return new Promise((resolve, reject) => {
					plus.runtime.getProperty(plus.runtime.appid, function(widgetInfo) {
						console.log(plus.runtime.appid);
						console.log(plus.runtime.version)
						uni.showLoading({
							title:'检测中'
						})
						uniCloud.callFunction({
							name: 'check-version',
							data: {
								appid: plus.runtime.appid,
								appVersion: plus.runtime.version,
								wgtVersion: widgetInfo.version
							},
							success: (e) => {
								console.log(e.result)
								uni.hideLoading()
								uni.showModal({
									title:e.result.title+" 版本号："+e.result.version,
									content:e.result.contents,
									success:res=>{
										uni.showToast({
											title: res.confirm+"",
											duration:10000
										})
										if(res.confirm){
											upAPP()
										}
									}
								})
								resolve(e)
							},
							fail: (error) => {
								uni.hideLoading()
								reject(error)
							}
						})
					})
				})
			},
			async getSwiper() {
				let res = await uniCloud.database().collection("opendb-banner").where({
					category_id: "index_swiper",
					status: true
				}).get()
				let data = res.result.data;
				data.forEach((item, index) => {
					data[index].image = item.bannerfile.path
				})
				this.list = data;
			},
			async getMenu() {
				let res = await uniCloud.database().collection("opendb-banner").where({
					category_id: "grid_menu",
					status: true
				}).get()
				console.log(res.result.data)
				this.menus = res.result.data
			},
			goPage(url) {
				console.log(url)
				uni.navigateTo({
					url
				})
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
