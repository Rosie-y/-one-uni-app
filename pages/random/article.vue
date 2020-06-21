<template>
	<view class="index" style="">
		<view class="pagbg" :style="{backgroundColor:pageBg}"></view>
		<!-- 左侧弹窗菜单 -->
		<view class="menu">
			<uni-drawer :visible="menu" mode="left" @close="menu=false">
				<view style="padding: 30upx;">
					<uni-list class=menu-area>
						<uni-list-item title="我的收藏" :show-arrow="false" @click="getCollection()" />
						<uni-list-item title="阅读设置" :show-arrow="false" @click="click()" />
					</uni-list>
				</view>
			</uni-drawer>
		</view>
		<!-- 右侧弹窗菜单 -->
		<view class="more">
			<uni-drawer :visible="more" mode="right" @close="more=false">
				<view style="padding: 30upx;">
					<uni-list>
						<uni-list-item title="收藏" :show-arrow="false" @click="onClick(title,author)" />
						<uni-list-item title="分享" :show-arrow="false" @click="togglePopup('bottom', 'share')" />
						<uni-list-item title="前一天" :show-arrow="false" @click="previous(pre)" />
						<uni-list-item title="后一天" :show-arrow="false" @click="next(nex)" />
						<uni-list-item title="随机" :show-arrow="false" @click="random()" />
						<uni-list-item title="今日" :show-arrow="false" @click="today()" />
					</uni-list>
				</view>
			</uni-drawer>
			<!-- 阅读设置 -->
			<view class="bottomBox" :style="{color:fontColor,backgroundColor:menuBg,bottom:showit?'0':'-150px'}">
				<view style="overflow: hidden;">
					<view style="float: left;width: 50%;overflow: hidden;">
						<view style="float: left;width: 30%;line-height: 70upx;text-align: center;font-size: 24upx;">字体</view>
						<view style="float: left;width: 70%;height: 70upx;display: flex;align-content: center;justify-content: center;">
							<slider style="width: 100%;" :value="size" min="20" max="100" @changing="changeFontSize" @change="changeFontSize"
							 :activeColor="fontColor" :backgroundColor="pageBg" :block-color="fontColor" block-size="20" />
						</view>
					</view>
					<view style="float: left;width: 50%;overflow: hidden;">
						<view style="float: left;width: 30%;line-height: 70upx;text-align: center;font-size: 24upx;">间距</view>
						<view style="float: left;width: 70%;height: 70upx;display: flex;align-content: center;justify-content: center;">
							<slider style="width: 100%;" :value="lineHeight" min="50" max="150" @changing="changeLineHeight" @change="changeLineHeight"
							 :activeColor="fontColor" :backgroundColor="pageBg" :block-color="fontColor" block-size="20" />
						</view>
					</view>
				</view>
				<view style="overflow: hidden;">
					<view style="float: left;width: 15%;line-height: 100upx;text-align: center;font-size: 24upx;">背景</view>
					<view style="float: left;width: 85%;height: 100upx;display: flex;">
						<view class="sekuai" v-for="(item,index) in zhutiList" @tap="qiehuan(index)" :key="item.name" :style="{backgroundColor:item.pageBg,borderColor:dqzhuti==index?item.fontColor:'rgba(0,0,0,0)'}"
						 v-if="index!=1&&index!=2"></view>
					</view>
				</view>
				<view style="width: 100%;display: flex;" class="ddd">
					<view @click="qiehuan(dqzhuti==1?0:1)">
						<view><text class="tficon">{{dqzhuti==1?'&#xe699;':'&#xe612;'}}</text></view>
						<view>{{dqzhuti==1?'白天':'夜间'}}</view>
					</view>
					<view @click="qiehuan(dqzhuti==2?0:2)" :style="dqzhuti==2?'color:green':''">
						<view><text class="tficon">&#xe639;</text></view>
						<view>护眼</view>
					</view>
				</view>
			</view>
			<!--布局-->
			<view class="essay" v-for="item in list" :key="item.title" :style="{color:textColor,fontSize:forUpx(size)+'px',lineHeight:forUpx(lineHeight)+'px'}">
				<view class="title-area">
					<text class="title">{{item.title}}</text>
				</view>
				<view class="line"></view>
				<view class="author-area">
					<text class="author">{{item.author}}</text>
				</view>
				<view class="content-area">
					<rich-text class="content" :nodes="item.content"></rich-text>
				</view>
				<view class="line"></view>
				<view class="words-area">
					<text class="words">全文完&nbsp;&nbsp;&nbsp;&nbsp;共{{item.wc}}字</text>
				</view>
			</view>
			<!-- 底部分享弹窗 -->
			<view class="share">
				<uni-popup ref="showshare" :type="type" @change="change">
					<view class="uni-share">
						<text class="uni-share-title">分享到</text>
						<view class="uni-share-content">
							<view v-for="(item, index) in bottomData" :key="index" class="uni-share-content-box">
								<view class="uni-share-content-image">
									<image :src="item.icon" class="content-image" mode="widthFix" />
								</view>
								<text class="uni-share-content-text">{{ item.text }}</text>
							</view>
						</view>
						<text class="uni-share-btn" @click="cancel('share')">取消分享</text>
					</view>
				</uni-popup>
			</view>

		</view>
	</view>
