<template>
	<div class="calendar">
		<div class="calendar_content">
			<div class="todayheader">
				<div>
					<em @click="selectYears('preYear')"><i class="icon left"></i></em>
					<span @click="showlist('isshowyear')">{{year}}</span>
					<em @click="selectYears('nextYear')"><i class="icon right"></i></em>
					<p v-if="liststatus.isshowyear">
						<ul class="list" v-animate="ulscroll" data-type="year">
							<li @click="selectYears(item)" v-for="item in yearList">{{item}}</li>
						</ul>
					</p>
				</div>
				<div>
					<em @click="selectMonth('preMonth')"><i class="icon left"></i></em>
					<span @click="showlist('isshowmoth')">{{monthArray[lang][month]}}</span>
					<em @click="selectMonth('nextMonth')"><i class="icon right"></i></em>
					<p v-if="liststatus.isshowmoth">
						<ul class="list" v-animate="ulscroll" data-type="month">
							<li @click="selectMonth(item)" v-for="item in monthList">{{monthArray[lang][item]}}</li>
						</ul>
					</p>
				</div>
			</div>
			<div class="weekul">
				<ul>
					<li v-for="item in weekdays[lang]">{{item}}</li>
				</ul>
			</div>
			<div class="daylist">
				<p @click="userselect(index)" v-for="(item ,index) in days" :class="item.istoday ? 'inmonth on' : item.inMonth ? 'inmonth' : ''"><span>{{item.values}}</span></p>
			</div>
			<div class="userpanel">
				<p @click="settoday">今天</p>
				<p @click="confirm">确认</p>
			</div>
		</div>
		<div class="calendar_hms" v-if="needms">
			<ul class="hms">
				<li><em><i :class="liststatus.isshowhour ? 'icon top' : 'icon bottom'"></i></em></li>
				<li class="showhms">
					<p class="bor">
						<span @click="showlist('isshowhour')">{{hour}} 时</span>
						<ul v-if="liststatus.isshowhour" class="list" v-animate="ulscroll" data-type="hour">
							<li @click="selectHour(item)" v-for="(item ,index) in hourList">{{item}}</li>
						</ul>
					</p>
					<p>
						<span @click="showlist('isshowminute')">{{minute}} 分</span>
						<ul v-if="liststatus.isshowminute" class="list" v-animate="ulscroll" data-type="minute">
							<li @click="selectMinute(item)" v-for="(item ,index) in minuteList">{{item}}</li>
						</ul>
					</p>
				</li>
				<li><em><i :class="liststatus.isshowminute ? 'icon top' : 'icon bottom'"></i></em></li>
			</ul>
		</div>
	</div>
