<template>
  <div>
    <div class="holiday" :style="{'background-color': bgColor}">
      <img id="loading-icon" alt="loading icon" src="https://samherbert.net/svg-loaders/svg-loaders/three-dots.svg" v-if="!holidayLoaded"/>
      <div>
        <p id="holiday-title" v-if="holidayLoaded">{{holidayTitle}}</p>
        <p id="holiday-date" v-if="holidayLoaded">{{holidayDateText}}</p>
        <p id="holiday-emoji" v-if="holidayLoaded">{{emoji}}</p>
      </div>
      <div id="btn-container">
        <input type="button" class="btn" @click="previousHoliday" :disabled="holidayIndex == 0" value="Previous" />
        <input type="button" class="btn" @click="nextHoliday" :disabled="holidayIndex == holidays.length - 1" value="Next" />
      </div>
    </div>
    <div id="btn-info" @click="showModal = !showModal">
      <img alt="info icon" src="https://image.flaticon.com/icons/svg/3/3716.svg" />
    </div>
    <div id="modal" v-if="showModal">
      <p>Good Friday, Easter Sunday, Easter Monday, Eid-Ul-Fitr, and Eid-Ul-Adha are not included in the holidays featured in this application.</p>
      <br>
      <p>This is because they usually occur on a different day each year.</p>
    </div>
  </div>
</template>

<script>
import { holidayMixin } from '../mixins/holidays';

export default {
	data() {
		return {
			holidayLoaded: false,
			nearestHoliday: null,
			holidayTitle: '',
			holidayDate: '',
			holidayDateText: '',
			emoji: '',
			bgColor: '#F48FB1',
			holidayIndex: 0,
			showModal: false
		};
	},
	mixins: [holidayMixin],
	created() {
		this.calculateNearestHoliday();
	},
	methods: {
		calculateNearestHoliday() {
			let todayMillis = new Date().getTime();

			let nearestHoliday = null;
			let nearestHolidayDate = null;
			let nearestHolidayMillis = Number.MAX_SAFE_INTEGER;
			let index = 0;

			this.holidays.forEach((holiday, holidayIndex) => {
				let holidayDate = this.getHolidayDate(holiday);
				let holidayDateMillis = holidayDate.getTime();
				let timeDifference = holidayDateMillis - todayMillis;

				if (
					todayMillis <= holidayDateMillis &&
					timeDifference < nearestHolidayMillis
				) {
					nearestHoliday = holiday;
					nearestHolidayMillis = timeDifference;
					nearestHolidayDate = holidayDate;
					index = holidayIndex;
				}
			});

			this.holidayLoaded = true;
			this.nearestHoliday = nearestHoliday;
			this.holidayDate = nearestHolidayDate;
			this.holidayIndex = index;
			console.log(this.holidayIndex);

			this.showNearestHoliday();
		},
		getHolidayDate(holiday) {
			let data = holiday.day.split('*');

			return new Date(
				new Date().getFullYear(),
				parseInt(data[1]) - 1,
				parseInt(data[0])
			);
		},
		showNearestHoliday() {
			this.holidayTitle = this.nearestHoliday.name.toUpperCase();
			this.holidayDateText = `${this.holidayDate.getDate()}/${this.holidayDate.getMonth() +
				1}/${this.holidayDate.getFullYear()}`;

			this.showIndicators();
		},
		showIndicators() {
			let now = new Date();

			now.setDate(new Date().getDate() + 14);
			let twoWeeks = now.getTime();

			now = new Date();
			now.setMonth(new Date().getMonth() + 1);
			let oneMonth = now.getTime();

			now.setMonth(new Date().getMonth() + 3);
			let threeMonths = now.getTime();

			now.setMonth(new Date().getMonth() + 6);
			let sixMonths = now.getTime();

			// Holiday is less than a month away
			if (twoWeeks >= this.holidayDate.getTime()) {
				this.emoji = '😁';
				this.bgColor = '#00796B';
			} else if (oneMonth >= this.holidayDate.getTime()) {
				this.emoji = '☹️';
				this.bgColor = '#FFD54F';
			} else if (threeMonths >= this.holidayDate.getTime()) {
				this.emoji = '😭';
				this.bgColor = '#F57C00';
			} else {
				this.bgColor = '#E53935';
				this.emoji = '🤬';
			}
		},
		previousHoliday() {
			if (this.holidayIndex - 1 >= 0) {
				this.nearestHoliday = this.holidays[--this.holidayIndex];
				this.holidayDate = this.getHolidayDate(this.nearestHoliday);

				this.showNearestHoliday();
			}
		},
		nextHoliday() {
			if (this.holidayIndex + 1 < this.holidays.length) {
				this.nearestHoliday = this.holidays[++this.holidayIndex];
				this.holidayDate = this.getHolidayDate(this.nearestHoliday);

				this.showNearestHoliday();
			}
		}
	}
};
</script>

<style>
.holiday {
	height: calc(100vh - 24px);
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	background-color: teal;
	color: rgba(0, 0, 0, 0.8);
	font-family: 'Playfair Display', sans-serif;
	text-align: center;
	padding-bottom: 24px;
	transition: 1s ease-out;
}

p {
	margin: 0;
	padding: 0;
}

#loading-icon {
	width: 48px;
}

#holiday-date {
	color: rgba(0, 0, 0, 0.54);
	font-size: 16px;
}

#holiday-title {
	font-size: 80px;
}

#holiday-emoji {
	font-size: 100px;
	margin-top: 40px;
}

.btn {
	background-color: rgba(0, 0, 0, 0.34);
	border: 2px solid rgb(0, 0, 0);
	color: rgba(0, 0, 0, 0.8);
	padding: 8px;
	width: 100px;
	border-radius: 30px;
	margin-left: 16px;
	transition: 1s;
	font-family: 'PlayFair Display', 'Arial';
}

.btn:disabled {
	background-color: gray;
	border-color: gray;
	color: rgba(0, 0, 0, 0.4);
	cursor: not-allowed;
}

.btn:disabled:hover {
	cursor: not-allowed;
	background-color: gray;
	color: rgba(0, 0, 0, 0.4);
}

.btn:hover {
	background-color: rgb(0, 0, 0);
	color: rgba(255, 255, 255, 0.8);
	cursor: pointer;
}

#btn-container {
	margin-top: 32px;
}

#btn-info {
	width: 56px;
	height: 56px;
	position: fixed;
	top: 90%;
	margin-left: 24px;
	border: 3px solid black;
	background-color: rgba(0, 0, 0, 0.34);
	border-radius: 50%;
	display: flex;
	justify-content: center;
	align-items: center;
	transition: 400ms ease-in;
}

#btn-info:hover {
	cursor: pointer;
	background-color: rgba(0, 0, 0, 0.5);
	transform: scale(0.95);
}

#btn-info > img {
	width: 30px;
}

#modal {
	position: fixed;
	top: calc(100vh - 320px);
	background-color: white;
	width: 60%;
	padding: 16px;
	border: 2px solid transparent;
	border-radius: 5px;
	font-family: 'PlayFair Display', 'Arial';
	box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.2);
	margin-left: 24px;
}

@media screen and (max-width: 568px) {
	#holiday-title {
		font-size: 40px;
	}

	#btn-info {
		top: 90%;
		width: 36px;
		height: 36px;
	}

	#modal p {
		font-size: 14px;
	}
}
</style>
