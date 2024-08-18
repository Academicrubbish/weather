<template>
	<view>
		<view class="cu-card case">
			<view class="cu-item shadow border" style="background-color: rgba(255, 255, 255, 0.8);">
				<view class="today-title">
					<text>
						<text class="cuIcon-locationfill" style="color: #1e86ee;" />
						{{loctionData.address.district+' '+loctionData.address.street}}
					</text>
					<text style="text-align: right;">{{now.obsTime.split('T')[0]}}</text>
				</view>
				<view class="today-weather">
					<view class="left">
						<image :src="require('@/static/weather/' + now.icon + '.png')">
					</view>
					<view class="right">
						<view class="content">
							<view class="temp">{{now.temp}}
								<view class="cu-tag tag">°C</view>
							</view>
							<view>{{now.text}}</view>
						</view>
					</view>
				</view>
				<view class="today-nowair">
					<span>
						<text class="cuIcon-timefill" />
						{{ summary }}
					</span>
					<span>
						<text class="cuIcon-evaluate_fill" />
						空气{{ nowAir.category+' '+ nowAir.aqi}}
					</span>

				</view>
				<view class="line" />
				<view class="today-other">
					<view class="flex-item">
						<span>
							<image src="@/static/WSpeed.png">
						</span>
						<span class="data">{{now.windSpeed}}km/h</span>
						<span class="describe">风速</span>
					</view>
					<view class="flex-item">
						<span>
							<image src="@/static/ATemp.png">
						</span>
						<span class="data">{{now.feelsLike}}°</span>
						<span class="describe">体感温度</span>
					</view>
					<view class="flex-item">
						<span>
							<image src="@/static/Humidity.png">
						</span>
						<span class="data">{{now.humidity}}%</span>
						<span class="describe">湿度</span>
					</view>
				</view>
			</view>
		</view>
		<view class="cu-card case">
			<view class="cu-item shadow border" style="margin-top: 0;">
				<view class="future-weather">
					<view v-for="(item,index) of daily" :key="item.fxDate">
						<span class="date">{{item.fxDate.split('-')[1]}}-{{item.fxDate.split('-')[2]}}</span>
						<span>
							<image :src="require('@/static/weather/' + item.iconDay + '.png')">
						</span>
						<span class="temp">{{item.tempMax}}°/{{item.tempMin}}°</span>
						<span class="air">{{air[index].category}}</span>
					</view>
				</view>
			</view>
		</view>
		<view class="cu-card case">
			<view class="cu-item shadow border" style="margin-top: 0;">
				<view class="hour-title">
					<text class="cuIcon-time"></text>24小时预报
				</view>
				<view class="hour-weather">
					<w-chart ref="chart"></w-chart>
				</view>
			</view>
		</view>
		<view class="finally">
			敬请期待等多功能
		</view>
	</view>
</template>

