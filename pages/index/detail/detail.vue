<template>
	<view>
		<page-head v-if="!data" :title="noData"></page-head>
		<view v-if="data" class="cu-bar bg-white solid-bottom">
			<view class="action" style="font-size: 20px;">
				{{domain}}
				<text class="cuIcon-titles text-grey " style="font-size: 12px;">{{'更新时间:'+data.queryTime}}</text>
			</view>
		</view>
		<view v-if="data" class="cu-list menu solid-top">
			<view class="cu-item">
				<view class="content">
					<text class="text-grey">所有者</text>
				</view>
				<view class="action">
					<text class="text-grey">{{data.registrantName}}</text>
				</view>
			</view>
			<view class="cu-item">
				<view class="content">
					<text class="text-grey">所有者联系邮箱</text>
				</view>
				<view class="action">
					<text class="text-grey">{{data.registrantEmail}}</text>
				</view>
			</view>
			<view class="cu-item">
				<view class="content">
					<text class="text-grey">注册商</text>
				</view>
				<view class="action">
					<text class="text-grey">{{data.sponsoringRegistrar}}</text>
				</view>
			</view>
			<view class="cu-item">
				<view class="content">
					<text class="text-grey">注册日期</text>
				</view>
				<view class="action">
					<text class="text-grey">{{data.registrationDate}}</text>
				</view>
			</view>
			<view class="cu-item">
				<view class="content">
					<text class="text-grey">到期日期</text>
				</view>
				<view class="action">
					<text class="text-grey">{{data.expirationDate}}</text>
				</view>
			</view>
			<view class="cu-item">
				<view class="content">
					<text class="text-grey">域名状态</text>
				</view>
				<view class="action">
					<text class="text-grey">{{data.domainStatus[0]}}</text>
				</view>
			</view>
			<view class="cu-item">
				<view class="content">
					<text class="text-grey">DNS服务器</text>
				</view>
				<view class="action">
					<text class="text-grey">{{data.nameServer[0]}}</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	const searchWhoIsUrl = 'https://cloud.baidu.com/api/bcd/whois/detail'
	const duration = 10000
	export default {
		data() {
			return {
				loading: false,
				domain: '',
				noData: '',
				data: {}
			}
		},
		onLoad(e) {
			if (e.data) {
				this.domain = e.data
				this.loadWhoIs()
			} else{
				uni.showToast({
					title: "啊呀，出错了",
					duration: 2000
				})
			}
		},
		onPullDownRefresh() {
			this.loadWhoIs();
		},
		methods: {
			loadWhoIs() {
				uni.showToast({
					title:'查询中',
					icon:'loading'
				})
				uni.request({
					method: 'POST',
					url: searchWhoIsUrl,
					data: {
						"domain": this.domain,
						"type": "NORMAL"
					}
				}).then(res => {
					const success = res[1].data.success
					const data = res[1].data.result.data
					if (success == true && data) {
						this.data = data
					} else{
						this.data = null
						this.noData = res[1].data.result.message
					}
					uni.hideToast()
					uni.stopPullDownRefresh()
					this.loading = false
				}).catch(err => {
					uni.hideToast()
					uni.stopPullDownRefresh()
					this.data = null
					this.loading = false;
				});
			}
		}
	}
</script>

<style>

</style>
