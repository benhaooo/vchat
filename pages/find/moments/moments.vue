<template>
	<view>
		<free-transparent-bar :scrollTop="scrollTop"
		@clickRight="clickRight"></free-transparent-bar>
		<view class="position-relative" style="height: 620rpx;">
			<image src="https://douyinzcmcss.oss-cn-shenzhen.aliyuncs.com/shengchengqi/datapic/1.jpg" mode="aspectFill" style="height: 590rpx;" class="bg-secondary w-100"></image>
			<image :src="avatar" style="width: 120rpx;height: 120rpx;right: 30rpx;" class="bg-secondary rounded position-absolute bottom-0"></image>
			<text class="text-white font-md position-absolute"
			style="bottom: 45rpx;right: 160rpx;">{{nickname}}</text>
		</view>
		
		<!-- 朋友圈列表 -->
		<free-moment-list v-for="(item,index) in list" :key="index"
		:item="item" :index="index" @action="doAction" @reply="replyEvent"></free-moment-list>
		
		<!-- 评论框 -->
		<free-popup ref="action" bottom transformOrigin="center bottom">
			<view style="height: 105rpx;" class="bg-light border-top flex align-center px-3">
				<textarea fixed class="bg-white rounded p-2 font-md" style="height: 80rpx;width: 480rpx;" :focus="true" v-model="content"/>
				<free-icon-button :icon="'\ue605'"
				@click="changeFaceModal"></free-icon-button>
				<free-main-button name="发送" :disabled="content.length === 0" @click="send"></free-main-button>
			</view>
			<scroll-view v-if="faceModal" scroll-y="true" 
			style="height: 350rpx;"
			class="bg-light">
				<view class="flex flex-wrap">
					<view style="width: 107rpx;height: 107rpx;"
					class="flex align-center justify-center"
					hover-class="bg-white"
					v-for="(item,index) in faceList"
					:key="index"
					@click="addFace(item)">
						<text>{{item}}</text>
					</view>
				</view>
			</scroll-view>
		</free-popup>
		
		<!-- 上拉加载 -->
		<view class="flex align-center justify-center py-5 bg-light"
		v-if="list.length >= 10">
			<text class="text-muted font">{{loadmore}}</text>
		</view>
		
		
	</view>
</template>

