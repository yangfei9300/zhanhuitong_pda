<template>
	<view>
		
	</view>
</template>

<script>
	const CryptoJS = require('crypto-js') // 引用AES源码js
	const key = CryptoJS.enc.Utf8.parse('2020081720200817') // 十六位十六进制数作为密钥
	
export default {
	
	data() {
		return {};
	},
	onLoad() {
		this.jiema();
	},
	methods: {
		
		jiema() {
			var txt = 'oxnFIhz8rcarN2E0p6CQFH8JME1dIwlshp4L5+/QwKM=';
			var resu = this.Decrypt(txt);
			console.log('resu', resu);
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
		
	}
};
</script>

<style></style>
