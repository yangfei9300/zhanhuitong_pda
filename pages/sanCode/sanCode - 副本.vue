<template>
	<view class="colonn center_center"
	>
		<view class="roww w-710 border_bottom" style="padding:20rpx 20rpx;font-size: 30rpx;">
			<view>当前展会</view>
			<view class="allline"></view>
				
				<view  style="color: #000000;text-align: right;" class="w-500 txtShowLength">
					<block v-if="exhInfo">
						{{exhInfo[0]}}
					</block>
					<block v-else>
						请选择展会
					</block>
				</view>
		</view>
		<view class="roww w-710 border_bottom" style="padding:20rpx 20rpx;font-size: 30rpx;">
			<view>当前状态</view>
			<view class="allline"></view>
				<view  style="color: #000000;text-align: right;" class="w-500 txtShowLength">
					<block v-if="isLixian==2">
						离线模式
					</block>
					<block v-else>
						在线模式
					</block>
				</view>
		</view>
		<view class="roww w-710 border_bottom" style="padding:20rpx 20rpx;font-size: 30rpx;">
			<view>懒人模式（连续扫描）</view>
			<view class="allline"></view>
			<view  style="color: #000000;text-align: right;font-weight: bold;" class="w-300 "
				@click.stop="lianxuIsClick"
			>
					{{lianxuIs?'关闭':'开启'}}
			</view>
		</view>
		<view class="btncen center_center colonn" style="margin-top: 40%;"
		:style="{
			'background-color':bg
		}"
		>
			 <!-- <block v-if="lianxu"> -->
			 <!--  @click="continueScanFunc(false)" -->
				<view style="color: #ffffff;font-size: 40rpx;line-height: 30rpx;font-weight: bold;" 
				class="colonn center_center">
					<view>核销手机号</view>
					<view style="color: #ffffff;margin-top: 20rpx;margin-top: 45rpx;">{{scanResult}}</view>
				</view>
			<!-- </block> -->
			<!-- <block v-else> -->
				<!-- <view style="color: #ffffff;font-size: 40rpx;" @click="continueScanFunc(true)">开启连续扫描</view> -->
			<!-- </block> -->
			
		</view>
		
<!-- 		<view class="btncen1" style="margin-top: 40%;text-align: center;font-size: 35rpx;text-align: left;padding-left: 25rpx;">
			核销手机号：{{scanResult}}</view> -->
		
		<view class="roww center_center width poab" style="bottom: 40rpx;">
			<view class="w-50"></view>
			<view style="color:#000000;">
				<view style="color:#000000;">
					<block v-if="exhInfo">
						{{exhInfo[3]}}
					</block>
					<block v-else>
						请选择门
					</block>
				</view>
			</view>
			<view class="allline roww center_center">
				<view v-if="selResultCount">今天总人数：{{selResultCount.count}}</view>
				<view v-else>数量统计未开放</view>
				  
			</view>
			<view style="color:#2E7EFC;" @click.stop="toLogin">退出登录</view>
			<view class="w-50"></view>
		</view>

	</view>
</template>