</template>
<script>
	import moment from "moment"

	export default {
		name: 'calendar',
		props : ['userNeedHm' ,'userTime' ,'userFormat' ,'userLang'],
		data () {
			return {
				weekdays : {
					'en' : ['Mon' ,'Tues' ,'Wed' ,'Thur' ,'Fri' ,'Sat' ,'Sun'],
					'zh' : ['一' ,'二' ,'三' ,'四' ,'五' ,'六' ,'日']
				},
				monthArray : {
					'en' : ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
					'zh' : ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12']
				},
				lang : this.userLang ? this.userLang : 'zh',
				format : this.userFormat || 'YYYY-MM-DD HH:mm',
				needms : this.userNeedHm==0 ? false : true,
				days : [],
				dayIndex : '',
				moment : moment(this.userTime),
				year : moment(this.userTime).year(),
				month : moment(this.userTime).month(),
				day : moment(this.userTime).date(),
				hour : moment(this.userTime).hour(),
				minute : moment(this.userTime).minute(),
				yearList : [],
				monthList : [],
				hourList : [],
				minuteList : [],
				showday : {
					year : moment(this.userTime).year(),
					month : moment(this.userTime).month(),
					day : moment(this.userTime).date(),
					hour : moment(this.userTime).hour(),
          			minute : moment(this.userTime).minute()
				},
				liststatus : {
					isshowyear : false,
					isshowmoth : false,
					isshowhour : false,
					isshowminute : false
				}
			}
		},
		mounted (){
			this.init();
			this.setyearsArray();
			this.setmonthArray();
			this.sethourArray();
			this.setminuteArray();
		},
		methods : {
			init (){
				let days = [] ,predays = [] ,nextdays = [];
				this.dayIndex = 0;

				this.year = this.moment.year();
				this.month = this.moment.month();

				let momentNumber = this.moment.daysInMonth();
				for(let i=1; i<=momentNumber; i++){
					days.push({
						inMonth : null,
						values : i,
						istoday : null,
						year : this.moment.year(),
						month : this.moment.month()
					});
				}
				days = this.checkdays(days);

				let firstDayWeek = this.moment.date(1).day(),
					nextMonth = moment(this.moment).add(1,'months'),
					preMonth = moment(this.moment).subtract(1,'months'),
					preMonthNumber = preMonth.daysInMonth();

				if(firstDayWeek==0) firstDayWeek = 7;

				for(let i=0; i<firstDayWeek-1 ;i++){
					predays.push({
						inMonth : false,
						values : preMonthNumber - i,
						istoday : false,
						year : preMonth.year(),
						month : preMonth.month()
					});
				}
				predays.reverse();

				for(let i=1;i<=42-predays.length-days.length;i++){
					nextdays.push({
						inMonth : false,
						values : i,
						istoday : false,
						year : nextMonth.year(),
						month : nextMonth.month()
					});
				}
				this.days = [...predays ,...days ,...nextdays];
				this.dayIndex = (predays.length ? predays.length - 1 : predays.length) * 1 + this.dayIndex;
			},
			isNumber(n){
				return !isNaN(parseFloat(n)) && isFinite(n);
			},
			showlist (arg){
				for(let item in this.liststatus){
					item==arg && !this.liststatus[item] ? this.$set(this.liststatus ,arg , true) : this.$set(this.liststatus ,item , false);
				}
			},
			selectMonth (selectMonth){
				if(this.isNumber(selectMonth)){
					this.moment.month(selectMonth);
				}else{
					selectMonth=='preMonth' ? this.moment.subtract(1,'months') : this.moment.add(1,'months');
				}
				this.month = this.moment.month();
				this.setmonthArray();
				this.showlist();
				this.init();
			},
			selectYears (selectYears){
				if(this.isNumber(selectYears)){
					this.moment.year(selectYears);
				}else{
					selectYears=='preYear' ? this.moment.subtract(1 ,'years') : this.moment.add(1 ,'years');
				}
				this.year = this.moment.year();
				this.setyearsArray();
				this.showlist();
				this.init();
			},
			selectHour (hour){
				this.hour = hour;
				this.$set(this.showday ,'hour' ,hour);
				this.sethourArray();
				this.showlist();
			},
			selectMinute (minute){
				this.minute = minute;
				this.$set(this.showday ,'minute' ,minute);
				this.setminuteArray();
				this.showlist();
			},
			checkdays (days){
				days.map((item)=>{
					if(item.year==this.year && item.month==this.month){
						item.inMonth = true;
						if(this.showday.year==item.year && this.showday.month==item.month && item.values==this.showday.day){
							item.istoday = true;
							this.dayIndex = item.values;
						}else{
							item.istoday = false;
						}
					}else{
						item.istoday = false;
						item.inMonth = false;
					}
				});
				return days;
			},
			setyearsArray (useryear = this.year){
				let newYear = [];
				for(let i=useryear-2;i<=useryear+2;i++){
					newYear.push(i);
				}
				this.yearList = newYear;
			},
			setmonthArray (usermonth = this.month){
				let newMonth = [];
				for(let i=usermonth-2;i<=usermonth+2;i++){
					if(i>=0 && i<this.monthArray[this.lang].length){
						newMonth.push(i);
					}else if(i<0){
						if(Math.abs(i)>this.monthArray[this.lang].length){
							newMonth.push(i + Math.ceil(Math.abs(i)/this.monthArray[this.lang].length) * this.monthArray[this.lang].length);
						}else{
							newMonth.push(this.monthArray[this.lang].length + i);
						}
					}else{
						newMonth.push(i - Math.floor(i/this.monthArray[this.lang].length) * this.monthArray[this.lang].length);
					}
				}
				this.monthList = newMonth;
			},
			sethourArray (userhour = this.hour){
				let newHour = [];
				for(let i=userhour-2;i<=userhour+2;i++){
					if(i>23){
						newHour.push(i % 24);
					}else if(i<0){
						!(i%24) ? newHour.push(i % 24) : newHour.push(24 + i % 24);
					}else{
						newHour.push(i);
					}
				}
				this.hourList = newHour;
			},
			setminuteArray (userminute = this.minute){
				let newMinute = [];
				for(let i=userminute-2;i<=userminute+2;i++){
					if(i>60){
						newMinute.push(i % 61);
					}else if(i<0){
						!(i%61) ? newMinute.push(i % 61) : newMinute.push(61 + i % 61);
					}else{
						newMinute.push(i);
					}
				}
				this.minuteList = newMinute;
			},
			checkNumber (number){
				return	number.toString().length==1 ? `0${number}` : number;
			},
			ulscroll (type ,direction){
				switch (type){
					case 'year' :
						this.setyearsArray(this.year+direction);
						break;
					case 'month' :
						this.setmonthArray(this.month+direction);
						break;
					case 'hour' :
						this.sethourArray(this.hour+direction);
						break;
					case 'minute' : 
						this.setminuteArray(this.minute+direction);
						break;
				}
			},
			userselect (index){
				if(!this.days[index].inMonth || this.days[index].istoday) return;
				this.$set(this.days ,index ,Object.assign({} ,this.days[index] ,{istoday : true}));
				this.$set(this.days ,this.dayIndex ,Object.assign({} ,this.days[this.dayIndex] ,{istoday : false}));
				this.dayIndex = index;

				this.showlist();
				this.showday = Object.assign({} ,this.showday ,{
					year : this.year,
					month : this.month,
					day : this.days[index].values
				});
			},
			settoday (){
				this.moment = moment();

				this.year = this.moment.year();
				this.month = this.moment.month();
				this.day = this.moment.date();
				this.hour = this.moment.hour();
				this.minute = this.moment.minute();
				this.setyearsArray();
				this.setmonthArray();
				this.sethourArray();
				this.setminuteArray();

				this.showday = Object.assign({} ,this.showday ,{
					year : this.moment.year(),
					month : this.moment.month(),
					day : this.moment.date(),
					hour : this.moment.hour(),
					minute : this.moment.minute()
				});
				this.init();
			},
			confirm (){
				this.needms ? this.$emit('calendartime', moment({
					year : this.showday.year,
					month : this.checkNumber(this.showday.month),
					day : this.checkNumber(this.showday.day),
					hour : this.checkNumber(this.showday.hour),
					minute : this.checkNumber(this.showday.minute)
				}).format(this.format))
				: this.$emit('calendartime', moment({
					year : this.showday.year,
					month : this.checkNumber(this.showday.month),
					day : this.checkNumber(this.showday.day)
				}).format(this.format.split(' ')[0]));
			}
		},
		directives : {
			animate : {
				inserted(element, binding){
					let number = 0;
					const type = element.getAttribute('data-type'),
						fn = (e)=>{
							const direction = e.wheelDelta ? e.wheelDelta > 0 ? -1 : 1 : e.detail > 0 ? 1 : -1;
							number = number + direction;
							binding.value(type ,number);
						};
					element.$fn = fn;
					element.addEventListener("DOMMouseScroll" ,fn ,false);
					element.addEventListener("mousewheel" ,fn ,false);
				},
				unbind(element ,bind){
					element.removeEventListener("DOMMouseScroll" ,element.$fn ,false);
					element.removeEventListener("mousewheel" ,element.$fn ,false);
				}
			}
		}
	}
