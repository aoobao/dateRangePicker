<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>

		<u-button type="primary" @click="pickDateRange">选择日期区间</u-button>
		<DateRangePicker ref="range" />
	</view>
</template>

<script>
	import DateRangePicker from '@/components/DateRangePicker.vue'
	export default {
		components: {
			DateRangePicker
		},
		data() {
			return {
				// title: 'Hello',
				range: [],
				type: 'month' // month or range
			}
		},
		computed: {
			title() {
				if (this.range.length) {
					return this.range.join(' ~ ')
				} else {
					return 'Hello'
				}
			}
		},
		methods: {
			pickDateRange() {
				this.$refs.range.pick({
					type: this.type,
					range: this.range
				}).then(res => {
					this.type = res.type
					this.range = res.range
				}).catch((err) => {
					console.log(err)
				})
			},
		}
	}
</script>

<style>
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
