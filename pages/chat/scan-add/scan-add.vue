<template>
	<view class="page">
		<!-- 导航栏 -->
		<free-nav-bar title="加入群聊" showBack :showRight="false"></free-nav-bar>
		<view class="p-5">
			<view class="bg-white rounded p-4">
				<view class="flex align-center">
					<free-avater :src="detail.avatar || '/static/images/demo/demo6.jpg'"></free-avater>
					<view class="pl-4 flex flex-column">
						<text class="font-md">{{detail.name}}</text>
						<text class="font-sm text-muted">群人数：{{detail.user_count}}</text>
					</view>
				</view>
			</view>
			<view class="mt-3">
				<view class="py-3 flex align-center justify-center main-bg-color rounded" hover-class="main-bg-hover-color" @click="addGroup">
					<text class="font-md text-white">加入群聊</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import freeNavBar from "@/components/free-ui/free-nav-bar.vue"
	import freeAvater from '@/components/free-ui/free-avater.vue';
	import { mapState } from 'vuex'
	import $C from '@/common/free-lib/config.js';
	import $H from '@/common/free-lib/request.js';
	export default {
		components: {
			freeAvater,
			freeNavBar,
		},
		data() {
			return {
				detail:{
					id:0,
					name:"",
					avatar:"",
					user_count:0
				},
			}
		},
		onLoad(e) {
			if(e.params){
				this.detail = JSON.parse(decodeURIComponent(e.params))
			}
		},
		computed: {
			...mapState({
				user:state=>state.user.user
			})
		},
		methods: {
			addGroup(){
				uni.showLoading();
				$H.post('/group/join',{
					id:parseInt(this.detail.id)
				}).then(res=>{
					uni.hideLoading()
					uni.navigateBack({
						delta: 1
					});
				}).catch(err=>{
					uni.hideLoading()
				})
			}
		}
	}
</script>

<style>

</style>
