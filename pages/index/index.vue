<template>
	<view class="content">
		<uni-easyinput v-model="cityName" placeholder="请输入内容" suffixIcon="search" @iconClick="search"></uni-easyinput>
		<view class="top-info-box">
			<view class="time">
				<text>数据更新时间</text>
				<text>{{myTime}}</text>
			</view>
			<view class="address">
				{{cityList.name}}
			</view>
			<view class="temp">{{weatherInfo.temp}}℃</view>
			<view class="condition">
				<image :src="imgUrl" mode=""></image>
				<text>{{weatherInfo.text}}</text>
			</view>
			<view class="wind">
				<text>风向：{{weatherInfo.windDir}}</text>
				<text class="line">|</text>
				<text>风力：{{weatherInfo.windScale}}</text>
			</view>
			<view class="water">
				<text>降水量：{{weatherInfo.precip}}</text>
				<text class="line">|</text>
				<text>湿度：{{weatherInfo.humidity}}%</text>
			</view>
		</view>
		<view class="life-index">
			<view class="title">生活指数</view>
			<view class="item" v-for="item in lifeIndex" :key="item.type">
				<view class="name">{{item.name}}:{{item.category}}</view>
				<text class="advice">{{item.text}}</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				//输入的搜索城市名
				cityName: '桐乡',
				//搜索到的城市列表(暂时只显示一个)
				cityList: [],
				//生活指数
				lifeIndex:[],
				//天气信息
				weatherInfo:{},
				imgUrl:"",
				time:""
			}
		},
		 watch:{
		     weatherInfo(newVal,oldVal){
					 console.log("新",newVal,"旧",oldVal)
					 this.imgUrl="../../static/weather_icon/"+newVal.icon+".png"
				 }
		 },
		 computed:{
			 myTime(){
				 return this.time.slice(0,10)+" "+this.time.slice(11,16)
			 }
		 },
		onLoad() {
			this.search()
		},
		onPullDownRefresh(){
			this.search(()=>{
				uni.stopPullDownRefresh()
				uni.showToast({
					title:"更新天气成功",
					icon:"success"
				})
			})
		},
		methods: {
			//查询城市
			async search(callBack) {
				const res = await this.$myRequest({
					url: "/v2/city/lookup",
					data: {
						location: this.cityName,
						number:1,
						isSearch:1
					}
				})
				console.log(res)
				this.cityList = res.location[0]
				this.getLifeIndex()
				this.getWeatherInfo()
				this.getWeatherOneDay()
				callBack && callBack()
			},
			//获取实时天气
			getWeatherInfo(){
				this.$myRequest({
					url:"/v7/weather/now",
					data:{
						location:this.cityList.id
					}
				}).then((res)=>{
					this.weatherInfo = res.now
					this.time = res.now.obsTime
				})
			},
			//获取未来24小时天气预报
			getWeatherOneDay(){
				this.$myRequest({
					url:"/v7/weather/24h",
					data:{
						location:this.cityList.id
					}
				}).then((res)=>{
					console.log(res)
				})
			},
			//获取生活指数
			getLifeIndex(){
				this.$myRequest({
					url:"/v7/indices/1d",
					data:{
						location:this.cityList.id,
						type:0
					}
				}).then((res)=>{
					this.lifeIndex = res.daily
				})
			}
		}
	}
</script>

<style lang="scss">
	.content {
		display: flex;
		flex-direction: column;
	}
	.top-info-box{
		padding:30rpx 0 20rpx 0;
		background-color:#5CACEE;
		display: flex;
		flex-direction: column;
		align-items: center;
		color:#fff;
		font-size: 30rpx;
		position: relative;
		border-radius: 48rpx;
		.time{
			position: absolute;
			right: 10rpx;
			top: 10rpx;
			font-size: 24rpx;
			display: flex;
			flex-direction: column;
		}
		.line{
			margin: 0 10rpx;
		}
		.address{
			font-size: 40rpx;
		}
		.temp{
			font-size: 50rpx;
			margin-top: 20rpx;
		}
		.condition{
			margin: 20rpx 0;
			display: flex;
			image{
				margin-right: 10rpx;
				width: 50rpx;
				height: 50rpx;
			}
		}
	}

	.life-index{
		background-color: #F0F0F0;
		.title{
			text-align: center;
			font-size: 40rpx;
			border-bottom: 2px solid #000;
			height: 80rpx;
			line-height: 80rpx;
		}
		.item{
			border-bottom: 1px solid #007AFF;
			padding: 26rpx 34rpx;
			.name{
				font-size: 36rpx;
			}
			.advice{
				font-size: 24rpx;
			}
		}
	}
</style>
