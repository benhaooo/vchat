<template>
	<view style="background-color: #EDEDED;">
		<!-- 导航栏 -->
		<free-nav-bar title="聊天信息" showBack :showRight="false"></free-nav-bar>
		<view class="flex flex-wrap py-3 bg-white">
			
			<view v-if="detail.chat_type === 'user'" class="flex flex-column align-center justify-center mb-2" style="width: 150rpx;">
				<free-avater :src="detail.avatar || '/static/images/userpic.jps'" size="110"></free-avater>
				<text class="font text-muted mt-1">{{detail.name}}</text>
			</view>
			
			<view v-else class="flex flex-column align-center justify-center mb-2" style="width: 150rpx;" v-for="(item,index) in list" :key="index">
				<free-avater :src="item.avatar || '/static/images/userpic.jpg'" size="110"></free-avater>
				<text class="font text-muted mt-1">{{item.name}}</text>
			</view>
			
			<view class="flex flex-column align-center justify-center mb-2" style="width: 150rpx;">
				<view class="flex align-center justify-center border" 
				style="width: 120rpx;height: 120rpx;" 
				hover-class="bg-light"
				@click="openMail">
					<text class="text-light-muted" 
					style="font-size: 100rpx;">+</text>
				</view>
			</view>
			
			<view class="flex flex-column align-center justify-center mb-2" style="width: 150rpx;">
				<view class="flex align-center justify-center border" 
				style="width: 120rpx;height: 120rpx;" 
				hover-class="bg-light"
				@click="deleteUser">
					<text class="text-light-muted" 
					style="font-size: 100rpx;">-</text>
				</view>
			</view>
			
		</view>
		<view v-if="detail.chat_type === 'group'">
			<free-divider></free-divider>
			<free-list-item title="群聊名称" showRight :showLeftIcon="false"
			@click="updateName">
				<text slot="right" class="font text-muted">{{detail.name}}</text>
			</free-list-item>
			<free-list-item title="群二维码" showRight :showLeftIcon="false"
			@click="openCode">
				<text slot="right" class="iconfont font-md text-light-muted">&#xe647;</text>
			</free-list-item>
			<free-list-item title="群公告" showRight :showLeftIcon="false" @click="openGroupRemark"></free-list-item>
		</view>
		<free-divider></free-divider>
		<free-list-item title="查找聊天记录" showRight :showLeftIcon="false" @click="openHistory"></free-list-item>
		<free-divider></free-divider>
		<free-list-item title="消息免打扰" showRight :showLeftIcon="false"
		:showRightIcon="false">
			<switch slot="right" :checked="detail.nowarn" color="#08C060" @change="updateChatItem($event,'nowarn')"/>
		</free-list-item>
		<free-list-item title="置顶聊天" showRight :showLeftIcon="false"
		:showRightIcon="false">
			<switch slot="right" :checked="detail.istop" color="#08C060"
			@change="updateChatItem($event,'istop')"/>
		</free-list-item>
		<free-list-item title="强提醒" showRight :showLeftIcon="false"
		:showRightIcon="false">
			<switch slot="right" :checked="detail.strongwarn" color="#08C060" @change="updateChatItem($event,'strongwarn')"/>
		</free-list-item>
		
		<view v-if="detail.chat_type === 'group'">
			<free-divider></free-divider>
			<free-list-item title="我在本群的昵称" showRight :showLeftIcon="false" @click="updateNickname">
				<text slot="right" class="font text-muted">{{nickname}}</text>
			</free-list-item>
			<free-list-item title="显示群成员昵称" showRight 
			:showLeftIcon="false"
			:showRightIcon="false">
				<switch slot="right" :checked="detail.shownickname" color="#08C060" @change="updateChatItem($event,'shownickname')"/>
			</free-list-item>
		</view>
		
		<free-divider></free-divider>
		<free-list-item title="清空聊天记录" showRight :showLeftIcon="false" @click="clear"></free-list-item>
		<free-divider></free-divider>
		<free-list-item title="投诉" showRight :showLeftIcon="false" @click="openReport"></free-list-item>
		<free-divider></free-divider>
		<view v-if="detail.chat_type === 'group'" class="py-3 flex align-center justify-center bg-white"
		hover-class="bg-light" @click="quit">
			<text class="font-md text-danger">删除并退出</text>
		</view>
		
		<free-confirm ref="confirm" :title="'修改'+confirmTitle">
			<input type="text" v-model="confirmText" class="border-bottom font-md" :placeholder="confirmTitle"/>
		</free-confirm>
		
		<view style="height: 200rpx;"></view>
	</view>
</template>

