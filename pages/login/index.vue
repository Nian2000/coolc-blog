<template>
	<view class="login">
		<cu-custom bgColor="bg-gradual-blue" :isBack="true">
			<block slot="content">登录</block>
		</cu-custom>

		<view class="content padding-xl">
			<!-- 头部logo -->
			<view class="flex justify-center">
				<image :src="logoImage" mode="widthFix" class="header"></image>
			</view>

			<!-- 主体表单 -->
			<view class="main">
				<wInput v-model="username" type="text" maxlength="11" placeholder="请输入登录账户"></wInput>
				<wInput v-model="password" type="password" maxlength="11" placeholder="请输入登录密码" isShowPass></wInput>
			</view>
			<wButton text="登 录" :rotate="isRotate" @click.native="formSubmit"></wButton>
			<view class="flex justify-center margin-top-xl">
				<navigator class="cl-white pointer flex-1" url="/pages/register/index">注册</navigator>
				<!-- <navigator class="cl-white pointer" url="/pages/login/findpwd" >忘记密码</navigator> -->
			</view>
		</view>
	</view>
</template>

<script>
	import wInput from '@/components/watch-login/watch-input.vue';
	import wButton from '@/components/watch-login/watch-button.vue';

	export default {
		data: function() {
			return {
				// logo图片
				logoImage: 'https://vkceyugu.cdn.bspapp.com/VKCEYUGU-blogfaka/cab53950-fe29-11ea-b680-7980c8a877b8.png',
				pageLoad: false,
				pageData: {},
				notephone: '请输入手机号码',
				notepwd: '请输入密码',
				isWeixin: false,
				username: '',
				password: '',
				isRotate: false //是否加载旋转
			};
		},
		components: {
			wInput,
			wButton
		},
		onLoad: function() {
			this.isWeixin = this.app.isWeixin();
		},
		methods: {
			goWeixin: function() {
				var backurl = '/pages/fenlei/index';
				this.app.goH5WeiXin(backurl);
			},
			goHome: function() {
				var that = this;
				that.app.goHome();
			},
			tel: function() {
				if (this.notephone == '请输入手机号码') {
					this.notephone = '';
				} else if (this.notephone == '') {
					this.notephone = '请输入手机号码';
				}
			},
			pass: function() {
				console.log(this.notepwd);
				if (this.notepwd == '请输入密码') {
					this.notepwd = '';
				} else if (this.notepwd == '') {
					this.notepwd = '请输入密码';
				}
			},
			wxLogin: function(e) {
				var that = this;
				var user = e.detail.userInfo;
				uni.login({
					provider: 'weixin',
					success: function(res) {
						var logincode = res.code;
						console.log(logincode);
						uni.request({
							url: that.app.apiHost + '/?m=open_wxapp&a=Login&ajax=1',
							data: {
								code: logincode,
								nickname: user.nickName,
								user_head: user.avatarUrl,
								gender: user.gender
							},
							success: function(res) {
								if (res.data.data.action == 'login') {
									uni.showToast({
										title: '登录成功'
									});
									that.app.setAuthCode(res.data.data.authcode);
									that.app.setAuthCodeLong(res.data.data.authcodeLong);
									that.app.setOpenid(res.data.data.openid);
									setTimeout(function() {
										uni.navigateBack();
									}, 300);
									uni.setStorageSync('uniIdToken', res.result.token);
								} else if (res.data.data.action == 'openlogin') {
									uni.navigateTo({
										url: '../openlogin/index?openToken=' + res.data.data.openToken
									});
								}
							},
							fail: function(e) {
								console.log(e);
							}
						});
					}
				});
			},
			formSubmit(e) {
				var that = this;
				this.isRotate = true;
				uniCloud.callFunction({
					name: 'user-center',
					data: {
						action: 'login',
						params: {
							username: this.username,
							password: this.password
						}
					},
					success(e) {
						that.isRotate = false;
						uni.showToast({
							title: e.result.msg,
							icon: 'none'
						});
						if (e.result.code == 0) {
							uni.setStorageSync('uniIdToken', e.result.token);
							var callRes = uniCloud.callFunction({
								name: 'user-center',
								data: {
									action: 'checkToken',
									uniIdToken: e.result.token
								},
								success: function(callRes) {
									if (callRes.result.uid == undefined) {
										return false;
									}
									getApp().globalData.ssuserid = callRes.result.uid;
									getApp().globalData.unLogin = false;
									uni.reLaunch({
										url: '/pages/index/index'
									});
								}
							});
						}
					},
					fail(e) {
						console.error(e);
						uni.showModal({
							showCancel: false,
							content: '登录失败，请稍后再试'
						});
					}
				});
			}
		}
	};
</script>

<style lang="scss" scoped>
	.login {
		background-color: #ffffff;
		width: 100vw;
		height: 100vh;
	}

	.main {
		margin-top: 40upx;
	}

	.header {
		width: 30vw;
	}
</style>
