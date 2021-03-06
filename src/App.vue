<template>
  <div id="app">
    <h1>Study Scheduler</h1>
    <div class="container">
      <table id="input" class="sticky">
        <tr>
          <th>
            <label for="participant_count">
              Participant Count<br/>
              <span class="small">Include backups, exclude pilot</span>
            </label>
          </th>
          <td>
            <input type="number"
              v-model.number="participant_count"
              id="participant_count"
              min="1"
              max="999"
            >
          </td>
        </tr>
        <tr>
          <th>
            <label for="concurrent_sessions">
              Concurrent Sessions
            </label>
          </th>
          <td>
            <input type="number"
              v-model.number="concurrent_sessions"
              id="concurrent_sessions"
              min="1"
              max="999"
            >
          </td>
        </tr>
        <tr>
          <th>
            <label for="first_day">First Day</label>
          </th>
          <td>
            <datepicker v-model="first_day" format="D, M/d/yy" size="5"></datepicker>
          </td>
        </tr>
        <tr>
          <th>
            <label>Day Begin</label>
          </th>
          <td :class="{error: session_error}">
            <vue-timepicker
              :format="date_format"
              v-model="day_start"
              :minute-interval="minute_interval"
              hide-clear-button
            ></vue-timepicker>
          </td>
        </tr>
        <tr>
          <th>
            <label>Day End</label>
          </th>
          <td :class="{error: session_error}">
            <vue-timepicker
              :format="date_format"
              v-model="day_end"
              :minute-interval="minute_interval"
              hide-clear-button
            ></vue-timepicker>
          </td>
        </tr>
        <tr>
          <th>
            <label for="session_duration">Session Duration</label>
          </th>
          <td :class="{error: session_error}">
            <input type="number"
              v-model.number="session_duration"
              id="session_duration"
              min="0"
              max="999"
              :step="minute_interval"
            > minutes
          </td>
        </tr>
        <tr>
          <th>
            <label for="buffer_duration">Buffer Duration</label>
          </th>
          <td>
            <input type="number"
              v-model.number="buffer_duration"
              id="buffer_duration"
              min="0"
              max="999"
              :step="minute_interval"
            > minutes
          </td>
        </tr>
        <tr>
          <th>
            <label for="lunch_duration">Lunch Duration</label>
          </th>
          <td :class="{error: lunch_error}">
            <input type="number"
              v-model.number="lunch_duration"
              id="lunch_duration"
              min="0"
              max="999"
              :step="minute_interval"
            > minutes
          </td>
        </tr>
        <tr v-show="lunch_duration > 0">
          <th>
            <label>Lunch Begin (earliest)</label>
          </th>
          <td :class="{error: lunch_error}">
            <vue-timepicker
              :format="date_format"
              v-model="lunch_start"
              :minute-interval="minute_interval"
              hide-clear-button
            ></vue-timepicker>
          </td>
        </tr>
        <tr v-show="lunch_duration > 0">
          <th>
            <label>Lunch End (latest)</label>
          </th>
          <td :class="{error: lunch_error}">
            <vue-timepicker
              :format="date_format"
              v-model="lunch_end"
              :minute-interval="minute_interval"
              hide-clear-button
            ></vue-timepicker>
          </td>
        </tr>
        <tr v-show="lunch_duration > 0">
          <th>
            <label for="show_lunch">Show Lunch</label>
          </th>
          <td>
            <input type="checkbox" v-model="show_lunch" id="show_lunch">
          </td>
        </tr>
        <tr>
          <th>
            <label for="include_saturday">Include Saturday</label>
          </th>
          <td>
            <input type="checkbox" v-model="include_saturday" id="include_saturday">
          </td>
        </tr>
        <tr>
          <th>
            <label for="include_sunday">Include Sunday</label>
          </th>
          <td>
            <input type="checkbox" v-model="include_sunday" id="include_sunday">
          </td>
        </tr>
        <tr>
          <th>
            <label for="hour12">12-Hour Time</label>
          </th>
          <td>
            <input type="checkbox" v-model="hour12" id="hour12">
          </td>
        </tr>
        <tr>
          <th>
            <label for="separate_date_time">Separate Date and Time</label>
          </th>
          <td>
            <input type="checkbox" v-model="separate_date_time" id="separate_date_time">
          </td>
        </tr>
        <tr>
          <th>
            <label for="separate_start_end">Separate Start and End</label>
          </th>
          <td>
            <input type="checkbox" v-model="separate_start_end" id="separate_start_end">
          </td>
        </tr>
        <tr v-show="!separate_start_end">
          <th>
            <label for="separator">Time Separator</label>
          </th>
          <td>
            <input type="text" size="3" v-model="separator" id="separator">
          </td>
        </tr>
      </table>
      <div id="output">
        <div class="sticky">
          Sessions per Day: <strong>{{ sessions_per_day }}</strong>
          <span style="float: right">Total Days: <strong>{{ total_days }}</strong></span>
        </div>
        <table
          @mousedown="select_all"
          @mouseup="select_all"
          @touchstart="select_all"
          @touchend="select_all"
        >
          <tr>
            <td>PID</td>
            <td v-if="concurrent_sessions > 1">RID</td>
            <td>Day</td>
            <td v-if="separate_date_time">Date</td>
            <td>{{separate_start_end ? 'Start' : 'Time'}}</td>
            <td v-if="separate_start_end">End</td>
          </tr>
          <tr v-for="(p, i) in participants" :key="i">
            <td>{{ p.pid }}</td>
            <td v-if="concurrent_sessions > 1">{{ p.rid }}</td>
            <td>{{ dow(p.start) }}</td>
            <td v-if="separate_date_time">{{ formatDate(p.start) }}</td>
            <td v-if="separate_date_time">
              {{ formatTime(p.start) + (separate_start_end ? '' : separator + formatTime(p.end)) }}
            </td>
            <td v-if="!separate_date_time">
              {{ formatDate(p.start) }}
              {{ formatTime(p.start) + (separate_start_end ? '' : separator + formatTime(p.end)) }}
            </td>
            <td v-if="separate_start_end">
              {{ separate_date_time ? '' : formatDate(p.start) }}
              {{ formatTime(p.end) }}
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */

