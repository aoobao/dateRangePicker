# UNI-APP 日期区间筛选

## 核心组件所在文件 `components/DateRangePicker.vue`
### 组件中有使用到uview-ui 1.x,使用前需要引入uview组件库
[https://v1.uviewui.com/](https://v1.uviewui.com/)

### 组件中有使用dayjs处理日期 
`npm install dayjs --save`

## 基本使用方法
可参考页面 `pages/index/index`

### 将组件引入到template中

````
<template>
  <view>
    ....
		<DateRangePicker ref="range" />
  </view> 
</template>

````

### js代码调用
````
this.$refs.range.pick({
					type: this.type,
					range: this.range
				}).then(res => {
					this.type = res.type
					this.range = res.range
				}).catch((err) => {
					console.log(err)
				})

````