</template>

<script>
	import zhuti from '../../zhuti'
	import uniPopup from '@/components/uni-popup/uni-popup.vue'
	import uniDrawer from '@/components/uni-drawer/uni-drawer.vue'
	import uniList from '@/components/uni-list/uni-list.vue'
	import uniListItem from '@/components/uni-list-item/uni-list-item.vue'
	import uniSection from '@/components/uni-section/uni-section.vue'
	import uniIcons from '@/components/uni-icons/uni-icons.vue'
	export default {
		components: {
			uniPopup,
			/* 弹出层 */
			uniIcons,
			/* 图标 */
			uniDrawer,
			/* 抽屉 */
			uniList,
			/* 列表 */
			uniListItem,
			/* 列表 */
			uniSection,
			zhuti
		},
		data() {
			return {
				showit: false, //菜单display
				dqzhuti: 0, //当前主题
				zhutiList: zhuti.data, //主题列表
				fontColor: 'rgb(100,103,120)', //菜单字体颜色
				pageBg: 'rgb(247,247,247)', //页面背景色
				menuBg: '#fff', //菜单背景色
				textColor: '#808080', //富文本文字颜色
				size: uni.getStorageSync('fontsize') ? uni.getStorageSync('fontsize') : 40, //正文字体大小
				lineHeight: uni.getStorageSync('lineHeight') ? uni.getStorageSync('lineHeight') : 70, //正文行间距
				refreshing: false,
				list: [],
				author:"",
				title:"",
				/* 存放数据 */
				prelist: [],
				pre: "",
				nexlist: [],
				nex: "",
				fetchPageNum: 1,
				menu: false,
				more: false,
				type: '',
				bottomData: [{
						text: '微信',
						icon: 'https://img-cdn-qiniu.dcloud.net.cn/uni-ui/grid-2.png',
						name: 'wx'
					},
					{
						text: '支付宝',
						icon: 'https://img-cdn-qiniu.dcloud.net.cn/uni-ui/grid-8.png',
						name: 'wx'
					},
					{
						text: 'QQ',
						icon: 'https://img-cdn-qiniu.dcloud.net.cn/uni-ui/gird-3.png',
						name: 'qq'
					},
					{
						text: '新浪',
						icon: 'https://img-cdn-qiniu.dcloud.net.cn/uni-ui/grid-1.png',
						name: 'sina'
					},
					{
						text: '百度',
						icon: 'https://img-cdn-qiniu.dcloud.net.cn/uni-ui/grid-7.png',
						name: 'copy'
					},
					{
						text: '其他',
						icon: 'https://img-cdn-qiniu.dcloud.net.cn/uni-ui/grid-5.png',
						name: 'more'
					}
				]
			}
		},
		onLoad(e) {
			this.getData();
			this.id = e.id; /* 防止app里由于渲染导致转场动画卡顿 */

		},
		onNavigationBarButtonTap(e) {
			console.log(e.index);
			if (e.index == 0) {
				this.menu = !this.menu;
				console.log("点击了菜单");
			};
			if (e.index == 1) {
				this.more = !this.more;
				console.log("点击了更多");
			}
		},
		onBackPress() {
			if (this.menu || this.more) {
				this.hide()
				return true
			};
			this.$refs['showshare'].close()
		},
		created() {
			var this_ = this;
			var zt = uni.getStorageSync('zhuti'); //主题索引
			if (zt) {
				this.dqzhuti = zt;
				this.fontColor = zhuti.data[zt].fontColor; //菜单字体颜色
				this.pageBg = zhuti.data[zt].pageBg; //页面背景色
				this.menuBg = zhuti.data[zt].menuBg; //菜单背景色
				this.textColor = zhuti.data[zt].textColor; //富文本文字颜色
			} else {
				this.dqzhuti = 0;
				this.fontColor = zhuti.data[0].fontColor; //菜单字体颜色
				this.pageBg = zhuti.data[0].pageBg; //页面背景色
				this.menuBg = zhuti.data[0].menuBg; //菜单背景色
				this.textColor = zhuti.data[0].textColor; //富文本文字颜色
			}
		},
		methods: {
			getData() {
				uni.request({
					url: this.$serverUrl + '/article/random?dev=1',
					success: (res) => {
						let obj = {
							curr: res.data.data.date.curr,
							prev: res.data.data.date.prev,
							next: res.data.data.date.next,
							title: res.data.data.title,
							author: res.data.data.author,
							digest: res.data.data.digest,
							content: res.data.data.content,
							wc: res.data.data.wc
						}
						this.title=res.data.data.title;
						this.author=res.data.data.author;
						this.pre = res.data.data.date.prev;
						let curr = res.data.data.date.curr;
						this.nex = res.data.data.date.next;
						console.log("title:"+this.title);
						console.log("author:"+this.author);
						console.log("pre:" + this.pre);
						console.log("curr:" + curr);
						console.log("nex:" + this.nex);
						this.list.push(obj);
						console.log("obj" + obj);
						console.log("list:" + this.list);
					},
					fail: (e) => {
						uni.showModal({
							content: e.errMsg,
							showCancel: false
						})
					}
				});
			},
			show() {
				this.menu = true;
				this.more = true
			},
			hide() {
				this.menu = false;
				this.more = false;
			},
			/* 跳转到collection页面 */
			getCollection(e) {
				uni.navigateTo({
					url: '../collect/collection'
				})
			},
			/* 跳转到setting页面 */
			Setting(e) {
				uni.navigateTo({
					url: '../setting/setting'
				})
			},
			/* 跳转到center页面 */
			center(e) {
				uni.navigateTo({
					url: '../center/center'
				})
			},
			/* 跳转到previous页面 */
			previous(e) {
				uni.request({
					url: this.$serverUrl + '/article/day?dev=1&date= ' + e,
					success: (res1) => {
						let obj = {
							curr: res1.data.data.date.curr,
							prev: res1.data.data.date.prev,
							next: res1.data.data.date.next,
							title: res1.data.data.title,
							author: res1.data.data.author,
							digest: res1.data.data.digest,
							content: res1.data.data.content,
							wc: res1.data.data.wc
						}
						if (this.list == [] && this.prelist == []) {
							this.prelist.push(obj);
							this.list = this.prelist;
						} else {
							this.list = [];
							this.prelist = [];
							this.prelist.push(obj);
							this.list = this.prelist;
						}
						var title1 = res1.data.data.title;
						this.title = title1;
						var author1 = res1.data.data.author;
						this.author = author1;
						console.log("title:" + this.title);
						console.log("author:" + this.author);
						this.pre = res1.data.data.date.prev;
						this.nex = res1.data.data.date.next;
						let curr = res1.data.data.date.curr;
						console.log("curr:" + curr);
					}
				})
			},
			/* 跳转到next页面 */
			next(e) {
				uni.request({
					url: this.$serverUrl + '/article/day?dev=1&date= ' + e,
					success: (res1) => {
						let obj = {
							curr: res1.data.data.date.curr,
							prev: res1.data.data.date.prev,
							next: res1.data.data.date.next,
							title: res1.data.data.title,
							author: res1.data.data.author,
							digest: res1.data.data.digest,
							content: res1.data.data.content,
							wc: res1.data.data.wc
						}
						if (this.list == [] && this.nexlist == []) {
							this.nexlist.push(obj);
							this.list = this.nexlist;
						} else {
							this.list = [];
							this.nexlist = [];
							this.nexlist.push(obj);
							this.list = this.nexlist;
						}
						var title1 = res1.data.data.title;
						this.title = title1;
						var author1 = res1.data.data.author;
						this.author = author1;
						console.log("title:" + this.title);
						console.log("author:" + this.author);
						this.pre = res1.data.data.date.prev;
						this.nex = res1.data.data.date.next;
						let curr2 = res1.data.data.date.curr;
						console.log("curr2:" + curr2);
					}
				})
			},
			/* 跳转到article页面 */
			random(e) {
				uni.navigateTo({
					url: '../random/article'
				})
			},
			/* 跳转到index页面 */
			today(e) {
				uni.navigateTo({
					url: '../current/index'
				})
			},
			onClick(title,author) {
				var strToObj={
					title: title,
					author: author
				}
				uni.getStorage({ //获取本地收藏数据,第一次获取因为还没有保存title键，会回调fail函数，可以在fail函数里保存第一次要保存的数据，
						key: 'title',
						success: function(res) { //回调成功则说明不是第一次保存数据，
							for (var i = 0; i < res.data.length; i++) { //过滤已收藏的数据
								if (res.data[i] != strToObj) {
									continue;
								} else { //重复收藏则提示
									console.log("不要重复收藏" + res.data[i]);
									uni.showToast({
										title: '请不要重复收藏',
										icon: 'none'
									})
									return false; //退出不再往下走
								}
							}
							res.data.push(strToObj); //将新的数据添加进已保存的数据中
							uni.setStorage({ //保存到本地缓存
								key: 'title',
								data: res.data, //将添加了新数据的数组重新保存
								success: function() {
									console.log('保存到本地成功');
									uni.showToast({
										title: '收藏成功',
										icon: 'success'
									});
								}
							});
						},
						fail: function() { //第一次保存则会来到这里，第二次或以上次则不会来到这里
							uni.setStorage({ //保存到本地
								key: 'title', //保存的键名
								data: [strToObj], //键数据保存为数组
								success: function() {
									console.log('首次收藏文章成功并保存到本地');
									uni.showToast({
										title: '首次收藏成功',
										icon: 'success'
									});
								}
							})
						}
					})
				},
			togglePopup(type, open) {
				switch (type) {
					case 'bottom':
						this.content = '分享'
						break

				}
				this.type = type
				this.$nextTick(() => {
					this.$refs['show' + open].open()
				})
			},
			cancel(type) {
				this.$refs['show' + type].close()
			},
			change(e) {
				console.log('是否打开:' + e.show)
			},
			//修改正文字体大小
			changeFontSize(e) {
				this.size = e.detail.value;
				uni.setStorageSync('fontsize', e.detail.value);
			},
			//修改正文行间距
			changeLineHeight(e) {
				this.lineHeight = e.detail.value;
				uni.setStorageSync('lineHeight', e.detail.value);
			},
			click() {
				this.showit = !this.showit;
			},
			//切换主题
			qiehuan(e) {
				this.fontColor = zhuti.data[e].fontColor; //菜单字体颜色
				this.pageBg = zhuti.data[e].pageBg; //页面背景色
				this.menuBg = zhuti.data[e].menuBg; //菜单背景色
				this.textColor = zhuti.data[e].textColor; //富文本文字颜色
				uni.setStorageSync('zhuti', e);
				this.dqzhuti = e;
			},
			forUpx(e) {
				return uni.upx2px(e)
			}
		}
	}