<script>
	import freeNavBar from "@/components/free-ui/free-nav-bar.vue"
	import freeAvater from '@/components/free-ui/free-avater.vue';
	import freeDivider from "@/components/free-ui/free-divider.vue"
	import freeListItem from "@/components/free-ui/free-list-item.vue"
	import { mapState } from 'vuex'
	import $H from '@/common/free-lib/request.js';
	import freeConfirm from '@/components/free-ui/free-confirm.vue';
	import auth from '@/common/mixin/auth.js';
	export default {
		mixins:[auth],
		components: {
			freeNavBar,
			freeAvater,
			freeDivider,
			freeListItem,
			freeConfirm
		},
		data() {
			return {
				confirmType:"name",
				confirmText:"",
				list:[],
				nickname:"", // 我在本群的昵称
				detail:{
					id:0, // 接收人/群 id
					chat_type:'user', // 接收类型 user单聊 group群聊
					avatar:'', // 接收人/群 头像
					name:'', // 接收人/群 昵称

					istop:false, // 是否置顶
					shownickname:false, // 是否显示昵称
					nowarn:false, // 消息免打扰
					strongwarn:false, // 是否开启强提醒
					
					user_id:0, // 群管理员id
					remark:"", // 群公告
					invite_confirm:0, // 邀请确认
				}
			}
		},
		onLoad(e) {
			if(!e.params){
				return this.backToast()
			}
			let detail = JSON.parse(e.params)
			// 获取当前会话详细资料
			detail = this.chat.getChatListItem(detail.id,detail.chat_type)
			if(!detail){
				return this.backToast()
			}
			this.detail = detail
		},
		onShow() {
			if(this.detail.chat_type === 'group'){
				$H.get('/group_info/'+this.detail.id).then(res=>{
					console.log(res);
					this.list = res.group_users.map(item=>{
						if(item.user_id === this.user.id){
							this.nickname = item.nickname
						}
						return {
							id:item.user_id,
							name:item.nickname || item.user.nickname || item.user.username,
							avatar:item.user.avatar
						}
					})
					this.detail.remark = res.remark
				})
			}
		},
		computed: {
			...mapState({
				chat:state=>state.user.chat,
				user:state=>state.user.user
			}),
			confirmTitle(){
				return this.confirmType === 'name' ? '群名称' : '昵称'
			}
		},
		methods: {
			updateChatItem(e,k){
				this.detail[k] = e.detail.value
				this.chat.updateChatItem({
					id:this.detail.id,
					chat_type:this.detail.chat_type
				},this.detail)
			},
			openMail(){
				let params = this.detail.chat_type === 'user' ? '?type=createGroup' : '?type=inviteGroup&id='+this.detail.id
				uni.navigateTo({
					url: '../../mail/mail/mail' + params,
				});
			},
			updateName(){
				this.confirmType = 'name'
				this.confirmText = this.detail.name
				this.$refs.confirm.show((close)=>{
					if(this.confirmText == ''){
						return uni.showToast({
							title: '群名称不能为空',
							icon: 'none'
						});
					}
					$H.post('/group/rename',{
						id:this.detail.id,
						name:this.confirmText
					}).then(res=>{
						uni.showToast({
							title: '修改成功',
							icon: 'none'
						});
						this.detail.name = this.confirmText
						close()
					})
				})
			},
			updateNickname(){
				this.confirmType = 'nickname'
				this.confirmText = this.nickname
				this.$refs.confirm.show((close)=>{
					$H.post('/group/nickname',{
						id:this.detail.id,
						nickname:this.confirmText
					}).then(res=>{
						this.nickname = this.confirmText
						uni.showToast({
							title: '修改成功',
							icon: 'none'
						});
						close()
					})
				})
			},
			openGroupRemark(){
				uni.navigateTo({
					url: '../group-remark/group-remark?params='+encodeURIComponent(JSON.stringify({
						id:this.detail.id,
						remark:this.detail.remark
					})),
				});
			},
			openReport(){
				uni.navigateTo({
					url: "/pages/mail/user-report/user-report?params="+JSON.stringify({
						user_id:this.detail.id,
						type:this.detail.chat_type
					})
				});
			},
			quit(){
				uni.showModal({
					content: '是否要删除或退出该群聊？',
					success:(res)=>{
						if (res.cancel) return
						$H.post('/group/quit',{
							id:this.detail.id
						}).then(res=>{
							uni.showToast({
								title: '操作成功',
								icon: 'none'
							});
							uni.navigateBack({
								delta: 1
							});
						})
					}
				});
			},
			openCode(){
				uni.navigateTo({
					url: '../../my/code/code?params='+encodeURIComponent(JSON.stringify({
						id:this.detail.id,
						name:this.detail.name,
						avatar:this.detail.avatar
					}))+'&type=group',
				});
			},
			clear(){
				uni.showModal({
					content: '是否要清空聊天记录？',
					success: (res)=> {
						if (res.confirm) {
							this.chat.clearChatDetail(this.detail.id,this.detail.chat_type)
							uni.showToast({
								title: '清空成功',
								icon: 'none'
							});
							uni.$emit('updateHistory')
						}
					}
				});
			},
			openHistory(){
				uni.navigateTo({
					url:`../chat-history/chat-history?chat_type=${this.detail.chat_type}&id=${this.detail.id}`,
				});
			},
			deleteUser(){
				uni.navigateTo({
					url: '../group-user/group-user?id='+this.detail.id,
				});
			}
		}
	}
</script>

<style>

</style>
