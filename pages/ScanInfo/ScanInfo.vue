<template>
	<view>
		<view style="width: 750rpx;height: 100vh;">
			<view>请扫码获取此次展会</view>
		</view>
	</view>
</template>

<script>
	var barcodeModel = uni.requireNativePlugin("iData-BarcodePlugin-BarcodeModule")
	var globalEvent = uni.requireNativePlugin('globalEvent');
	export default {
		data() {
			return {
				
			}
		},
		onLoad() {
			this.initScan();
		},
		methods: {
			// 初始化扫描
			initScan() {
				// 初始化扫描
				barcodeModel.initScan((ret) => {
					this.$tools.showToast(ret);
				});
			
				//页面监听event事件,建议在页面onLoad方法里调用
				var globalEvent = uni.requireNativePlugin('globalEvent');
				globalEvent.addEventListener('iDataBarcodeEvent', (e) => {
					console.log("扫码登录结果",e)
				});
				
			},
		}
	}
</script>

<style>

</style>
