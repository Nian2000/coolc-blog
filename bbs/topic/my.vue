<template>
	<view>
		<cu-custom bgColor="bg-gradual-blue" :isBack="true"><block slot="content">我的商品</block></cu-custom>

		<un-login v-if="unLogin"></un-login>
		<view v-else>
			<view class="text-gray padding justify-center flex" v-if="Object.keys(list).length == 0"><text>暂无数据</text></view>
			<view class="cu-card case isCard">
				<view class="cu-item shadow" v-for="(item, index) in list" :key="index">
					<view @click="goDetail(item._id)" class="image image-hidden">
						<image class="sglist-imglist-img" mode="widthFix" :src="item.imgList[0].imgurl"></image>
						<view class="cu-bar bg-shadeBottom">
							<text class="text-cut">{{ item.title }}</text>
						</view>
					</view>
					<view class="text-content padding-sm text-cut" style="width:100%;">{{ item.content }}</view>
					<view class="content flex-sub">
						<view class="text-gray flex justify-around padding-bottom">
							<view @click="goEdit(item._id)" class="text-blue">
								<text class="cuIcon-edit"></text>
								编辑
							</view>
							<view @click="goCode(item._id)" class="text-green">
								<text class="cuIcon-attention"></text>
								库存
							</view>
							<view @click="del(item._id)" class="text-red">
								<text class="cuIcon-all"></text>
								更多
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			pageLoad: false,
			list: [],
			isFirst: true,
			start: 0,
			limit: 4,
			unLogin: true,
			ssuserid: ''
		};
	},
	onLoad: function() {
		this.getPage();
	},
	onPullDownRefresh: function() {
		this.getPage();
		uni.stopPullDownRefresh();
	},
	onReachBottom: function() {
		this.getList();
	},
	methods: {
		goDetail: function(id) {
			uni.navigateTo({
				url: 'show?id=' + id
			});
		},
		goEdit: function(id) {
			uni.navigateTo({
				url: 'edit?id=' + id
			});
		},
		goCode: function(id) {
			uni.navigateTo({
				url: 'code?id=' + id
			});
		},
		getPage: async function() {
			var that = this;
			//start 检测登录
			this.ssuserid = getApp().globalData.ssuserid;
			this.unLogin = getApp().globalData.unLogin;
			if (this.ssuserid == '') {
				return false;
			}
			//End 检测是否登录
			uniCloud
				.callFunction({
					name: 'bbs_topic',
					data: {
						cloudAction: 'my',
						params: {
							ssuserid: this.ssuserid,
							start: 0,
							limit: this.limit
						}
					}
				})
				.then(result => {
					var res = result.result.data;
					that.list = res.list;
					that.isFirst = false;
					that.start = 0 + that.limit;
					that.pageLoad = true;
				});
		},
		getList: function() {
			var that = this;
			if (that.start == 0 && !that.isFirst) {
				return false;
			}
			uniCloud
				.callFunction({
					name: 'bbs_topic',
					data: {
						cloudAction: 'my',
						params: {
							ssuserid: this.ssuserid,
							start: this.start,
							limit: this.limit
						}
					}
				})
				.then(result => {
					var res = result.result.data;

					if (that.isFirst) {
						that.list = res.list;
						that.isFirst = false;
					} else {
						// console.log("getList")
						// console.log(that.start,res.list)
						if (res.list.length == 0) {
							that.start = 0;
							return false;
						}
						for (var i in res.list) {
							that.list.push(res.list[i]);
						}
					}
					that.start = that.start + that.limit;
				});
		},
		del: function(id) {
			var that = this;
			uni.showActionSheet({
				itemList: ['复制链接', '分享商品', '删除商品'],
				success: function(res) {
					if (res.tapIndex == 0) {
					} else if (res.tapIndex == 1) {
					} else if (res.tapIndex == 2) {
						uni.showModal({
							title: '删除提示',
							content: '删除后不可恢复',
							success: function(res) {
								if (res.confirm) {
									uniCloud
										.callFunction({
											name: 'bbs_topic',
											data: {
												cloudAction: 'delete',
												params: {
													id: id,
													ssuserid: this.ssuserid
												}
											}
										})
										.then(res => {
											for (var i in that.list) {
												if (that.list[i]._id == id) {
													that.list.splice(i, 1);
												}
											}
										});
								}
							}
						});
					} else {
						uni.showToast({
							title: '无效'
						});
					}
				},
				fail: function(res) {
					console.log(res.errMsg);
				}
			});
		}
	}
};
</script>

<style lang="scss" scoped></style>