import VueTimepicker from 'vue2-timepicker'
import Datepicker from 'vuejs-datepicker'
import dm from 'date-arithmetic'

var m = 'minutes'
var d = 'day'

export default {
  name: 'app',
  data () {
    return {
      date_format: 'hh:mm a',
      minute_interval: 5,
      participant_count: 35,
      concurrent_sessions: 1,
      session_duration: 60,
      buffer_duration: 15,
      lunch_duration: 60,
      day_start: {
        hh: '09',
        mm: '00',
        a: 'am'
      },
      day_end: {
        hh: '05',
        mm: '00',
        a: 'pm'
      },
      first_day: new Date(),
      include_saturday: false,
      include_sunday: false,
      lunch_start: {
        hh: '11',
        mm: '00',
        a: 'am'
      },
      lunch_end: {
        hh: '01',
        mm: '00',
        a: 'pm'
      },
      show_lunch: true,
      hour12: true,
      separate_date_time: true,
      separate_start_end: true,
      separator: ' – ',
      has_lunch: false
    }
  },
  computed: {
    participants () {
      var participants = []
      var day = dm.startOf(this.first_day, d)
      var session = day
      var lunchAdded = false
      var p
      this.has_lunch = false
      for (var i = 0; i < this.participant_count; i++) {
        if (
          dm.add(session, this.session_duration, m) >
            dm.add(day, this.day_end_m, m)
        ) {
          day = dm.add(day, 1, d)
          lunchAdded = false
        }

        if (day.getDay() === 6 && !this.include_saturday) {
          day = dm.add(day, 1, d)
          session = dm.add(session, 1, d)
        }
        if (day.getDay() === 0 && !this.include_sunday) {
          day = dm.add(day, 1, d)
          session = dm.add(session, 1, d)
        }

        session = dm.max(session, dm.add(day, this.day_start_m, m))

        p = {
          pid: i + 1,
          start: session,
          end: dm.add(session, this.session_duration, m),
          rid: 1
        }
        participants.push(p)

        for (
          var j = 1;
          j < this.concurrent_sessions && i + 1 < this.participant_count;
          j++
        ) {
          i++
          participants.push({
            pid: i + 1,
            start: p.start,
            end: p.end,
            rid: j + 1
          })
        }

        if (
          this.lunch_duration && !lunchAdded &&
          p.end >= dm.add(day, this.lunch_start_m - this.buffer_duration, m) &&
          p.end <= dm.add(day, this.lunch_end_m - this.lunch_duration, m)
        ) {
          session = dm.add(p.end, this.lunch_duration, m)
          lunchAdded = true
          this.has_lunch = true
          if (this.show_lunch) {
            session = dm.max(p.end, dm.add(day, this.lunch_start_m, m))
            p = {
              pid: 'lunch',
              start: session,
              end: dm.add(session, this.lunch_duration, m),
              rid: ''
            }
            participants.push(p)
            session = p.end
          }
        } else {
          session = dm.add(p.end, this.buffer_duration, m)
        }

        if (day.getDate() !== session.getDate()) {
          lunchAdded = false
        }

        day = dm.startOf(session, d)
      }
      return participants
    },
    session_error () {
      return this.day_start_m > this.day_end_m + this.session_duration
    },
    lunch_error () {
      return this.lunch_duration > 0 && !this.has_lunch
    },
    sessions_per_day () {
      var count = 0
      while (
        count < this.participants.length &&
        this.participants[count].start.getDay() ===
          this.participants[0].start.getDay()
      ) {
        count++
      }
      return count - 1 * (this.show_lunch && this.has_lunch)
    },
    total_days () {
      return Math.ceil(this.participant_count / this.sessions_per_day)
    },
    day_start_m () {
      return this.to_minutes(this.day_start)
    },
    day_end_m () {
      return this.to_minutes(this.day_end)
    },
    lunch_start_m () {
      return this.to_minutes(this.lunch_start)
    },
    lunch_end_m () {
      return this.to_minutes(this.lunch_end)
    }
  },
  methods: {
    select_all () {
      var el = document.querySelector('#output table')
      var range = document.createRange()
      range.selectNodeContents(el)
      var selection = window.getSelection()
      selection.removeAllRanges()
      selection.addRange(range)
    },
    to_minutes (time) {
      var minutes = 1 * time.mm + 60 * time.hh
      if (time.a === 'pm' && time.hh !== '12') {
        minutes += 60 * 12
      } else if (time.a === 'am' && time.hh === '12') {
        minutes -= 60 * 12
      }
      return minutes
    },
    dow (date) {
      return date.toLocaleDateString([], { weekday: 'short' })
    },
    formatDate (date) {
      return date.toLocaleDateString(
        [], {
          year: 'numeric',
          month: 'numeric',
          day: 'numeric'
        }
      )
    },
    formatTime (time) {
      return time.toLocaleTimeString(
        [], {
          hour: 'numeric',
          minute: 'numeric',
          hour12: this.hour12
        }
      )
    }
  },
  components: {
    VueTimepicker,
    Datepicker
  }
}
</script>

