<template>
	<view class="wrap bgc-scB2 font-scD1 column-js-ac">
		<view class="yearMonth row-jb-ac">
			<view class="fontSizeD font-scD2 icon-arrow-left icon-left" @click="before"></view>
			<view class="fontSizeF">{{showYear}}年   {{showMonth}}月</view>
			<view class="fontSizeD font-scD2 icon-arrow-left icon-right" @click="after"></view>
		</view>
		<view class="date fontSizeT medium">{{selectDate}}</view>
		<view class="fontSizeG">{{selectDay}}</view>
		<view class="dateBox">
			<view class="weeks row-js-ac">
				<view class="font-scA1 fontSizeC week center">日</view>
				<view class="font-scA1 fontSizeC week center">一</view>
				<view class="font-scA1 fontSizeC week center">二</view>
				<view class="font-scA1 fontSizeC week center">三</view>
				<view class="font-scA1 fontSizeC week center">四</view>
				<view class="font-scA1 fontSizeC week center">五</view>
				<view class="font-scA1 fontSizeC week center">六</view>
			</view>
			<view class="days row-js-ac">
				<view class="day fontSizeE column-js-ac" v-for="(item,index) in days" :key="index" :class="item.class" @click="clickDay(index)">
				{{item.value}}
				<view class="dot" v-if="isClass(index)" :class="[selectDate === item.value ? 'bgc-scD3' : 'bgc-mcA3']"></view>
				</view>
			</view>
		</view>
		<view class="backToday border-mcA3 fontSizeB font-mcA3 center medium" @click="backToday">
			返回今天
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				days: [], // 显示对应日期的数组
				showYear: '', //当前显示的年
				showMonth: '', //当前显示的月
				selectYear: '',//选择的年
				selectMonth: '',// 选择的月份
				selectDate: '',// 选择的日期
				selectDay:'',// 选择的星期
			}
		},
		props: ['classData'],
		created() {
			this.getNow()
			// setTimeout(()=> {
			// 	this.backToday()
			// },200)
			
		},
		watch: {
			showMonth(newValue,oldValue) {
				this.getDays(this.showYear,this.showMonth)
			},
			selectDate(newDate,oldValue) {
				this.getWeek()
			},
			classData(newValue,oldValue) {
				// this.backToday()
			}
		},
		methods: {
			// 获取当前时间
			getNow() {
				var date = new Date()
				this.showYear = date.getFullYear()
				this.showMonth = date.getMonth()+1
				this.selectYear = date.getFullYear()
				this.selectMonth = date.getMonth()+1
				this.selectDate = date.getDate()
				this.getWeek(date.getDay())
				
			},
			// 获取星期
			getWeek() {
				var day = new Date(this.selectYear,this.selectMonth-1,this.selectDate).getDay()
				switch(day) {
					case 0: this.selectDay = '星期日';break;
					case 1: this.selectDay = '星期一';break;
					case 2: this.selectDay = '星期二';break;
					case 3: this.selectDay = '星期三';break;
					case 4: this.selectDay = '星期四';break;
					case 5: this.selectDay = '星期五';break;
					case 6: this.selectDay = '星期六';break;
				}
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
			
			//判断选中的日期
            checkSelected(){
				if(this.showYear === this.selectYear && this.showMonth === this.selectMonth) {
					var index = this.days.findIndex(item => {return (item.class === 'font-scD1' && item.value === this.selectDate)})
					if(index !== -1) {
						this.days[index].class='font-scD3 bgc-mcA3'
					}
				}
			},
			// 点击日期
			clickDay(index) {
				var now = new Date()
				if(this.days[index].class === 'font-scD1') {
					if(this.selectYear === this.showYear && this.selectMonth === this.showMonth) {
						var idx = this.days.findIndex(item => {return item.class === 'font-scD3 bgc-mcA3' || item.class=== 'font-scD3 bgc-scD1'})
						if(idx !== -1) {
							this.days[idx].class="font-scD1"
						}
					}
					this.selectYear = this.showYear
					this.selectMonth = this.showMonth
					this.selectDate = this.days[index].value
					if(this.selectMonth === now.getMonth()+1 && this.selectYear === now.getFullYear() && this.selectDate === now.getDate()) {
						this.days[index].class="font-scD3 bgc-mcA3"
					}else {
						this.days[index].class="font-scD3 bgc-scD1"
					}
					
					// this.getDays(this.showYear,this.showMonth)
				}
				
				this.$emit('jbClick',this.selectYear+'-'+this.checkNum(this.selectMonth)+'-'+this.checkNum(this.selectDate))
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
			// 判断某日期是否有课
			isClass(index) {
				var key = this.showYear+'-'+this.checkNum(this.showMonth)
				
				if(this.classData[key] && this.days[index].class !== "font-scD2") {
					var idx = this.classData[key].findIndex(item => {return item === this.days[index].value})
					if(idx === -1) {
						return false
					}else {
						return true
					}
				}else {
					return false
				}
				
			},
			checkNum(num) {
				if(num < 10) {
					return '0'+num
				}else {
					return num
				}
			},
			// 返回今天
			backToday() {
				this.getNow();
				this.getDays(this.showYear,this.showMonth)
				this.$emit('jbClick',this.selectYear+'-'+this.checkNum(this.selectMonth)+'-'+this.checkNum(this.selectDate))
			}
			
		}
	}
</script>

<style scoped lang="less">
	.wrap {
		position: relative;
		width: 227upx;
		height: 392upx;
	}
	.yearMonth {
		box-sizing: border-box;
		padding: 25upx 61upx 19upx;
		width: 100%;
		.icon-left {
			cursor: pointer;
		}
		.icon-right {
			cursor: pointer;
			transform: rotate(180deg);
		}
	}
	.date {
		margin: 3upx 0 6upx;
	}
	.dateBox {
		box-sizing: border-box;
		padding: 0 6upx;
		width: 100%;
		.weeks,.days {
			width: 100%
		}
		.week {
			width: 24upx;
			margin: 30upx 0 3upx 6upx;
		}
		.day {
			position: relative;
			cursor: pointer;
			width: 24upx;
			height: 24upx;
			border-radius: 50%;
			line-height: 24upx;
			text-align: center;
			margin: 4upx 0 0 6upx;
			.dot {
				position: absolute;
				bottom: 3upx;
				// left: 50%;
				// margin-left: -1.5upx;
				width: 3upx;
				height: 3upx;
				border-radius: 50%;
			}
		}
	}
	.backToday {
		position: absolute;
		left: 18upx;
		bottom: 46upx;
		box-sizing: border-box;
		width:46upx;
		height:19upx;
		border-radius:9upx;
		border-style: solid;
		border-width: 1upx;
		line-height: 18upx;
		cursor: pointer;
	}
</style>