<script>
	import freeTransparentBar from '@/components/free-ui/free-transparent-bar.vue';
	import freeMomentList from '@/components/free-ui/free-moment-list.vue';
	import freePopup from '@/components/free-ui/free-popup.vue';
	import freeIconButton from "@/components/free-ui/free-icon-button.vue"
	import freeMainButton from '@/components/free-ui/free-main-button.vue';
	import { mapState } from 'vuex'
	import $H from '@/common/free-lib/request.js';
	export default {
		components: {
			freeTransparentBar,
			freeMomentList,
			freePopup,
			freeIconButton,
			freeMainButton
		},
		data() {
			return {
				content:"",
				scrollTop:0,
				list:[],
				
				faceModal:false,
				faceList:["😀","😁","😂","😃","😄","😅","😆","😉","😊","😋","😎","😍","😘","😗","😙","😚","😇","😐","😑","😶","😏","😣","😥","😮","😯","😪","😫","😴","😌","😛","😜","😝","😒","😓","😔","😕","😲","😷","😖","😞","😟","😤","😢","😭","😦","😧","😨","😬","😰","😱","😳","😵","😡","😠"],
				// 评论的对象
				commentIndex:-1,
				
				page:1,
				loadmore:"上拉加载更多",
				key:"moment_timeline",
				reply_user:false,
				
				params:false
			}
		},
		onPageScroll(e) {
			this.scrollTop = e.scrollTop
		},
		computed: {
			...mapState({
				user:state=>state.user.user
			}),
			nickname(){
				if(!this.params){
					return this.user.nickname || this.user.username
				}
				return this.params.name
			},
			avatar(){
				let avatar = ''
				avatar = !this.params ? this.user.avatar : this.params.avatar
				return avatar || '/static/images/userpic.jpg'
			}
		},
		onLoad(e) {
			if(e.key){
				this.key = e.key
			}
			if(e.params){
				this.params = JSON.parse(decodeURIComponent(e.params))
				console.log(this.params);
			}
			this.page = 1
			this.getData()
			
			uni.$on('momentNotice',this.momentNotice)
		},
		destroyed() {
			uni.$off('momentNotice',this.momentNotice)
		},
		onPullDownRefresh() {
			this.page = 1
			this.getData().then(res=>{
				uni.showToast({
					title: '刷新成功',
					icon: 'none'
				});
				uni.stopPullDownRefresh()
			}).catch(err=>{
				uni.showToast({
					title: '刷新失败',
					icon: 'none'
				});
				uni.stopPullDownRefresh()
			})
		},
		onReachBottom() {
			if(this.loadmore !== '上拉加载更多'){
				return
			}
			this.page += 1
			this.loadmore = '加载中...'
			this.getData().catch(err=>{
				this.page -= 1
				uni.showToast({
					title: '加载失败',
					icon: 'none'
				});
				this.loadmore = '上拉加载更多'
			})
		},
		methods: {
			momentNotice(){
				this.page = 1
				this.getData().then(res=>{
					uni.showToast({
						title: '刷新成功',
						icon: 'none'
					});
				})
			},
			getData(){
				return new Promise((result,reject)=>{
					let page = this.page
					let param = ''
					if(this.params && this.key == 'moment'){
						param = '?user_id='+this.params.id
					}
					$H.get(`/${this.key}/${this.page}${param}`).then(res=>{
						this.list = page === 1 ? res : [...this.list,...res]
						this.loadmore = this.list.length === (page * 10) ? '上拉加载更多' : '没有更多了'
						result(res)
					}).catch(err=>{
						reject(err)
					})
				})
			},
			// 点击操作菜单
			doAction(e){
				uni.showActionSheet({
					itemList: ['点赞','评论'],
					success: res => {
						if(res.tapIndex === 0){
							this.doSupport(e)
						} else {
							this.content = ''
							this.faceModal = false
							this.commentIndex = e.index
							this.reply_user = false
							this.$refs.action.show()
						}
					},
				});
			},
			// 点赞
			doSupport(e){
				console.log(e);
				$H.post('/moment/like',{
					id:e.item.moment_id
				}).then(res=>{
					let i = e.item.likes.findIndex(item=>item.id === this.user.id)
					if(i !== -1){ // 取消点赞
						e.item.likes.splice(i,1)
					} else { // 点赞
						e.item.likes.push({
							id:this.user.id,
							name:this.user.nickname || this.user.username
						})
					}
					uni.showToast({
						title: i !== -1 ? '取消点赞成功' : '点赞成功',
						icon: 'none'
					});
				})
			},
			// 添加表情
			addFace(item){
				this.content += item
			},
			// 开启/关闭表情包面板
			changeFaceModal(){
				uni.hideKeyboard()
				setTimeout(()=>{
					this.faceModal = !this.faceModal
				},100)
			},
			// 发送
			send(){
				let item = this.list[this.commentIndex]
				$H.post('/moment/comment',{
					id:item.moment_id,
					content:this.content,
					reply_id:this.reply_user ? this.reply_user.id : 0
				}).then(res=>{
					item.comments.push({
						content:this.content,
						user:{
							id:this.user.id,
							name:this.user.nickname || this.user.username
						},
						reply:this.reply ? this.reply : null
					})
					uni.showToast({
						title: '评论成功',
						icon: 'none'
					});
				})
				this.$refs.action.hide()
			},
			// 选择发表朋友圈类型
			clickRight(){
				let list = [{
					name:"图文",
					key:"image"
				},{
					name:"短视频",
					key:"video"
				},{
					name:"文字",
					key:"content"
				}]
				uni.showActionSheet({
					itemList: list.map(v=>v.name),
					success: res => {
						uni.navigateTo({
							url: '../add-moment/add-moment?type='+list[res.tapIndex].key,
						});
					},
				});
			},
			replyEvent(e){
				this.content = ''
				this.faceModal = false
				this.commentIndex = e.index
				this.reply_user = e.reply
				this.$refs.action.show()
			}
		}
	}
</script>

<style>

</style>
