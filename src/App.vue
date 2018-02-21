<template>
  <div id="app">
    <h1>Study Scheduler</h1>
    <div class="container">
      <table id="input">
        <tr>
          <th>
            <label for="participant_count">
              Participant Count<br/>
              <span class="small">Include backups, exclude pilot</span>
            </label>
          </th>
          <td>
            <input type="number" v-model.number="participant_count" id="participant_count" min="1" max="999">
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
          <td>
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
          <td>
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
          <td>
            <input type="number" v-model.number="session_duration" id="session_duration" min="0" max="999" step="5"> minutes
          </td>
        </tr>
        <tr>
          <th>
            <label for="buffer_duration">Buffer Duration</label>
          </th>
          <td>
            <input type="number" v-model.number="buffer_duration" id="buffer_duration" min="0" max="999" step="5"> minutes
          </td>
        </tr>
        <tr>
          <th>
            <label for="lunch_duration">Lunch Duration</label>
          </th>
          <td>
            <input type="number" v-model.number="lunch_duration" id="lunch_duration" min="0" max="999" step="5"> minutes
          </td>
        </tr>
        <tr>
          <th>
            <label>Lunch Begin (earliest)</label>
          </th>
          <td>
            <vue-timepicker
              :format="date_format"
              v-model="lunch_start"
              :minute-interval="minute_interval"
              hide-clear-button
            ></vue-timepicker>
          </td>
        </tr>
        <tr>
          <th>
            <label>Lunch End (latest)</label>
          </th>
          <td>
              <vue-timepicker
              :format="date_format"
              v-model="lunch_end"
              :minute-interval="minute_interval"
              hide-clear-button
            ></vue-timepicker>
          </td>
        </tr>
        <tr>
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
          <th>Sessions per Day</th>
          <td>{{ sessions_per_day }}</td>
        </tr>
        <tr>
          <th>Total Days</th>
          <td>{{ total_days }}</td>
        </tr>
      </table>
      <table id="output"
        @mousedown="select_all"
        @mouseup="select_all"
        @touchstart="select_all"
        @touchend="select_all"
      >
        <tr>
          <td>PID</td>
          <td>Day</td>
          <td v-if="separate_date_time">Date</td>
          <td>Start</td>
          <td>End</td>
        </tr>
        <tr v-for="p in participants" :key="p.pid">
          <td>{{ p.pid }}</td>
          <td>{{ dow(p.start) }}</td>
          <td v-if="separate_date_time">{{ formatDate(p.start) }}</td>
          <td v-if="separate_date_time">{{ formatTime(p.start) }}</td>
          <td v-if="!separate_date_time">{{ formatDate(p.start) }} {{ formatTime(p.start) }}</td>
          <td>{{ separate_date_time ? '' : formatDate(p.start) }} {{ formatTime(p.end) }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
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
      show_lunch: false,
      hour12: true,
      separate_date_time: true
    }
  },
  computed: {
    participants () {
      var participants = []
      var day = dm.startOf(this.first_day, d)
      var session = day
      var lunchAdded = false
      var p
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
          end: dm.add(session, this.session_duration, m)
        }
        participants.push(p)

        if (
          this.lunch_duration && !lunchAdded &&
          p.end > dm.add(day, this.lunch_start_m, m) &&
          p.end <= dm.add(day, this.lunch_end_m, m)
        ) {
          session = dm.add(p.end, this.lunch_duration, m)
          lunchAdded = true
          if (this.show_lunch) {
            participants.push({
              pid: 'lunch',
              start: p.end,
              end: session
            })
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
    sessions_per_day () {
      var count = 0, lunch = 0
      while (
        this.participants[count].start.getDay() ===
          this.participants[0].start.getDay()
      ) {
        if (this.participants[count].pid === 'lunch') {
          lunch++
        }
        count++
      }
      return count - lunch
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
      var el = document.getElementById('output')
      var range = document.createRange()
      range.selectNodeContents(el)
      var selection = window.getSelection()
      selection.removeAllRanges()
      selection.addRange(range)
    },
    to_minutes (time) {
      var minutes = 1 * time.mm + 60 * time.hh
      if (time.a === 'pm' && time.hh !== 12) {
        minutes += 60 * 12
      } else if (time.a === 'am' && time.hh === 12) {
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
  position: sticky
  top: 1em
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
