<template>
	<view>
		<view style="min-height: calc(100vh - 50px);">
			<page-head v-if="recommend==null||recommend.length==0" :title="noData"></page-head>
			<view class="cu-list menu solid-top" style="margin: 0;" v-for="(row,index) in common" :key="index">
				<view class="cu-item" @tap="loadDomainStatus(row.domainName)">
					<view class="content padding-tb-sm">
						<view style="font-size: 35rpx;">
							<text class="cuIcon-tagfill text-red margin-right-xs"></text>
							{{row.domainName}}
							<view class="cu-tag bg-orange light" style="margin-left: 10px;">{{row.displayTag}}</view>
						</view>
						<view class="text-gray text-sm">{{row.description}}</view>
					</view>
					<view class="action">
						<!-- 原价:
						<text class="text-price text-grey">{{row.originPrice + '				'}}</text>
						现价: -->
						百度云售价:
						<text class="text-price text-red">{{row.price}}</text>
					</view>
				</view>
			</view>
			<view class="cu-list menu solid-top" style="margin: 0;" v-for="(row,index) in recommend" :key="index">
				<view class="cu-item" @tap="loadDomainStatus(row.domainName)">
					<view class="content padding-tb-sm">
						<view style="font-size: 35rpx;">
							<text class="cuIcon-tagfill text-red margin-right-xs"></text>
							{{row.domainName}}
							<view class="cu-tag bg-orange light" style="margin-left: 10px;">{{row.displayTag}}</view>
						</view>
						<view class="text-gray text-sm">{{row.description}}</view>
					</view>
					<view class="action">
						<!-- 原价:
						<text class="text-price text-grey">{{row.originPrice + '				'}}</text>
						现价: -->
						百度云售价:
						<text class="text-price text-red" style="font-size: 20px;">{{row.price}}</text>
					</view>
				</view>
			</view>
		</view>
		<!-- <view>
			<button class='cu-btn shadow' style="background-color: #1296db;color: #F1F1F1;" @tap="shake">换一批</button>
		</view> -->
		<uni-fab v-if="!searchShow" ref="fab" :pattern="pattern" :content="content" :horizontal="horizontal" :vertical="vertical" :direction="direction" @trigger="trigger" />
		<view v-else class="cu-form-group">
			<input ref="search" confirm-type="search" placeholder="请输入域名名称" maxlength="15" @input="onKeyInput" @confirm="confirm"></input>
			<button class='cu-btn shadow' style="background-color: #1296db;color: #F1F1F1;" @tap="confirm">{{searchName}}</button>
		</view>

		<view class="cu-modal" :class="modalName=='Image'?'show':''">
			<view class="cu-dialog">
				<view>
					
				</view>
				<view class="cu-bar bg-red">
					<view class="action margin-0 flex-sub  solid-left" @tap="hideModalAndPush">可恶，盘他！</view>
				</view>
				<view class="cu-bar bg-white">
					<view class="action margin-0 flex-sub  solid-left" @tap="hideModal">算了，伤心。</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import uniFab from '@/components/uni-fab/uni-fab.vue'
	const searchInfoUrl = 'https://cloud.baidu.com/api/bcd/search/info'
	const searchStatusUrl = 'https://cloud.baidu.com/api/bcd/search/status'
	const duration = 10000
	export default {
		components: {
			uniFab
		},
		data() {
			return {
				title: '云域名',
				domain: '',
				searchName: '换一批',
				directionStr: '垂直',
				horizontal: 'right',
				vertical: 'bottom',
				direction: 'horizontal',
				pattern: {
					color: '#7A7E83',
					backgroundColor: '#fff',
					selectedColor: '#1296db',
					buttonColor: '#1296db'
				},
				content: [{
						iconPath: '/static/search.png',
						selectedIconPath: '/static/searchL.png',
						text: '搜索',
						active: false
					},
					{
						iconPath: '/static/recommend.png',
						selectedIconPath: '/static/recommendL.png',
						text: '推荐',
						active: false
					}
				],
				searchShow: true,
				loading: false,
				common: [],
				recommend: [],
				noData: '查无数据',
				label: '',
				tld: '',
				max: 0
			}
		},
		onLoad() {
			this.initRecommendData();
		},
		onPullDownRefresh() {
			this.initData();
		},
		methods: {
			trigger(e) {
				console.log(e)
				// this.content[e.index].active = !e.item.active
				this.searchShow = true
				this.searchName = '换一批'
				if (e.index == 0) {
					 // 使页面滚动到底部
					 wx.pageScrollTo({
					   scrollTop: 10000
					 })
					// this.$nextTick(() => {
					// 	this.$refs.search.focus()
					// })
				} else{
					this.domain = ''
					this.initRecommendData()
				}
			},
			pageScrollToBottom: function () {
			   wx.createSelectorQuery().select('#j_page').boundingClientRect(function (rect) {
				 // 使页面滚动到底部
				 wx.pageScrollTo({
				   scrollTop: rect.bottom
				 })
			   }).exec()
			},
			hideModalAndPush() {
				
			},
			hideModal() {
				
			},
			onKeyInput: function(event) {
				console.log(event.target.value)
				this.domain = event.target.value
				if (this.domain == '') {
					this.searchName = '换一批'
					this.initRecommendData()
				} else{
					this.searchName = '搜索'
					this.label = this.domain.split('.')[0]
					this.tld = this.domain.split('.')[1]?this.domain.split('.')[1]:''
				}
			},
			shake() {
				this.initRecommendData()
			},
			confirm() {
				// let regex = /^(?=^.{3,255}$)[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+$/
				if (this.domain == '') {
					this.initRecommendData()
				} else if (this.tld.length < 2 && this.tld != '') {
					uni.showToast({
						title:'域名格式错误',
						icon:'none',
						duration: 2000
					})
				} else if (this.domain.split('.').length == 2 || this.tld == '') {
					this.initData()
				} else{
					uni.showToast({
						title:'域名格式错误',
						icon:'none',
						duration: 2000
					})
				}
			},
			initData() {
				uni.showToast({
					title:'查询中',
					icon:'loading'
				})
				uni.request({
					method: 'POST',
					url: searchInfoUrl,
					data: {
						"domainNames": [],
						"labels": [this.label],
						"others": false,
						"tlds": [this.tld]
					}
				}).then(res => {
					uni.setNavigationBarTitle({
						title: "云域名:"+this.domain
					})
					uni.hideToast()
					uni.stopPullDownRefresh()
					const success = res[1].data.success
					const common = res[1].data.result.common
					const recommend = res[1].data.result.recommend
					if (success == true) {
						this.common = common
						this.recommend = recommend
						this.searchShow = false
					} else{
						this.common = null
						this.recommend = null
					}
					this.loading = false
				}).catch(err => {
					uni.hideToast()
					uni.stopPullDownRefresh()
					this.common = null
					this.recommend = null
					this.loading = false;
				});
			},
			initRecommendData() {
				uni.setNavigationBarTitle({
					title: "推荐域名"
				})
				const arr = ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z']
				this.label = arr[Math.floor(Math.random() * 26)] + arr[Math.floor(Math.random() * 26)] + arr[Math.floor(Math.random() * 26)] + arr[Math.floor(Math.random() * 26)] + arr[Math.floor(Math.random() * 26)]
				this.tld = '.com'
				
				uni.showToast({
					title:'查询中',
					icon:'loading'
				})
				uni.request({
					method: 'POST',
					url: searchInfoUrl,
					data: {
						"domainNames": [],
						"labels": [this.label],
						"others": false,
						"tlds": [this.tld]
					}
				}).then(res => {
					uni.hideToast()
					uni.stopPullDownRefresh()
					const success = res[1].data.success
					const common = res[1].data.result.common
					const recommend = res[1].data.result.recommend
					if (success == true) {
						this.common = common
						const number = (wx.getSystemInfoSync().windowHeight - 50 - 64)/80 - 2
						this.recommend = recommend.slice(0,number)
					} else{
						this.common = null
						this.recommend = null
					}
					this.loading = false
				}).catch(err => {
					uni.hideToast()
					uni.stopPullDownRefresh()
					this.common = null
					this.recommend = null
					this.loading = false;
				});
			},
			loadDomainStatus(domain) {
				uni.showToast({
					title: "查询中",
					icon: "loading"
				})
				const label = domain.split('.')[0]
				const tld = domain.split('.')[1]
				uni.request({
					method: 'POST',
					url: searchStatusUrl,
					data: {
						"domainNames": [{"label": label, "tld": tld}]
					}
				}).then(res => {
					const success = res[1].data.success
					const status = res[1].data.result.accurate[0].status
					const domainName = res[1].data.result.accurate[0].domainName
					uni.hideToast()
					if (success == true && status == 'UNREGISTERED') {
						uni.showModal({
							content: "恭喜您，域名未注册！请及时到百度云等交易所购买",
							confirmText: "好的",
							showCancel: false
						})
					} else{
						uni.showModal({
							content: "很遗憾，域名已经被注册，点击查看whois信息",
							confirmText: "查看",
							cancelText: "取消",
							success: function (res) {
								if (res.confirm) {
									uni.navigateTo({
										url: 'detail/detail?data='+domainName
									})
								}
							}
						})
					}
					this.loading = false
				}).catch(err => {
					uni.showToast({
						title: "啊呀，出错了",
						icon:'none',
						duration: 2000
					})
					this.loading = false;
				});
			}
		}
	}
</script>

<style>
	.logo {
		height: 200upx;
		width: 200upx;
		margin-top: 200upx;
	}

	.title {
		font-size: 36upx;
		color: #8f8f94;
	}
</style>