<script>
	import { getWeather, getWill, getAir, getHour, getNowAir, getMinutely } from '../../api/weather.js'
	import wChart from '@/component/w-chart/index.vue'
	export default {
		components: { wChart },
		data() {
			return {
				loctionData: {
					longitude: 0, //经度
					latitude: 0, //纬度
					address: {}
				},
				now: { icon: '100', obsTime: '2023-02-14T00:46+08:00' }, //实时天气信息
				daily: [], //未来3天天气
				air: [], //未来3天空气质量
				nowAir: {}, //空气质量
				summary: '', //分钟级降水
			}
		},
		methods: {
			requestData() {
				let lon = this.loctionData.longitude
				let lat = this.loctionData.latitude
				let requestData = lon + ',' + lat
				//24h天气画图
				this.$refs.chart.getServerData(requestData)
				//实时天气				
				getWeather(requestData).then(res => {
					if (res[1].data.code != '200') {
						this.$message('暂无此地区实时天气信息')
					} else {
						this.now = res[1].data.now
					}
				})
				//未来3天天气
				getWill(requestData).then(res => {
					if (res[1].data.code != '200') {
						this.$message('暂无此地区未来3天天气信息')
					} else {
						this.daily = res[1].data.daily.slice(0, 4)
					}
				})
				//未来3天天气质量
				getAir(requestData).then(res => {
					if (res[1].data.code != '200') {
						this.$message('暂无此地区未来3天天气质量信息')
					} else {
						this.air = res[1].data.daily.slice(0, 4)
					}
				})
				//当前实时天气质量
				getNowAir(requestData).then(res => {
					if (res[1].data.code != '200') {
						this.$message('暂无此地区实时天气质量信息')
					} else {
						this.nowAir = res[1].data.now
					}
				})
				//分钟级降水
				getMinutely(requestData).then(res => {
					if (res[1].data.code != '200') {
						this.$message('暂无此地区分钟级降水信息')
					} else {
						this.summary = res[1].data.summary
					}
				})
			},
			getRegeo() {
				let _this = this
				uni.getLocation({
					type: 'gcj02',
					geocode: true,
					success: function(res) {
						_this.loctionData.longitude = res.longitude
						_this.loctionData.latitude = res.latitude
						_this.loctionData.address = res.address
						_this.requestData()
					}
				})
			}
		},

		onLoad() {
			this.getRegeo()
		}
	}
</script>

<style lang="scss" scoped>
	.border {
		border-radius: 50rpx;
	}

	.today-title {
		font-size: 30rpx;
		color: gray;
		padding: 30rpx;
		display: flex;

		>text {
			line-height: 30rpx;
			flex: 1;
		}
	}

	.today-weather {
		height: 300rpx;

		>view {
			display: inline-block;
			height: 100%;
			width: 50%;
			vertical-align: middle;
			position: relative;
		}

		.left {
			image {
				width: 200rpx;
				height: 200rpx;
				position: absolute;
				top: 50%;
				left: 50%;
				transform: translate(-20%, -50%);
			}
		}

		.right {
			.content {
				position: absolute;
				top: 50%;
				left: 50%;
				transform: translate(-80%, -50%);

				.temp {
					font-size: 150rpx;
					text-align: center;
					position: relative;

					.tag {
						padding: 0;
						background-color: transparent;
						position: absolute;
						top: 10rpx;
					}
				}
			}
		}
	}

	.today-nowair {
		height: 50rpx;
		text-align: center;

		>span {
			color: white;
			background-color: gray;
			border-radius: 20rpx;
			margin: 0 10rpx;
			padding: 0 10rpx;

			>text {
				margin-right: 10rpx;
			}
		}
	}

	.line {
		height: 1rpx;
		background-color: #f7f7f7;
		width: 86%;
		margin-left: 7%;
	}

	.today-other {
		height: 150rpx;
		text-align: center;
		align-items: center;
		display: flex;

		.flex-item {
			flex: 1;

			>span {
				display: block;
			}

			image {
				width: 35rpx;
				height: 35rpx;
			}

			.data {
				font-size: 20rpx;
			}

			.describe {
				font-size: 20rpx;
				color: gray;
			}
		}
	}

	.future-weather {
		background-color: #000032;
		color: white;
		height: 200rpx;
		text-align: center;
		align-items: center;
		display: flex;

		>view {
			flex: 1;

			>span {
				display: block;
			}

			image {
				height: 35rpx;
				width: 35rpx;
			}

			.date {
				font-size: 20rpx;
			}

			.temp {
				font-size: 20rpx;
			}

			.air {
				font-size: 20rpx;
			}
		}
	}

	.hour-title {
		line-height: 60rpx;
		padding-left: 40rpx;
		background-color: #000032;
		color: white;
	}

	.hour-weather {
		height: 400rpx;
		padding: 40rpx;
		background-color: #000032;
		color: white;
	}

	.finally {
		font-size: 20rpx;
		color: gray;
		line-height: 120rpx;
		text-align: center;
	}
</style>
