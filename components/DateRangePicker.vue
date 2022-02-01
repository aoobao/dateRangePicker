<template>
	<u-popup :mask-close-able="maskCloseAble" border-radius="25" mode="bottom" v-model="visible" safe-area-inset-bottom
		:mask-custom-style="maskCustomStyle" :z-index="uZIndex">
		<view class="dr-picker">
			<view class="dr-picker-header" @touchmove.stop.prevent>
				<text class="picker-text u-btn-picker--tips" @click="cancel">{{cancelText}}</text>
				<view class="dr-picker-title">
					<text @click="changeType('month')" :class="type === 'month' ? 'active' : ''">按月</text>
					<text @click="changeType('range')" :class="type === 'range' ? 'active' : ''">时间段</text>
				</view>
				<text class="picker-text u-btn-picker--primary" @click="submit">{{confirmText}}</text>
			</view>

			<view class="u-picker-body">
				<picker-view class="u-picker-view" v-if="type === 'month' && visible" :value="value" @change="change"
					@pickstart="pickstart" @pickend="pickend">
					<picker-view-column>
						<view class="item" v-for="year in yearList" :key="year">{{year}}年</view>
					</picker-view-column>
					<picker-view-column>
						<!-- 微信小程序下会编译成0-11 -->
						<!-- <view class="item" v-for="i in 12" :key="i">{{i}}月</view> -->
						<view class="item" v-for="i in months" :key="i">{{i}}月</view>
					</picker-view-column>
				</picker-view>
				<template v-else-if="type === 'range' && visible">
					<view class="shortcuts">
						<view :class="currentBtnIndex === 0 ? 'active btn' : 'btn'" @click="resetThisWeek(0)">本周</view>
						<view :class="currentBtnIndex === 1 ? 'active btn' : 'btn'" @click="resetThisMonth(1)">本月</view>
						<view :class="currentBtnIndex === 2 ? 'active btn' : 'btn'" @click="resetThisQuarter(2)">本季度
						</view>
						<view :class="currentBtnIndex === 3 ? 'active btn' : 'btn'" @click="resetThisYear(3)">本年</view>
					</view>

					<view class="range" v-if="range.length === 2">
						<view :class="activeIndex === 0 ? 'date-text active':'date-text'" @click="setActiveIndex(0)">
							{{ range[0]  | toDateText}}
						</view>
						<view class="line"></view>
						<view :class="activeIndex === 1 ? 'date-text active':'date-text'" @click="setActiveIndex(1)">
							{{range[1] | toDateText}}
						</view>
					</view>

					<picker-view :value="value" @change="change" @pickstart="pickstart" @pickend="pickend"
						class="u-picker-view">
						<picker-view-column>
							<view class="item" v-for="year in yearList" :key="year">{{year}}年</view>
						</picker-view-column>
						<picker-view-column>
							<!-- <view class="item" v-for="i in 12" :key="i">{{i}}月</view> -->
							<view class="item" v-for="i in months" :key="i">{{i}}月</view>
						</picker-view-column>
						<picker-view-column>
							<view class="item" v-for="i in dayList" :key="i">{{i}}日</view>
						</picker-view-column>

					</picker-view>
				</template>

			</view>
		</view>
	</u-popup>
</template>