</script>

<style>
	page {
		letter-spacing: 6upx;
	}

	.bottomBox {
		padding: 0 0 20upx 0;
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
		opacity: 1;
		z-index: 9;
	}

	.bottomBox .ddd>view {
		width: 100%;
		text-align: center;
		font-size: 24upx;
		line-height: 40upx;
	}

	.bottomBox .ddd image {
		width: 40upx;
		height: 40upx;
	}

	@font-face {
		font-family: "ydiconfont";
		src: url('https://at.alicdn.com/t/font_1282539_9h0uwv1sxps.ttf') format('truetype');
		/* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
	}

	.tficon {
		font-family: ydiconfont;
		font-size: 34upx;
	}

	.sekuai {
		width: 150upx;
		height: 100upx;
		background-color: #EC706B;
		border-radius: 12upx;
		border: 5upx solid #000;
		transform: scale(0.4);
		margin: -10upx -35upx 0;
	}

	.pagbg {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	.essay {
		/* width: calc(100% - 40upx); */
		font-size: 45upx;
		line-height: 90upx;
		position: relative;
		text-indent: calc(2em + 12upx);
		margin: 0 auto;
		z-index: 5;
		white-space: normal;
		word-break: break-all;
		word-wrap: break-word;
		overflow: hidden;
		padding: 0 20upx 20upx;
	}

	.title-area {
		text-align: center;
		margin-bottom: 10px;
		margin-top: 10px;
		font-size: 18px;
	}

	.title {
		text-align: center;
	}

	.author-area {
		text-align: center;
		margin-top: 5px;
		margin-bottom: 10px;
	}

	.author {
		color: #808080;
		font-size: 13px;
		font-weight: 300;
	}

	..content-area {
		margin-bottom: 15px;
		margin-left: 8px;
		margin-right: 8px;
	}

	.content {
		font-size: 16px;
	}

	.line {
		border-top: 1px solid #dddddd;
		margin-left: 3px;
		margin-right: 3px;
	}

	.words-area {
		text-align: center;
		margin-top: 10px;
		margin-bottom: 10px;
	}

	.words {
		color: #808080;
		font-size: 13px;
		font-weight: 200;
	}

	/* 底部分享 */
	.uni-share {
		/* #ifndef APP-NVUE */
		display: flex;
		flex-direction: column;
		/* #endif */
		background-color: #fff;
	}

	.uni-share-title {
		line-height: 60rpx;
		font-size: 24rpx;
		padding: 15rpx 0;
		text-align: center;
	}

	.uni-share-content {

		display: flex;

		flex-direction: row;
		flex-wrap: wrap;
		justify-content: center;
		padding: 15px;
	}

	.uni-share-content-box {

		display: flex;

		flex-direction: column;
		align-items: center;
		width: 200rpx;
	}

	.uni-share-content-image {

		display: flex;

		flex-direction: row;
		justify-content: center;
		align-items: center;
		width: 60rpx;
		height: 60rpx;
		overflow: hidden;
		border-radius: 10rpx;
	}

	.content-image {
		width: 60rpx;
		height: 60rpx;
	}

	.uni-share-content-text {
		font-size: 26rpx;
		color: #333;
		padding-top: 5px;
		padding-bottom: 10px;
	}

	.uni-share-btn {
		height: 90rpx;
		line-height: 90rpx;
		font-size: 14px;
		border-top-color: #f5f5f5;
		border-top-width: 1px;
		border-top-style: solid;
		text-align: center;
		color: #666;
	}

	.menu {
		background-color: #2a2a29;
	}
</style>
