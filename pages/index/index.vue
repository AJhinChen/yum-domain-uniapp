<template>
	<view>
		<view style="min-height: calc(100vh - 70px);">
			<page-head v-if="recommend==null||recommend.length==0" :title="noData"></page-head>
			<view class="cu-list menu solid-top" style="margin: 0;" v-for="(row,index) in common" :key="index">
				<view class="cu-item" :class="viewAnimation" :style="[{animationDelay: (index + 1)*0.1 + 's'}]" @tap="loadDomainStatus(row.domainName)">
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
						<text class="text-price text-red" style="font-size: 16px;">{{row.price}}</text>
					</view>
				</view>
			</view>
			<view class="cu-list menu solid-top" style="margin: 0;" v-for="(row,index) in recommend" :key="index">
				<view class="cu-item" :class="viewAnimation" :style="[{animationDelay: (index + common.length + 1)*0.1 + 's'}]" @tap="loadDomainStatus(row.domainName)">
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
						<text class="text-price text-red" style="font-size: 16px;">{{row.price}}</text>
					</view>
				</view>
			</view>
		</view>
		<!-- <view>
			<button class='cu-btn shadow' style="background-color: #1296db;color: #F1F1F1;" @tap="shake">换一批</button>
		</view> -->
		<uni-fab v-if="!searchShow" ref="fab" :pattern="pattern" :content="content" :horizontal="horizontal" :vertical="vertical" :direction="direction" @trigger="trigger" />
		<view v-else class="cu-form-group" style="height: 70px">
			<input type="text" style="height: 1.8rem;" focus="{{inputShow}}" confirm-type="search" placeholder="请输入域名名称" maxlength="15" @input="onKeyInput" @confirm="confirm"></input>
			<button class='cu-btn animation-reverse shadow' :class="animation" style="background-color: #1296db;color: #F1F1F1; height: 40px;" @tap="confirm">{{searchName}}</button>
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
				inputShow: false,
				loading: false,
				animation: '',
				viewAnimation: '',
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
					this.domain = ''
					this.inputShow = true
				} else{
					this.domain = ''
					this.initRecommendData()
				}
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
					this.searchName = '搜  索'
					this.label = this.domain.split('.')[0]
					this.tld = this.domain.split('.')[1]?this.domain.split('.')[1]:''
				}
			},
			shake() {
				this.initRecommendData()
			},
			confirm(e) {
				this.inputShow = false
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
					// 使页面滚动到顶部
					wx.pageScrollTo({
					   scrollTop: 0
					})
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
				uni.showLoading({
					title:'查询中'
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
						title: this.domain.length>0?this.domain:'推荐域名'
					})
					const success = res[1].data.success
					const accurate = res[1].data.result.accurate
					const common = res[1].data.result.common
					const recommend = res[1].data.result.recommend
					if (success == true) {
						this.common = accurate.length > 0 ? accurate.concat(common) : common
						this.recommend = recommend
						this.searchShow = false
						this.viewAnimation = 'animation-slide-bottom'
						setTimeout(()=>{
							this.viewAnimation = ''
						}, 1000)
					} else{
						this.common = null
						this.recommend = null
					}
					this.loading = false
					uni.hideLoading()
					uni.stopPullDownRefresh()
				}).catch(err => {
					uni.hideLoading()
					uni.stopPullDownRefresh()
					this.common = null
					this.recommend = null
					this.loading = false;
				});
			},
			initRecommendData() {
				this.animation = 'animation-shake'
				uni.setNavigationBarTitle({
					title: "推荐域名"
				})
				const arr = ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z']
				//随机生成3-5位数的.com域名
				const domainLength = Math.floor(Math.random() * 3) + 3
				this.label = ''
				for (var i = 0; i < domainLength; i++) {
					this.label += arr[Math.floor(Math.random() * 26)]
				}
				this.tld = '.com'
				
				uni.showLoading({
					title:'查询中'
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
					const success = res[1].data.success
					const common = res[1].data.result.common
					const recommend = res[1].data.result.recommend
					if (success == true) {
						this.common = common
						const number = (wx.getSystemInfoSync().windowHeight - 50 - 64)/80 - 2
						this.recommend = recommend.slice(0,number)
						this.viewAnimation = 'animation-slide-bottom'
						setTimeout(()=>{
							this.viewAnimation = ''
						}, 1000)
					} else{
						this.common = null
						this.recommend = null
					}
					this.loading = false
					this.animation = ''
					uni.hideLoading()
					uni.stopPullDownRefresh()
				}).catch(err => {
					this.common = null
					this.recommend = null
					this.loading = false;
					this.animation = ''
					uni.hideLoading()
					uni.stopPullDownRefresh()
				});
			},
			loadDomainStatus(domain) {
				uni.showLoading({
					title: '查询中'
				});
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
					uni.hideLoading()
				}).catch(err => {
					uni.hideLoading()
					this.loading = false;
					uni.showToast({
						title: "啊呀，出错了",
						icon:'none',
					})
				});
			}
		}
	}
</script>

<style>
	@import "../../colorui/animation.css";
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