<script>
	import DB from '@/common/sqlite.js'
	const CryptoJS = require('crypto-js') // 引用AES源码js
	const key = CryptoJS.enc.Utf8.parse('2020081720200817') // 十六位十六进制数作为密钥
	export default {
		data() {
			return {
				zhanhuiInfo: null,
				exhList: ['展会1', '展会2'],
				cishuNum: 0,
				lianxu: false,
				invTime: "1000",
				menType: null,
				menList: [],
				exhInfo:null,
				scanResult:"",//扫码结果
				isLixian:1,//2是离线  1在线
				
				selResultCount:null,//统计表的实体类
				lianxuIs:false,
				'bg':'green',
			}
		},
		onLoad() {
			var exhInfo=uni.getStorageSync("exhInfo")
			console.log("当前扫码信息",exhInfo);
			if(!exhInfo){
				this.$tools.showToast("请扫展会二维码进行登录");
				return false;
			}
			var isLixian=uni.getStorageSync("isLixian")
			if(isLixian){
				this.isLixian=isLixian;
			}
			this.exhInfo=exhInfo;
			this.openSQL();
		},

		onShow() { 
			uni.$off('scan')  //在此生命周期里销毁地址改变事件的监听
			 uni.$on('scan',(res)=>{
			        console.log("扫码界面监听结果",res) 
					console.log("扫码界面监听结果",res.data.barcode) 
					if(res.data.barcode.indexOf("https")>=0){
						var restxt=res.data.barcode.replace('https://frdzhtsignup.zsyflive.com/frd/','');
						// var toJaimiStr=getApp().aesEncrypt(txt);
						var jiemiResult=getApp().aesDecrypt(restxt);
						console.log("==",'https://frdzhtsignup.zsyflive.com?'+jiemiResult)
						var jsonUrl=this.getDate('https://frdzhtsignup.zsyflive.com?'+jiemiResult)
						console.log('jsonUrl',jsonUrl)
						if(jsonUrl.timestamp&&jsonUrl.upUid){
							var timeQr=jsonUrl.timestamp;
							var timeNew=Date.parse(new Date());
							var syTime=(timeNew-parseInt(timeQr))/1000;
							console.log("dang钱事假",syTime,timeNew,timeQr);
							if(syTime>1800){
								this.bg="red";
							}else{
								this.bg="green";
							}
							jiemiResult=jsonUrl.upUid
						}
					}else{
						var restxt=res.data.barcode;
						restxt=this.Decrypt(restxt)
						
						if(restxt.indexOf("-")>=0){
							
							var timeQr=restxt[0];
							var timeNew=Date.parse(new Date());
							var syTime=(timeNew-parseInt(timeQr))/1000;
							console.log("dang钱事假",syTime,timeNew,timeQr);
							if(syTime>1800){
								this.bg="red";
							}else{
								this.bg="green";
							}
							jiemiResult=restxt[1]
						}
					}
					
					
					
					var obg={
						'barcode':jiemiResult,
					}
					this.initScan(obg);
			  })
		},  
		onUnload() {
			// this.closeSQL();
			console.log("卸载")
			 getApp().globalData.barcodeModel.continueScan(false,(ret) => {
			 	this.lianxuIs=false;
			   
			 });  
		},

		methods: {
			
			// 解密方法
					Decrypt(word) {
						let decrypt = CryptoJS.AES.decrypt(word, key, {
							mode: CryptoJS.mode.ECB,
							padding: CryptoJS.pad.Pkcs7
						})
						let decryptedStr = decrypt.toString(CryptoJS.enc.Utf8)
						return decryptedStr.toString()
					},
			
			
			     getDate(url){
			        // str为？之后的参数部分字符串
			        const str = url.substr(url.indexOf('?') + 1)
			        // arr每个元素都是完整的参数键值
			        const arr = str.split('&')
			        // result为存储参数键值的集合
			        const result = {}
			        for (let i = 0; i < arr.length; i++) {
			            // item的两个元素分别为参数名和参数值
			            const item = arr[i].split('=')
			            result[item[0]] = item[1]
			        }
			        return result
			    },
			
			// 解密方法
			 Decrypt(word) {
			    let decrypt = CryptoJS.AES.decrypt(word, key, {
			        mode: CryptoJS.mode.ECB,
			        padding: CryptoJS.pad.Pkcs7
			    })
			    let decryptedStr = decrypt.toString(CryptoJS.enc.Utf8)
			    return decryptedStr.toString()
			},
			 
			// 加密方法
			 Encrypt(word) {
			    let encrypted = CryptoJS.AES.encrypt(word, key, { 
			        mode: CryptoJS.mode.ECB, 
			        padding: CryptoJS.pad.Pkcs7 
			    });
			    return encrypted.toString()
			},
			
			// 修改是否连续扫描硬件
			lianxuIsClick(){
				this.lianxuIs=!this.lianxuIs;
				getApp().globalData.barcodeModel.continueScan(this.lianxuIs,(ret) => {
					if(this.lianxuIs){
						 this.$tools.showToast("连续扫描开启成功");
					}else{
						this.$tools.showToast("连续扫描已关闭");
					}
				  
				});  
			},
			
			// 初始化扫描
			initScan(e) {  
				console.log("扫码结果",e);
                  	if (!this.exhInfo) {
						this.$tools.showToast("请先前往登录页面选择展会");
						return false;
					}
					var barcode = e.barcode;
					// if (barcode.indexOf("-") >= 0) {
					// 	//核销活动
					// 	this.hexiaoLuntan(barcode);
					// } else
					 
					 
					 if(this.$tools.isphone(barcode)){
						//核销展会预报名入场
						this.hexiao(barcode);
					}else{
						this.$tools.showToast("请勿扫描其他码-----"+barcode);
						this.scanResult=barcode;
					}
			},
			
			// 连续扫描开启关闭
			continueScanFunc(enable) {
				this.lianxu = enable;
				// 连续扫描设置：先设置扫描间隔时间，然后开启扫描开关
				if (enable) {
					//设置连续扫描间隔时间
					barcodeModel.intervalSet(this.invTime,
						(ret) => {
							this.$tools.showToast(ret);
						});
				}

				//设置连续扫描
				barcodeModel.continueScan(enable,
					(ret) => {

						this.$tools.showToast(ret);
					});
			},
			// 开启单次扫描
			scanStartFunc() {
				barcodeModel.scanStart((ret) => {

					this.$tools.showToast(ret);
				});
			},

			// 核销论坛入场
			hexiaoLuntan(data1) {
				
				data1 = data1.split("-");
				var data = {
					exhId: this.exhInfo[1],
					unionid: data1[0],
					userId: uni.getStorageSync("userInfo").userId,
					activityId: data1[1],
				};
				if(this.isLixian==2){//离线
					this.saveZhanhui(data1[0],data1[1],2,false);//1是展会2是论坛  falses是离线
					return
				}
				console.log("开始核销论坛",this.$paths.activitypda,data)
				this.$axios
					.axios('post', this.$paths.activitypda, data)
					.then(res => {
						console.log("论坛结果",res,res.code == 200)
						if (res.code == 200) {
							this.scanResult=data1[0];
							this.$tools.showToast(data1[0] + "论坛核销成功");
							
							this.saveZhanhui(data1[0],data1[1],2,true);//1是展会2是论坛  true是在线
						} else {
							this.$tools.showToast(res.msg);
							this.scanResult=res.msg;
						}
					})
					.catch(err => {
						console.log("报错",err)
					});
			},
			// 核销展会入场
			hexiao(unionid) {
			
				// 扫码结果
				var data = {
					exhId: this.exhInfo[1],
					params: { 
						unionid: unionid,
						userId: uni.getStorageSync("userInfo").userId,
						doorId: this.exhInfo[2],
					},
					// 可以在这里添加大门信息
				};
				if(this.isLixian==2){//离线
					this.saveZhanhui(unionid,"",1,false);//1是展会2是论坛  false离线
					return;
				}
				this.$axios
					.axios('post', this.$paths.visitpdaapi, data)
					.then(res => {
						console.log("和小爱", res)
						if (res.code == 200) {
							if(this.bg=='red'){
								this.$tools.showToast(unionid + "二维码过期了");
							}else{
								this.$tools.showToast(unionid + "核销成功");
							}
							
							
							this.scanResult=unionid;
							this.saveZhanhui(unionid,"",1,true);//1是展会2是论坛  true在线
						} else {
							this.$tools.showToast(res.msg);
						}
					})
					.catch(err => {
						
						console.log("报错了",err)
					});
			},
			// 获取展会列表
			getExhList() {
				var data = {};
				console.log("获取展会列表", data, this.$paths.listexhpda)
				this.$axios
					.axios('post', this.$paths.listexhpda, data)
					.then(res => {
						console.log("展会列博鳌", res)
						if (res.code == 200) {
							this.exhList = res.rows;
							if (uni.getStorageSync("zhanhuiInfo")) {
								uni.removeStorageSync("zhanhuiInfo")
							}

						} else {
							this.$tools.showToast(res.msg);
						}
					})
					.catch(err => {});
			},

			// 获取大门列表
			getMenList() {
				var data = {
					exhId: uni.getStorageSync("zhanhuiInfo").id,
				};
				console.log("获取大门列表", data, this.$paths.listmenpda);
				this.$axios
					.axios('post', this.$paths.listmenpda, data)
					.then(res => {
						if (res.code == 200) {
							this.menList = res.data;
						} else {
							this.menList = [];
							this.$tools.showToast(res.msg);
						}
					})
					.catch(err => {});
			},

			// 展会修改
			exhChange(res) {
				console.log(res)
				this.zhanhuiInfo = this.exhList[res.detail.value];

				uni.setStorageSync("zhanhuiInfo", this.zhanhuiInfo)
				uni.removeStorageSync("menType")
				this.menType = null;
				this.getMenList();
			},
			// 修改门
			menChange(res) {
				this.menType = this.menList[res.detail.value];
				uni.setStorageSync("menType", this.menType)
			},
			// 退出登录
			toLogin() {
				wx.showModal({
					title: "提示",
					content: "确定要退出吗",
					success: (res) => {
						if (res.confirm) {
							uni.reLaunch({ 
								url: "/pages/login/login"
							})
						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				})
			},
			
			
			// 打开数据库
			openSQL() {
				// 这个是查询有没有打开数据库
				let open = DB.isOpen();
				console.log("数据库状态", open ? "开启" : "关闭");
				if (!open) {
					DB.openSqlite()
						.then(res => {
							console.log("数据库已打开");
							this.selectHexiaoNum();
						})  
						.catch(error => {
							console.log("数据库开启失败");
						});
				}else{
					this.selectHexiaoNum();
				}
			},
			// 关闭数据库
			closeSQL() {
				// 这个是查询有没有打开数据库
				let open = DB.isOpen();
				if (open) {
					DB.closeSqlite()
						.then(res => {
							console.log("数据库已关闭");
						})
						.catch(error => {
							console.log("数据库关闭失败");
						});
				}
			},
			
			// 离线保存展会数据
			saveZhanhui(unionid,activityId,type,isLine){
				console.log("离线保存",unionid,activityId,type)
				var userInfo=uni.getStorageSync("userInfo");
				var exhInfo=uni.getStorageSync("exhInfo")
				if(!userInfo){
					this.$tools.showToast("请先登录");
					return false;
				}
				if(!exhInfo){
					this.$tools.showToast("请先扫码获取展会信息");
					return false;
				}     
				var obg={
					loginPhone:userInfo.phoneNum,
					loginName:userInfo.staffName,  
					exhId:exhInfo[1],
					activityId:activityId,
					doorId:exhInfo[2],
					unionid:unionid,
					userId:"",//离线没有这个
					type:type,  
					'isLine':isLine,
					createTime:this.formatDate(new Date().getTime())
				}
				if(this.isLixian==1||(userInfo&&userInfo.userId)){
					obg.userId=userInfo.userId;
				}  
				obg.isUpload=false;
				    
				let time = this.formatDate(new Date().getTime());
				let sql = `'${obg.loginPhone}','${obg.loginName}','${obg.exhId}','${obg.activityId}','${obg.doorId}','${obg.unionid}','${obg.userId}','${obg.type}','${obg.isLine}','${obg.isUpload}','${obg.createTime}'`;
				let condition = "'loginPhone','loginName','exhId','activityId','doorId','unionid','userId','type','isLine','isUpload','createTime'";
				// 新增 DB.insertTableData(表名, 对应表头列的数据)
				DB.insertTableData("scan_info", sql, condition)
					.then(res => {
						console.log("新增数据成功");
						this.updateHexiaoNum();  
						if(!isLine){
							this.scanResult=unionid;  
							if(type==1){
								this.$tools.showToast(unionid+"入场离线核销成功");
							}else if(type==2){
								this.$tools.showToast(unionid+"论坛离线核销成功");
							}
						}
						
					})  
					.catch(error => {
						console.log("失败", error);
					});
			},
			
			// 查询当前核销的数量
			selectHexiaoNum(){
					var time=this.formatDate1(new Date().getTime());
					var exhInfo=this.exhInfo;
					var userInfo=uni.getStorageSync("userInfo");
					DB.selectTableData("statistics_num", 'exhId',exhInfo[1],'doorId',exhInfo[2],'loginPhone',userInfo.phoneNum,
					'dayTime',time)
						.then(res => {  
							console.log("查询成功",res);
							if(res.length>0){
								var selResultCount=res[0];
								selResultCount.count=parseInt(selResultCount.count);
								this.selResultCount=selResultCount;
							}else{
								this.$tools.showToast("查询失败");
							}
						})                   
						.catch(error => {   
						});
			},
			// 修改核销的数量
			updateHexiaoNum(){
				var selResultCount=this.selResultCount;
				selResultCount.count=selResultCount.count+1;
				var updateTxt="count="+parseInt(selResultCount.count);
				this.selResultCount=selResultCount;
				DB.updateTableData("statistics_num",updateTxt,"id",selResultCount.id)
					.then(res => {  
						console.log("修改成功",res);
					})          
					.catch(error => {  
						console.log("修改数量失败",error);
						this.$tools.showToast("修改数量失败");
					});
			},
			// 提示框
			showToast: function(str) {
				uni.showToast({
					icon: "none",
					title: str,
					mask: true
				});
			},
			 
			// 时间戳转年月日
			formatDate(data) {
				let now = new Date(data);
				var year = now.getFullYear(); //取得4位数的年份
				var month =
					now.getMonth() + 1 < 10 ?
					"0" + (now.getMonth() + 1) :
					now.getMonth() + 1;
				var date = now.getDate() < 10 ? "0" + now.getDate() : now.getDate();
				var hour = now.getHours() < 10 ? "0" + now.getHours() : now.getHours();
				var minute =
					now.getMinutes() < 10 ? "0" + now.getMinutes() : now.getMinutes();
				var second =
					now.getSeconds() < 10 ? "0" + now.getSeconds() : now.getSeconds();
				return (
					year +
					"-" +
					month +
					"-" +
					date +
					" " +
					hour +
					":" +
					minute +
					":" +
					second
				);
			},
			
			formatDate1(data) {
				let now = new Date(data);
				var year = now.getFullYear(); //取得4位数的年份
				var month =
					now.getMonth() + 1 < 10 ?
					"0" + (now.getMonth() + 1) :
					now.getMonth() + 1;
				var date = now.getDate() < 10 ? "0" + now.getDate() : (now.getDate());
				var hour = now.getHours() < 10 ? "0" + now.getHours() : now.getHours();
				var minute =
					now.getMinutes() < 10 ? "0" + now.getMinutes() : now.getMinutes();
				var second =
					now.getSeconds() < 10 ? "0" + now.getSeconds() : now.getSeconds();
				return (
					year +
					"-" +
					month +
					"-" +
					date 
				);
			}
		}
	}
</script>

<style>
	.btncen {
		width: 700rpx;
		height: 250rpx;
		/* background-color: green; */
		line-height: 350rpx;
		text-align: center;
		color: white;
		border-radius: 2%;
		color: white;
	}
	 
	.btncen1 {
		width: 750rpx;
	}

	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
