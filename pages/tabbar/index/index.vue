<template>
	<view>
		
		
		<!-- 导航栏 -->
		<free-nav-bar title="微信" :noreadnum="totalNoreadnum">
		</free-nav-bar>
		
		<view class="bg-danger p-3 left-0 right-0 flex align-center justify-between" v-if="!chat || !chat.isOnline">
			<text class="text-white font">当前处于离线状态</text>
			<view class="border py-1 px-2 rounded" hover-class="bg-hover-danger" @click="chat.reconnectConfirm()">
				<text class="text-white font">重新连接</text>
			</view>
		</view>
		
		<!-- 置顶列表 -->
		<block v-for="(item,index) in list" :key="index">
			<free-media-list v-if="item.istop" :item="item" :index="index"
			@long="long"></free-media-list>
		</block>
		
		<!-- 非置顶列表 -->
		<block v-for="(item,index) in list" :key="index">
			<free-media-list v-if="!item.istop" :item="item" :index="index"
			@long="long"></free-media-list>
		</block>
		
		
		<!-- 弹出层 -->
		<free-popup ref="extend" :bodyWidth="240" :bodyHeight="getMenusHeight">
			<view class="flex flex-column" 
			style="width: 240rpx;"
			:style="getMenusStyle">
				<view class="flex-1 flex align-center" 
				hover-class="bg-light"
				v-for="(item,index) in menus"
				:key="index"
				@click="clickEvent(item.event)">
					<text class="font-md pl-3">{{item.name}}</text>
				</view>
			</view>
		</free-popup>
		
		
	</view>
</template>

<script>
	import freeNavBar from "@/components/free-ui/free-nav-bar.vue"
	import freeMediaList from "@/components/free-ui/free-media-list.vue"
	
	import freePopup from "@/components/free-ui/free-popup.vue"
	import auth from '@/common/mixin/auth.js';
	import { mapState } from 'vuex'
	export default {
		mixins:[auth],
		components:{
			freeNavBar,
			freeMediaList,
			freePopup
		},
		data() {
			return {
				propIndex:-1,
				menus:[
					{
						name:"设为置顶",
						event:"setTop"
					},
					{
						name:"删除该聊天",
						event:"delChat"
					}
				],
			}
		},
		onLoad() {
			
		},
		computed:{
			...mapState({
				list:state=>state.user.chatList,
				totalNoreadnum:state=>state.user.totalNoreadnum,
				chat:state=>state.user.chat
			}),
			// 动态获取菜单高度
			getMenusHeight(){
				let H = 100
				return this.menus.length * H
			},
			// 获取菜单的样式
			getMenusStyle(){
				return `height: ${this.getMenusHeight}rpx;`
			}
		},
		methods: {
			long({x,y,index}){
				// 初始化 索引
				this.propIndex = index
				// 拿到当前对象
				let item = this.list[index]
				// 判断之前是否处于置顶状态
				this.menus[0].name = item.istop ? '取消置顶' : '设为置顶'
				
				this.$refs.extend.show(x,y)
			},
			// 分发菜单事件
			clickEvent(event){
				switch (event){
					case "setTop": // 置顶/取消置顶会话
					this.setTop()
						break;
					case "delChat": // 删除当前会话
					this.delChat()
						break;
				}
				this.$refs.extend.hide()
			},
			// 删除当前会话
			delChat(){
				// this.list.splice(this.propIndex,1)
				let item = this.list[this.propIndex]
				this.chat.removeChatItem(item.id,item.chat_type)
			},
			// 置顶/取消置顶会话
			setTop(){
				let item = this.list[this.propIndex]
				item.istop = !item.istop
			}
		}
	}
</script>

<style>
	
</style>