<style lang="stylus">
#app
  font-family calibri, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  color #2c3e50

h1
  text-align center

.small
  font-size 0.8em
  font-weight normal

.sticky
  background #fff
  position sticky
  top 0

.container
  display flex
  flex-direction row
  justify-content space-evenly

td
  padding .3em .5em

table#output
  font-size 11pt
  min-width 5em
  border-collapse collapse
  td
    white-space nowrap
    vertical-align bottom

table#input
  flex-shrink 0
  th
    text-align left
    padding-right 0.5em

  th, td
    padding-bottom 1em
  td
    input
      font-size 1em
      padding .3em .5em
      border 1px solid #d2d2d2
    &.error input
      border-color #f00
      background rgba(#f00, 0.1)

.time-picker
  input.display-time
    height 1.7em
  .dropdown
    top 1.7em

.time-picker .dropdown,
.vdp-datepicker__calendar
  user-select none
  tap-highlight-color transparent

.vdp-datepicker
  input
    width 10em

.vdp-datepicker__calendar
  border 0 none !important
  box-shadow 0 1px 6px rgba(0,0,0,.15)
  .cell.selected,
  .cell.selected.highlighted,
  .cell.selected:hover
    background #41b883 !important
    color #fff
  .cell:not(.blank):not(.disabled).day:hover,
  .cell:not(.blank):not(.disabled).month:hover,
  .cell:not(.blank):not(.disabled).year:hover
    border-color transparent !important
    background rgba(0,0,0,.08)
</style>