</script>

<style scoped>
	body,p,form,input,button,dl,dt,dd,ul,ol,li,h1,h2,h3,h4,h5,h6{
		margin : 0;
		padding : 0;
		box-sizing : border-box;
	}
	ul li{
		list-style : none;
	}
	.calendar{
		width : 340px;
		user-select : none;
	}
	.calendar_content{
		overflow : hidden;
		border-radius : 15px;
		background : #fff;
		box-shadow : 0 0 20px 0 #ededed;
	}
	i.icon{
		display : inline-block;
		background : url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAABGCAYAAADIOmTMAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyBpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMC1jMDYwIDYxLjEzNDc3NywgMjAxMC8wMi8xMi0xNzozMjowMCAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNSBXaW5kb3dzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjNDMjIwOUY1M0JBNzExRTg4NDcxODA5NDI3RjlCRTExIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjNDMjIwOUY2M0JBNzExRTg4NDcxODA5NDI3RjlCRTExIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6M0MyMjA5RjMzQkE3MTFFODg0NzE4MDk0MjdGOUJFMTEiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6M0MyMjA5RjQzQkE3MTFFODg0NzE4MDk0MjdGOUJFMTEiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz541xyWAAAEsElEQVR42rSYW2gcVRiAM7Obi3WrqKQtKGiwb5HNPUGaFgs2aUq91NYbtgiVooLgQ0EE0QcR+1Io9KFIfAhYFPpgsUVpo5EIDdhcNiaRaDApUbOFYqrWuCu7yW7i9y8z4XTYnT1npv3h9Pw7k/PNP+f8t6lVoSEjIyPvr62tvYuatyzrufb29rP89l1j+d1ksQX0JOob6wssazkSidQ3NzfP+a2N+kCjo6OjfagHPdffa2lpmQtk8fz8fM3i4uIZFj+pWLrKeL2tra3XeYAZeGZmZuPS0tJ5Fj6mQFeYDrG3ZxTL9cHDw8P3AbnIolYF+h/jAJZe8GyJL9h2lYmJifuZLnmg/9i23e2F6kjh8BKJxNZsNvsN6kPKvT8Y3a2trRMVAcQGGs/lckMqFEt/Z9rOngaCFizO5/PvMG9WL+KnB3GpXypCiF1dXX0YC79XL/KwPgJjSyhwQ0NDurKycg/6lHLiDzP6p6am7gkMln+amppuYHU3qhqm8Uwm89Xk5OSdgcEiHNS1qqqqXahXFcsfBX52enq6KjDYsfxXtqUL6/9ULnel0+nPeEgkMFiErPUTUw/jX8Xy/Rxmbyiwsy2juNxTqBnl8mHgx0OBRYi4QcL5edScYvlRUulbocAi5IjzwMXP15yIvM7vb0ODHfhpLH0TNQl0OxGZqLiVMjs7e5c3bfoNq1xeDRV5t1XGxsZ24k4npDLr/H25rYg6fUMbGe0c6kbcaRM3XsYDcmEMtcgDsVQqdQV9k5Lov6ytrX22rq4u42ex7x7X19enpCGRRkRZtJfyf9HrCUYWKxW6G7i0ThuU+wlCezdReN3YYlfp6OjoZ9oF/IZyv2V1dfUSOfmBwBa7Mj4+3rCystLvqYO/Sa4mrc4GaljW3592gMo9gPqg2g5IlXErt/ZWqCJNH0V2G+rPymXxmu9wx22hIq+xsfEqB7cDK8eU17+b8TV+vztUSDve8IzHFcVrPgwFXlhYuAOQ1Du1mM5Ho9G9xl6h5I5KXO0c0B4lIq+Rkzt5kyuBDo9FNtBPVCjyN9AugWp3m0U+Zk4xvaBYmuYg9+AtPwbOx0CPMb2qfswwngZ6OXCilwrM67+t9oeMF6l7A4FDGkuPAO1VLJXTeYVI6wuSNqNOZtvH9NFNT7Sso1jaF6rmcTAJtdME+gHQE6EqyO2q0uvZjD0eDJJ3RaS8sf4CQdXp7r/lpMhBfgt0rqamZiddftIESs2UzzWBptjWHlxzyKY6f+xARbbSaGtb7oGKxIjYT92Pm0OeTwQteBGoyF8STO7HTVJe3wReCkoueRxv+mHd3UzgOtCbQloHrgstmo8FIjCBKpfnOO0nnIMuCy24W7E9LAGXhBQpB3XBRRN9iW3RgpateSXgWtCyxbQIXAta9r/H1D3PZrOf4/yv6UBL7rFXlpeXa2Wmf1u8Zd8glKsm4n8AK9pN8oitA0W91zRJ2QZQoyRlG0DzJnBbEzpESD9ikgFtHWgsFuuhtZoxSa+2DtT5sjJKr7Yu1CS9FiLPBKqTXuWh8Xg8aeP4X5hC/Swn9E8XtgJ5Scq2CdQHnsR7jqjde6c0HFJ6gjQssi3S8EiP4iah/wUYAMy+LqgcPtUcAAAAAElFTkSuQmCC') no-repeat 0 0;
		background-size : 11px 35px;
	}
	i.icon.right{
		background-position : 0 -11px;
		width : 7px;
		height : 11px;
	}
	i.icon.left{
		background-position : 0 0;
		width : 7px;
		height : 11px;
	}
	i.icon.top{
		background-position : 0 -28.5px;
		width : 11px;
		height : 6px;
	}
	i.icon.bottom{
		background-position : 0 -23px;
		width : 11px;
		height : 6px;
	}
	.todayheader{
		display : flex;
		margin : 20px 25px 16px 25px;
		align-items : center;
		justify-content : space-between;
	}
	.todayheader div{
		display : inline-flex;
		justify-content : space-around;
		position : relative;
		align-items : center;
		align-content : center;
		border : 1px solid #f5f5f5;
		height : 28px;
		border-radius : 20px;
		width : 109px;
	}
	.todayheader div em{
		display : inline-flex;
		justify-content : space-around;
		align-items : center;
		align-content : center;
		border-radius : 100%;
		width : 18px;
		height : 18px;
		background : #f0f0f0;
		cursor : pointer;
	}
	.todayheader div span{
		display : inline-block;
		font-size : 16px;
		color : #333333;
		width : 35px;
		text-align : center;
		cursor : pointer;
	}
	.todayheader div p,.calendar_hms ul.list{
		overflow : hidden;
		position : absolute;
		top : 100%;
		left : 0;
		z-index : 10;
		border : 1px solid #f5f5f5;
		background : #fff;
		border-radius : 15px;
		width : 100%;
		padding : 5px 0;
	}
	ul.list li:nth-child(1),ul.list li:last-child{
		opacity : 0.3;
	}
	ul.list li:nth-child(2),ul.list li:nth-child(4){
		opacity : 0.7;
	}
	ul.list li{
    	list-style : none;
    	color : #333333;
    	text-align : center;
    	line-height : 25px;
    	height : 25px;
    	cursor : pointer;
	}
	.weekul ul,.daylist{
		margin : 0 10px 0 15px;
	}
	.weekul ul li{
		display : inline-block;
		width : 40px;
		text-align : center;
		font-size : 14px;
		color : #31dac3;
		margin-right : 5px;
	}
	.daylist p{
		display : inline-block;
		position : relative;
		width : 40px;
		font-size : 12px;
		text-align : center;
		color : #999999;
		margin : 12px 5px 0 0;
		padding : 2px 0;
	}
	.daylist p span{
		position : relative;
		z-index : 5;
	}
	.daylist p.on span{
		color : #fff;
	}
	.daylist p.on:before{
		display : block;
		position : absolute;
		top : 50%;
		left : 50%;
		transform : translate(-50%,-50%);
		content : '';
		background : #31dac3;
		height : 34px;
		width : 34px;
		border-radius : 100%;
		z-index : 4;
	}
	.daylist p.inmonth{
		cursor : pointer;
		color : #666666;
	}
	.userpanel{
		overflow : hidden;
		display : flex;
		justify-content : center;
		align-items : center;
		margin : 10px 30px 0 30px;
		border : 1px solid #31dac3;
		height : 30px;
		border-radius : 30px;
		font-size : 12px;
		margin-bottom : 10px;
	}
	.userpanel p{
		flex : 1;
		height : 30px;
		text-align : center;
		line-height : 30px;
		cursor : pointer;
	}
	.userpanel p:nth-child(1){
		background : #31dac3;
		color : #fff;
	}
	.userpanel p:nth-child(2){
		background : #ffffff;
		color : #31dac3;
	}
	.calendar_hms{
		background : #d2f3ef;
		box-shadow : 0 0 20px 0 #ededed;
		height : 41px;
		padding-top: 9px;
		border-radius : 15px;
	}
	.calendar_hms ul.hms{
		display : flex;
		justify-content : center;
		align-items : center;
		margin : 0 48px;
	}
	.calendar_hms ul.hms li em{
		display : flex;
		justify-content : space-around;
		align-items : center;
		align-content : center;
		border-radius : 100%;
		width : 18px;
		height : 18px;
		background : #f0f0f0;
		cursor : pointer;
	}
	.calendar_hms ul.hms li.showhms{
		margin : 0 18px;
		display : flex;
		justify-content : center;
		align-items : center;
		background : #ffffff;
		border : 1px solid #31dac3;
		width : 180px;
		height : 30px;
		border-radius : 100px;
	}
	.calendar_hms ul.hms li p{
		position : relative;
		flex : 1;
		cursor : pointer;
	}
	.calendar_hms ul.list{
		position : absolute;
		width : 100%;
		bottom : 100%;
		left : 0;
		top : auto;
	}
	.calendar_hms ul.hms li p span{
		display : block;
		height : 30px;
		line-height : 30px;
		text-align : center;
		font-size : 14px;
	}
	.calendar_hms ul li p.bor{
		border-right : 1px solid #31dac3;
	}
</style>