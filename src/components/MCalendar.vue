<template>
  <div class="calendar">
    <table>
      <thead class="title">
        <tr>
          <td :colspan="calendar.weeks[0].length">
            <div class="titlebox">
              <span></span>
              <span>{{ calendar.title }}</span>
              <img crossorigin="anonymous" :src="calendar.logo">
            </div>
          </td>
        </tr>
      </thead>
      <thead class="days">
        <tr>
          <th v-for="(day, i) in calendar.weeks[0]" :key="i">
            {{ day.format('ddd') }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(week, i) in calendar.weeks" :key="i">
          <td
            v-for="(day, ii) in week"
            :key="ii"
            class="day"
            :class="{ 'day--other-month': day.month() !== calendar.activeMonth }"
          >
            <div class="day-container">
              <span class="date">
                {{ day.format('D') }}
              </span>
              <span
                v-if="day.event"
                class="event"
                :class="{
                  'event--start': day.event.start,
                  'event--ongoing': day.event.ongoing,
                  'event--end': day.event.end,
                  'event--single': day.event.single,
                }"
                :style="{ '--event-color': day.event.color, '--event-border': day.event.border }"
              >
                {{ day.event.text }}
              </span>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'MCalendar',
  props: {
    calendar: Object,
  },
};
</script>

<style scoped lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@700&display=swap');

.calendar {
  border: 1px solid transparent; // hover the border w/ firefox screenshot tool ;)
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #D6D7D8;
  background-color: #36393F;

  font-size: 10px;
  @media (min-width: 500px) {
    font-size: unset;
  }

  thead {
    &.title {
      font-family: 'Open Sans', sans-serif;
      font-size: x-large;
      color: #FFFFFF;

      img {
        max-height: 5em;
        width: auto;
      }

      .titlebox {
        display: grid;
        grid-template-columns: 1fr 2fr 1fr;
        grid-template-rows: 1fr;
        &>*+* {
          padding-left: 0.5em;
        }
      }
    }

    &.days {
      text-transform: uppercase;
      font-size: smaller;
      color: #A1A4A8;
      th, td {
        height: 2em;
        width: 6em;
      }
    }
  }

  tbody {
    th, td {
      height: 6em;
      width: 6em;
    }
  }

  table {
    border-collapse: collapse;
    thead.days, tbody {
      tr, td, th {
        border: 1px solid #41464D;
        padding: 0.5em;
      }
    }
  }

  .day {
    &--other-month {
      color: rgba(0,0,0,0);
    }
  }

  .day-container {
    position: relative;
    width: 100%;
    height: 100%;
  }
  .date {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    text-align: center;
  }
  .event {
    background-color: var(--event-color);
    border: 2px solid;
    border-color: var(--event-border);
    position: absolute;
    height: 1em;
    padding: 0.5em;

    top: calc(50% - 0.5em);
    bottom: 50%;

    &--start {
      text-align: left;
      border-radius: 1em 0 0 1em;
      border-right: 0;
      left: calc(50% - 1em);
      right: -0.55em;
    }
    &--ongoing {
      text-align: center;
      border-left: 0;
      border-right: 0;
      left: -0.55em;
      right: -0.55em;
    }
    &--end {
      text-align: right;
      border-left: 0;
      border-radius: 0 1em 1em 0;
      left: -0.55em;
      right: calc(50% - 1em);
    }
    &--single {
      text-align: center;
      border-radius: 1em;
      left: calc(50% - 2em);
      right: calc(50% - 2em);
    }
  }
}
</style>
