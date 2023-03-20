<script>
  export default {
    data() {
      return {
        events: [
          {
            day: 26,
            month: 3,
            year: 2023,
            timeFrom: '14:00',
            timeTo: '15:00',
            timeZone: 'Europe/Kyiv',
            saved: true
          }
        ],
        selectedMonth: +(new Date().toLocaleString('en-GB', { timeZone: 'Europe/London' }).slice(3, 5)) - 1,
        selectedYear: +(new Date().toLocaleString('en-GB', { timeZone: 'Europe/London' }).slice(6, 10)),
        selectedEvent: undefined,
        monthList: ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"]
      }
    },
    computed: {
      days() {
        return +(new Date(this.selectedYear, this.selectedMonth + 1, 1).toLocaleString('en-GB', { timeZone: 'Europe/London' }).slice(0, 2))
      },
      dayOfWeek() {
        return new Date(this.selectedYear, this.selectedMonth, 1).getDay()
      }
    },
    methods: {
      changeMonth(isForward) {
        if (!isForward) {
          if (this.selectedMonth) {
            this.selectedMonth = this.selectedMonth - 1;
          } else {
            this.selectedMonth = 11;
            this.selectedYear = this.selectedYear - 1;
          }
        } else {
          if (this.selectedMonth !== 11) {
            this.selectedMonth = this.selectedMonth + 1;
          } else {
            this.selectedMonth = 0;
            this.selectedYear = this.selectedYear + 1;
          }
        }
      },
      handleClose() {
        if (!this.events[this.selectedEvent].saved) {
          this.events.splice(this.selectedEvent, 1);
        }

        this.selectedEvent = undefined;
      },
      handleSave() {
        this.events[this.selectedEvent].saved = true;
        this.selectedEvent = undefined;
      },
      handleDelete() {
        this.events.splice(this.selectedEvent, 1);
        this.selectedEvent = undefined;
      },
      handleSelect(day) {
        if (this.events.some(event => event.day === day && event.month === this.selectedMonth + 1 && event.year === this.selectedYear)) {
          this.selectedEvent = this.events.findIndex(event => event.day === day && event.month === this.selectedMonth + 1 && event.year === this.selectedYear)
        } else {
          this.events.push({
            day,
            month: this.selectedMonth + 1,
            year: this.selectedYear,
            timeFrom: '00:00',
            timeTo: '00:00',
            timeZone: Intl.DateTimeFormat().resolvedOptions().timeZone,
            saved: false
          });
          this.selectedEvent = this.events.length - 1;
        }
      }
    }
  }
</script>

<template>
  <div class="calendar">
    <div class="calendar__top">
      <button
        class="calendar__button"
        @click="changeMonth(false)"
      >
        &lt;
      </button>
      {{monthList[selectedMonth]}}
      {{selectedYear}}
      <button
        class="calendar__button"
        @click="changeMonth(true)"
      >
        &gt;
      </button>
    </div>

    <div class="calendar__middle">
      <div class="calendar__row">
        <div class="calendar__cell">
          Monday
        </div>

        <div class="calendar__cell">
          Tuesday
        </div>

        <div class="calendar__cell">
          Wednesday
        </div>

        <div class="calendar__cell">
          Thursday
        </div>

        <div class="calendar__cell">
          Friday
        </div>

        <div class="calendar__cell">
          Saturday
        </div>

        <div class="calendar__cell">
          Sunday
        </div>
      </div>

      <div class="calendar__row">
        <div
          class="calendar__cell"
          v-for="blank in dayOfWeek ? dayOfWeek - 1 : 6"
          v-bind:key="blank"
        >
        </div>

        <div
          class="calendar__cell"
          :class="{
            'calendar__cell--current': +(new Date().toLocaleString('en-GB', { timeZone: 'Europe/London' }).slice(0, 2)) === day
              && +(new Date().toLocaleString('en-GB', { timeZone: 'Europe/London' }).slice(3, 5)) - 1 === selectedMonth
              && +(new Date().toLocaleString('en-GB', { timeZone: 'Europe/London' }).slice(6, 10)) === selectedYear,
            'calendar__cell--event': events.some(test => test.day === day && test.month === selectedMonth + 1 && test.year === selectedYear)
          }"
          v-for="day in days"
          v-bind:key="day"
          v-on:click="handleSelect(day)"
        >
          {{day}}
        </div>
      </div>
    </div>

    <div class="modal" v-if="selectedEvent !== undefined">
      <div>
        <label>
          Day:
          <input type="number" min="1" :max="days" v-model="events[selectedEvent].day">
        </label>

        <label>
          Month:
          <select v-model="events[selectedEvent].month">
            <option v-for="(month, index) in monthList" v-bind:key="month" :value="index + 1">
              {{month}}
            </option>
          </select>
        </label>

        <label>
          Year:
          <input type="number" v-model="events[selectedEvent].year">
        </label>
      </div>

      <div>
        <label>
          Time:
          <input type="time" v-model="events[selectedEvent].timeFrom">
          -
          <input type="time" v-model="events[selectedEvent].timeTo">
        </label>

        <label>
          Time zone:
          <input type="text" v-model="events[selectedEvent].timeZone">
        </label>
      </div>

      <div class="modal__buttons">
        <button class="modal__button--green" v-on:click="handleSave()">Save</button>

        <button class="modal__button--red" v-on:click="handleDelete()">Delete</button>
      </div>

      <button v-on:click="handleClose()">Close</button>
    </div>
  </div>
</template>

<style>
  .calendar {
    position: relative;
    margin: auto;
    width: 700px;
    background-color: #eeeeee;
    box-shadow: 5px 5px #afafaf;
  }

  .calendar__top {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    width: 100%;
    height: 75px;
    background-color: #f6f6f6;
    border-bottom: 2px solid #afafaf;
    font-size: 36px;
    box-sizing: border-box;
  }

  .calendar__row {
    display: flex;
    flex-wrap: wrap;
  }

  .calendar__cell {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100px;
    height: 100px;
    font-size: 20px;
  }

  .calendar__cell--current {
    background-color: #dfdfdf;
  }

  .calendar__cell--event {
    background-color: #bffdb9;
  }

  .calendar__button {
    margin: 0 25px;
    padding: 15px;
  }

  .modal {
    position: absolute;
    display: flex;
    top: calc(50% - 150px);
    left: calc(50% - 283px);
    flex-direction: column;
    flex-wrap: wrap;
    justify-content: space-around;
    padding: 50px;
    width: 566px;
    height: 300px;
    background: #f7f7f7;
    box-shadow: 5px 5px #afafaf;
    box-sizing: border-box;
  }

  .modal__buttons {
    display: flex;
    justify-content: space-between;
    margin: 0 auto;
    width: 150px;
  }

  .modal__button--green {
    background-color: rgb(160, 240, 40);
  }

  .modal__button--red {
    background-color: rgb(238, 0, 0);
  }
</style>