<script>
	import dayjs from 'dayjs'

	// var quarterOfYear = require('dayjs/plugin/quarterOfYear')
	// dayjs.extend(quarterOfYear)
	export default {
		name: "DateRangePicker",
		props: {
			maskCustomStyle: {
				type: Object,
				default () {
					return {}
				}
			},
			maskCloseAble: {
				type: Boolean,
				default: true
			},
			zIndex: {
				type: Number,
				required: false
			},
			cancelText: {
				type: String,
				default: '取消'
			},
			confirmText: {
				type: String,
				default: '确定'
			},
			minYear: {
				type: Number,
				default: 0
			},
			maxYear: {
				type: Number,
				default: 0
			}
		},
		data() {
			const months = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
			return {
				months,
				range: [],
				value: [0, 0, 0],
				type: 'month', // month / range
				visible: false,
				activeIndex: 0,
				currentBtnIndex: -1
			};
		},
		computed: {
			activeMonth() {
				const date = this.range[this.activeIndex]
				return dayjs(date).format('YYYY-MM')

			},
			dayList() {
				const date = this.activeMonth + '-01'
				const endDay = dayjs(this.getLastDay(date))
				const dayCount = endDay.date()

				const list = []
				for (var i = 1; i <= dayCount; i++) {
					list.push(i)
				}
				// console.log(endDay.format('YYYY-MM-DD'), dayCount, list)
				return list

			},
			maxYearValue() {
				if (this.maxYear) {
					return this.maxYear
				} else {
					const year = dayjs().year()
					return year
				}
			},
			minYearValue() {
				if (this.minYear) {
					return this.minYear
				} else {
					return this.maxYearValue - 5
				}
			},
			yearList() {
				const list = []
				for (var i = this.minYearValue; i <= this.maxYearValue; i++) {
					list.push(i)
				}
				return list
			},
			uZIndex() {
				return this.zIndex ? this.zIndex : this.$u.zIndex.popup;
			}
		},
		watch: {
			visible(val) {
				if (!val) {
					const reject = this.clearPromise(false)
					if (reject) {
						reject({
							type: 'close'
						})
					}
				}
			}
		},
		filters: {
			toDateText(val) {
				return dayjs(val).format('YYYY年M月D日')
			}
		},
		// just a test
		// created() {
		// 	this.pick({
		// 		type: 'range'
		// 	})
		// },
		methods: {
			resetThisWeek(currentBtnIndex) {
				const day = dayjs()
				let weekDay = day.day() // 获取星期
				if (weekDay === 0) weekDay = 7

				const startDay = day.subtract(weekDay - 1, 'day')
				const endDay = startDay.add(6, 'day')

				// 重置时间区间
				this.resetRangeDate(currentBtnIndex, startDay, endDay)

			},
			resetThisMonth(currentBtnIndex) {
				const day = dayjs()
				const startDay = day.startOf('month')
				const endDay = day.endOf('month')

				// 重置时间区间
				this.resetRangeDate(currentBtnIndex, startDay, endDay)
			},
			resetThisQuarter(currentBtnIndex) {
				const date = dayjs() // '2022-11-8'

				const month = ~~(date.month() / 3) // 0,1,2,3
				const startDay = dayjs(`${date.year()}-${month * 3 + 1}-01`)
				const endDay = startDay.add(3, 'month').subtract(1, 'day')

				// console.log(startDay.format('YYYY-MM-DD'), endDay.format('YYYY-MM-DD'))

				// 重置时间区间
				this.resetRangeDate(currentBtnIndex, startDay, endDay)
			},
			resetThisYear(currentBtnIndex) {
				const day = dayjs()
				const startDay = day.startOf('year')
				const endDay = day.endOf('year')

				// 重置时间区间
				this.resetRangeDate(currentBtnIndex, startDay, endDay)
			},
			resetRangeDate(currentBtnIndex, startDay, endDay) {
				this.currentBtnIndex = currentBtnIndex

				this.range = [startDay.format('YYYY-MM-DD'), endDay.format('YYYY-MM-DD')]

				this.resetValue(this.range[this.activeIndex])
			},
			// 时间区间切换
			setActiveIndex(index) {
				if (index === this.activeIndex) return
				this.activeIndex = index
				const date = this.range[index]

				this.resetValue(date)
			},
			change(e) {
				this.currentBtnIndex = -1
				// console.log(e)
				// debugger
				const value = e.detail.value
				const date = this.getDate(value)

				if (this.type === 'month') {
					const range = this.resetMonthRange(date)
					this.range = range
				} else {
					const range = [...this.range]
					range[this.activeIndex] = date
					// 如果前面的日期小于后面
					if (dayjs(range[0]).isAfter(dayjs(range[1]))) {
						if (this.activeIndex === 0) {
							range[1] = range[0]
						} else {
							range[0] = range[1]
						}
					}

					this.range = range
				}

				const day = Math.min(dayjs(date).date() - 1, value[2] || 0)

				this.value = [value[0], value[1], day]
			},
			getDate(value) {
				const year = this.yearList[value[0]]
				const month = ('0' + ((this.months[value[1]] || 1))).slice(-2)

				const maxDayValue = dayjs(`${year}-${month}-1`).endOf('month').date()

				const dayValue = Math.min(maxDayValue, (value[2] || 0) + 1)

				const day = ('0' + dayValue).slice(-2)
				return `${year}-${ month}-${day}`
			},
			clearPromise(isResolve) {
				let v
				if (isResolve) {
					v = this.resolve
				} else {
					v = this.reject
				}
				this.resolve = undefined
				this.reject = undefined

				return v
			},
			pick({
				type = 'month',
				range
			}) {
				return new Promise((resolve, reject) => {
					this.resolve = resolve
					this.reject = reject

					this.type = type
					this.currentBtnIndex = -1
					this.activeIndex = 0
					this.initRange(range)
					this.initValue()
					this.visible = true
				})
			},
			changeType(type) {
				this.type = type
				this.initRange(this.range)
				this.initValue()
			},
			// 初始化时间区间
			initRange(range) {
				if (!range || range.length !== 2) {
					this.range = this.resetMonthRange()
				} else {
					if (this.type === 'month') {
						// 按照区域的最大值 重置月份
						this.range = this.resetMonthRange(this.range[1])
					} else {
						// 乐观认为外部传入就是正确的
						this.range = [...range]
					}
				}

				// console.log('range', this.range)
			},
			resetMonthRange(d) {
				let t = d
				if (t) {
					t = t.replace(/-\d{1,2}$/, '') + '-01'
				}
				const day = dayjs(t)
				// 当前月的第一天
				const startDay = day.format('YYYY-MM') + '-01'
				const endDay = this.getLastDay(startDay) // 当前月最后一天
				const range = [startDay, endDay]
				return range
			},
			getLastDay(dateText) {

				const endDay = dayjs(dateText).endOf('month')
				// const firstDate = dayjs(dayjs(dateText).format('YYYY-MM') + '-01')

				// const endDay = firstDate.add(1, 'month').subtract(1, 'day')
				return endDay.format('YYYY-MM-DD')
			},
			// 初始化值
			initValue() {
				// 按月
				if (this.type === 'month') {
					const day = this.range[1]
					this.resetValue(day)
				} else {
					const day = this.range[this.activeIndex]
					this.resetValue(day)
				}
			},
			resetValue(d) {
				const day = dayjs(d)
				const year = day.year()
				const month = day.month()
				const date = day.date() - 1
				let yearIndex = this.yearList.findIndex(t => t === year)
				if (yearIndex === -1) yearIndex = 0

				// console.log([yearIndex, month, date])
				if (this.type === 'month') {
					this.value = [yearIndex, month]
				} else {
					this.value = [yearIndex, month, date]
				}
			},
			cancel() {
				const reject = this.clearPromise(false)
				if (reject) {
					reject({
						type: 'cancel'
					})
				}
				this.visible = false
			},
			submit() {
				// 等停下来后再提交,滚动过程中可能不准
				if (this.moving) {
					setTimeout(this.submit, 50)
					return
				}

				this.$nextTick(function() {

					const result = {
						type: this.type,
						range: [...this.range]
					}

					const resolve = this.clearPromise(true)
					resolve && resolve(result)

					this.$emit('submit', result)

					this.visible = false
				})
			},
			// 标识滑动开始，只有微信小程序才有这样的事件
			pickstart() {
				this.moving = true;
			},
			// 标识滑动结束
			pickend() {
				this.moving = false;
			},
		}
	}
