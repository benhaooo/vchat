<template>
	<view class="page">
		<!-- 导航栏 -->
		<free-nav-bar showBack :showRight="false" :backEvent="false"
		@back="back">
			<input type="text" :placeholder="placeholder" v-model="keyword"
			style="width: 650rpx;" class="font-md" @confirm="confirm"/>
		</free-nav-bar>
		
		<view v-if="searchType == '' && list.length === 0">
			<view class="py-5 flex align-center justify-center">
				<text class="font text-light-muted">搜索指定内容</text>
			</view>
			
			<view class="px-4 flex flex-wrap">
				<view class="flex align-center justify-center mb-3 border-left border-right" style="width: 223rpx;" v-for="(item,index) in typeList" :key="index" @click="changeSearchType(item)">
					<text class="font text-hover-primary">{{item.name}}</text>
				</view>
			</view>
		</view>
		
		
		<free-list-item v-for="(item,index) in list" :key="index" 
		:title="item.nickname ? item.nickname : item.username"
		:cover="item.avatar ? item.avatar : '/static/images/userpic.jpg'"
		@click="openUserBase(item.id)"></free-list-item>
		
	</view>
</template>

<script>
	import freeNavBar from "@/components/free-ui/free-nav-bar.vue"
	import freeListItem from "@/components/free-ui/free-list-item.vue"
	import $H from "@/common/free-lib/request.js"
	export default {
		components: {
			freeNavBar,
			freeListItem
		},
		data() {
			return {
				typeList:[{
					name:"聊天记录",
					key:"history"
				},{
					name:"用户",
					key:"user"
				},{
					name:"群聊",
					key:"group"
				}],
				searchType:"",
				keyword:"",
				list:[]
			}
		},
		computed: {
			placeholder() {
				let obj = this.typeList.find((item)=>{
					return item.key === this.searchType
				})
				if(obj){
					return '搜索'+obj.name
				}
				return '请输入关键字'
			}
		},
		methods: {
			back(){
				if(this.searchType == ''){
					return uni.navigateBack({
						delta: 1
					});
				}
				this.searchType = ''
			},
			confirm(){
				$H.post('/search/user',{
					keyword:this.keyword
				}).then(res=>{
					this.list = []
					if(res){
						this.list.push(res)
					}
				})
			},
			// 打开用户资料
			openUserBase(user_id){
				uni.navigateTo({
					url: '../../mail/user-base/user-base?user_id='+user_id,
				});
			},
			changeSearchType(item){
				console.log(item);
				this.searchType = item.key
			}
		}
	}
</script>

<style>

</style>
