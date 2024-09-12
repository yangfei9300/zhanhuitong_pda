
<script>
	export default {             
		globalData:{
			globalEvent:uni.requireNativePlugin('globalEvent'),
			barcodeModel : uni.requireNativePlugin("iData-BarcodePlugin-BarcodeModule")
		},      
		onLaunch: function() {   
			console.log("globalEvent",this.globalData.globalEvent)
			// var barcodeModel = uni.requireNativePlugin("iData-BarcodePlugin-BarcodeModule")
			// var globalEvent = uni.requireNativePlugin('globalEvent');
			// uni.removeStorageSync('exhInfo');  
			// 初始化扫描 
			this.globalData.barcodeModel.initScan((ret) => {
				console.log("初始化结果",ret)  
				this.$tools.showToast(ret);
			});          
			//页面监听event事件,建议在页面onLoad方法里调用
			this.globalData.globalEvent.addEventListener('iDataBarcodeEvent', (e) => {
				uni.$emit('scan',{'data':e})
			});      
		},          
		onShow: function() {
			console.log('App Show');
			
			uni.onNetworkStatusChange(function (res) {
				console.log("监听网络变化",res.isConnected);
				console.log("监听网络变化",res.networkType);
			});
			uni.getNetworkType(function (res) {
				console.log("获取网络变化",res.isConnected);
				console.log("获取网络变化",res.networkType);
			});
			
		}, 
		onHide: function() {  
			console.log('App Hide');
		},
		onUnload: function()  {
			console.log("退出软件")
		},
		methods:{
			//AES加密
			aesEncrypt(encrypt) {
				//加密值
				var e = this.AES.encrypt(encrypt, '1234567891234567', '1234567891234567')
				// console.error("加密结果",e);
				return e;
			},
			//AES解密
			aesDecrypt(decrypt) {
				// console.error("解密值",decrypt);
				//解密值
				var d = this.AES.decrypt(decrypt, '1234567891234567', '1234567891234567')
				// console.error("解密结果",d);
				return d;
			},
		}
	}  
</script>  

<style  lang="scss">
	/*每个页面公共css */
	@import url("/style/style.css");
	@import url("/style/style_rpx.css");
	@import "/style/ty_style_scss.scss";
</style>
