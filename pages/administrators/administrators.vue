<template>
	<view>
		<view class="colonn center_center">
			<view class="h-50"></view>
			<input placeholder="请输入密码" placeholder-class="text_align1_r fs-30"
			 class="text_align1_r fs-30"
			 v-model="inputValue" />
			<view class="h-50"></view>
			<block v-if="inputValue=='ttec902'">
				<view class="xiayibubtn" @longtap="clear">清空数据库数据</view>
			</block>
		</view>
	</view>
</template>

<script>
	import DB from '@/common/sqlite.js'
	export default {
		data() {
			return {
				inputValue:""
			}
		},
		methods: {
			
			// 提示框
			showToast: function(str) {
				uni.showToast({
					icon: "none",
					title: str,
					mask: true
				});
			},
			clear(){
				let open = DB.isOpen();
				if (open) {
					// 删除表 DB.deleteTableData(表名,查询条件列名,查询条件列值)
					DB.deleteTableData("scan_info")
						.then(res => {
							// this.$tools.showToast("删除表数据成功");
						})          
						.catch(error => {
							console.log("删除失败", error);
						});
						DB.deleteTableData("statistics_num")
							.then(res => {
								// this.$tools.showToast("删除表数据成功");
							})  
							.catch(error => {
								console.log("删除失败", error);
							});
						setTimeout(res=>{
							plus.runtime.restart();

						},1000)
				} else {
					this.$tools.showToast("数据库未打开");
				}
			}
		}
	}
</script>

<style>
@import url(administrators.css);
</style>
