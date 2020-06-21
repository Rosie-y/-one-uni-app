<template>
	<view>
		<view class="collect" v-for="(item,index) in list" :key="index" @longpress="remove">
			<label class="collect-body">
				<text class="title">{{item.title}}</text>
				<text class="author">{{item.author}}</text>
			</label>
		</view>
	</view>
</template>

<script>
	import uniList from '@/components/uni-list/uni-list.vue'
	import uniListItem from '@/components/uni-list-item/uni-list-item.vue'
	export default {
		components: {
			uniList,
			/* 列表 */
			uniListItem
			/* 列表 */
		},
		data() {
			return {
				list: []
			}
		},
		onLoad: function() {
			this.getdata();
		},
		methods: {
			getdata() {
				uni.getStorage({
					key: 'title',
					success: (res) => {
						console.log(res.data);
						this.list = res.data;
					}
				})
			},
			remove() {
				uni.showActionSheet({
					itemList: ['删除全部'],
					success: (res) => {
						if (res.tapIndex == 0) {
							uni.removeStorage({
								key: 'title',
								success: function() {
									this.list = []
									console.log('删除全部成功');
								}
							})
						}
					}
				})
			}
		}
	}
</script>

<style>
	.collect-body {
		margin: 10px 10px 10px 10px;
	}

	.title {
		font-family: block;
		font-size: 15px;
		margin-left: 10px;
		margin-top: 15px;
		margin-bottom: 15px;
		position: absolute;
		left: 0;
	}

	.author {
		position: absolute;
		right: 0;
		margin-top: 15px;
		margin-bottom: 15px;
		margin-right: 10px;
		font-size: 13px;
		font-family: block;
		color: #808080;
	}
</style>
