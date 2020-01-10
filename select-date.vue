<template>
	<view class="calendarBox column-js-ac">
		<!-- <view class="bgcMonth font-mcA5">{{showMonth}}</view> -->
		<view class="top row-jc-ac font-scD1">
			<view class="icon-arrow-left fontSizeC" @click="before"></view>
			<view class="fontSizeE showDate">{{showYear}}年    {{showMonth}}月</view>
			<view class="icon-arrow-left fontSizeC icon-right" @click="after"></view>
			<view class="allselect fontSizeC" :class="[isall ? 'font-scA1' : 'font-mcA3']" @click="allSelect">{{isall ? '取消' : '全选'}}</view>
		</view>
		<view class="section column-js-ac">
			<view class="weeks row-js-ac">
				<view class="week fontSizeC font-scA1">日</view>
				<view class="week fontSizeC font-scA1">一</view>
				<view class="week fontSizeC font-scA1">二</view>
				<view class="week fontSizeC font-scA1">三</view>
				<view class="week fontSizeC font-scA1">四</view>
				<view class="week fontSizeC font-scA1">五</view>
				<view class="week fontSizeC font-scA1">六</view>
			</view>
			<view class="days row-js-ac fontSizeE">
				<view class="day" v-for="(item,index) in days" :key="index" :class="item.class" @click="clickDay(index)">{{item.value}}</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				days: [],
				showYear: '', //当前显示的年
				showMonth: '',
				selectDays:[],
				isall: false,
			}
		},
		created() {
			this.getNow()
		},
		props:['reseted'],
		watch: {
			showMonth(newValue,oldValue) {
				this.getDays(this.showYear,this.showMonth)
			},
			reseted(newValue,oldValue) {
				this.reset()
			}
		},
		methods: {
			// 获取当前时间
			getNow() {
				var date = new Date()
				this.showYear = date.getFullYear()
				this.showMonth = date.getMonth()+1
				// this.getWeek(date.getDay())
			},
			// 获取固定年月的天数
			getDays(year,month){
			    this.days=[];
			    var nums=new Date(year,month-1,1).getDay();
			    var d=new Date(year,month,0).getDate();
				var before = new Date(year,month-1,0).getDate()
			    for(var i=0;i<nums;i++){
			        this.days.push({value:before-nums+i+1,class:'font-scD2'});
			    }
			    for(var i=1;i<=d;i++){
			        this.days.push({value:i,class:'font-scD1'});
			    }
				for(var i=0;i<42-nums-d;i++){
				    this.days.push({value:i+1,class:'font-scD2'});
				}
				this.checkSelected()
			    // this.getDisabled();
			},
			// 点击上个月
			before() {
				if(this.showMonth === 1) {
					this.showMonth = 12
					this.showYear = this.showYear-1
				}else {
					this.showMonth = this.showMonth - 1
				}
			},
			// 点击下个月
			after() {
				if(this.showMonth === 12) {
					this.showMonth = 1
					this.showYear++
				}else {
					this.showMonth++
				}
			},
			clickDay(index) {
				if(this.days[index].class !== 'font-scD2') {
					var selectDate = this.showYear + '-'+ this.checkNum(this.showMonth) + '-' + this.checkNum(this.days[index].value)
					if(this.days[index].class === 'bgc-mcA3 font-scD3') {
						var idx = this.selectDays.findIndex(item => {return item===selectDate})
						 this.days[index].class = 'font-scD1'
						this.selectDays.splice(idx,1)
					}else {
						this.days[index].class = 'bgc-mcA3 font-scD3'
						this.selectDays.push(selectDate)
					}
				}
				this.$emit('jbClick',this.selectDays)
			},
			checkNum(num) {
				if(num < 10) {
					return '0'+num
				}else {
					return num
				}
			},
			checkSelected() {
				this.selectDays.forEach(item => {
					var year = parseInt(item.split('-')[0])
					var month = parseInt(item.split('-')[1])
					var day = parseInt(item.split('-')[2])
					if(this.showYear === year && this.showMonth === month) {
						var index = this.days.findIndex(item1 => {return item1.value === day && item1.class=== 'font-scD1'})
						this.days[index].class='bgc-mcA3 font-scD3'
					}
				})
			},
			// 点击全选
			allSelect() {
				var arr = this.days.filter(item => {return item.class !== 'font-scD2'})
				var flag = arr.every(item => {return item.class === 'bgc-mcA3 font-scD3'})
				this.isall = !flag
				if(flag) {
					this.days.forEach((item,index) => {
						if(item.class!== 'font-scD2') {
							item.class='font-scD1'
						}
					})
					this.cancelSelect()
				}else {
					this.days.forEach((item,index) => {
						if(item.class !== 'font-scD2') {
							item.class='bgc-mcA3 font-scD3'
						}
						
					})
					this.allSelected()
				}
				this.$emit('jbClick',this.selectDays)
			},
			// 全选
			allSelected() {
				this.days.forEach(item => {
					if(item.class !== 'font-scD2') {
						var date = this.showYear + '-'+ this.checkNum(this.showMonth) + '-' + this.checkNum(item.value)
						if(!this.selectDays.some(item1 => {return item1 === date})) {
							this.selectDays.push(date)
						}
					}
					
				})
			},
			// 全不选
			cancelSelect() {
				this.days.forEach(item => {
					if(item.class !== 'font-scD2') {
						var date = this.showYear + '-'+ this.checkNum(this.showMonth) + '-' + this.checkNum(item.value)
						var index = this.selectDays.findIndex(item1 => {return item1 === date})
						if(index !== 0) {
							this.selectDays.splice(index,1)
						}
					}
					
				})
			},
			reset() {
				this.days.forEach((item,index) => {
					if(item.class!== 'font-scD2') {
						item.class='font-scD1'
					}
				})
				this.selectDays = []
				this.$emit('jbClick',this.selectDays)
			}
			
		}
	}
</script>

<style scoped lang="less">
	.calendarBox {
		position: relative;
		width: 100%;
		font-size: 78upx;
	}
	.bgcMonth {
		// z-index: -1;
		position: absolute;
		top: 70upx;
	}
	.icon-right {
		transform: rotate(180deg)
	}
	.top {
		position: relative;
		width: 100%;
		margin-top: 11upx;
		.showDate {
			margin: 0 13upx;
		}
		.allselect {
			position: absolute;
			right: 13upx;
			cursor: pointer;
		}
	}
	.section {
		width: 100%;
		background: rgba(0, 0, 0, 0);
		.weeks {
			margin-left: -9upx;
			margin-top: 15upx;
			.week {
				width: 20upx;
				margin-left: 9upx;
				text-align: center;
			}
		}
		.days {
			margin-left: 5upx;
			.day {
				margin-left: 9upx;
				margin-top: 7upx;
				width: 20upx;
				height: 20upx;
				border-radius: 50%;
				line-height: 20upx;
				text-align: center;
				cursor: pointer;
			}
		}
	}
</style>