</script>

<style lang="scss" scoped>
	.dr-picker {
		width: 100%;
		position: relative;
		background-color: #fff;
		// height: 822rpx;
	}

	.dr-picker-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-top: 26rpx;
		padding: 0 20rpx;

		.dr-picker-title {
			text {
				font-size: 32rpx;
				padding: 0 16rpx;
				color: #080707;
				font-weight: bold;
			}

			.active {
				color: #0070D2;
			}
		}
	}

	.u-picker-body {
		width: 100%;
		min-height: 500rpx;
		overflow: hidden;
		background-color: #fff;

		.u-picker-view {
			height: 450rpx;
			box-sizing: border-box;

			.item {
				// @include vue-flex;
				display: flex;
				flex-direction: row;
				align-items: center;
				justify-content: center;
				font-size: 32rpx;
				color: $u-main-color;
				padding: 0 8rpx;
			}
		}

		.shortcuts {
			padding: 0 24rpx;
			display: flex;
			align-items: center;
			justify-content: space-between;
			margin-top: 26rpx;

			.btn {
				width: 160rpx;
				height: 64rpx;
				display: flex;
				align-items: center;
				justify-content: center;
				background-color: #F7F8F9;
				color: #C8C9CC;
				font-size: 28rpx;

				&.active {
					color: #0070D2;
				}
			}
		}

		.range {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 0 24rpx;
			margin-top: 32rpx;
			margin-bottom: 16rpx;

			.date-text {
				width: 282rpx;
				height: 64rpx;
				background-color: #F7F8F9;
				color: #080707;
				display: flex;
				align-items: center;
				justify-content: center;

				&.active {
					color: #0070D2;
				}
			}

			.line {
				width: 44rpx;
				height: 2rpx;
				background-color: #080707;
			}
		}
	}

	.u-btn-picker--primary {
		color: $u-type-primary;
	}

	.u-btn-picker--tips {
		color: $u-tips-color;
	}
</style>
