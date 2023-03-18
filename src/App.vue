<template>
  <div id="app" class="calender" v-cloak>
    <h1 class="title">カレンダーUI</h1>
    <table class="table">
      <tr class="table-tr">
        <th class="table-th table-th-time"></th>
        <th
          v-for="th in tableHead"
          :key="th.index"
          class="table-th"
        >
          {{ getFormatedDate(th) }}
        </th>
      </tr>
      <tr
        v-for="(tr, index) in rows"
        :key="tr.index"
        class="table-tr"
      >
        <td class="table-td table-td-time">
          <p>{{ times[index] }}</p>
        </td>
        <td
          v-for="(cell, cellIndex) in tr.table_cells"
          :key="cell.index"
          class="table-td table-td-schedule"
        >
          <div v-for="(meet, meetIndex) in meetings[tableHead[cellIndex]]" :key="meet.index">
            <div
              v-if="getStartIndex(meetings[tableHead[cellIndex]][meetIndex].start) === index"
              class="meet-content"
              :style="getMeetStyle(meetings[tableHead[cellIndex]][meetIndex])"
            >
              <p class="meet-summary">{{meetings[tableHead[cellIndex]][meetIndex].summary}}</p>
            </div>
          </div>
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      message: 'Hello Vue!',
      meetings: [],
      workingHours: [],
      times: [],
      allTimes: ['0:00', '1:00', '2:00', '3:00', '4:00', '5:00', '6:00', '7:00', '8:00', '9:00', '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', '19:00', '20:00', '21:00', '22:00', '23:00'],
      rows: []
    }
  },
  computed: {
    tableHead () {
      return Object.keys(this.meetings)
    }
  },
  mounted () {
    this.fetchSchedules()
  },
  methods: {
    fetchSchedules () {
      this.axios.get('https://mixtend.github.io/schedule.json', {
        headers: {'User-Agent': 'Mixtend Coding Test'}
      })
        .then((response) => {
          this.meetings = response.data.meetings
          this.workingHours = response.data.working_hours
          const startIndex = this.allTimes.indexOf(this.workingHours.start)
          const endIndex = this.allTimes.indexOf(this.workingHours.end)
          this.times = this.allTimes.slice(startIndex, endIndex + 1)
          const rowObj = { 'table_cells': [ {}, {}, {} ] }
          this.times.forEach(() => {
            this.rows.push(rowObj)
          })
        })
        .catch(error => console.log(error))
    },
    getStartIndex (time) {
      const startTime = Number(time.slice(0, 2))
      return startTime - 10
    },
    getFormatedDate (dateVal) {
      const targetDate = new Date(dateVal)
      const month = String(targetDate.getMonth() + 1)
      const date = String(targetDate.getDate())
      const day = [ '日', '月', '火', '水', '木', '金', '土' ][targetDate.getDay()]
      return `${month}/${date}（${day}）`
    },
    getMeetStyle (meet) {
      if (!meet) return
      const tempDate = '2021-03-22 '
      const start = new Date(tempDate + meet.start).getTime()
      const end = new Date(tempDate + meet.end).getTime()
      const MeetHour = (end - start) / 3600000 // ミリ秒を時間に換算
      const tdHeight = 110
      const height = String(tdHeight * MeetHour) + 'px'
      const min = Number(meet.start.slice(-2))
      const top = min !== 0 ? String(min / 60 * tdHeight) + 'px' : '0'
      return `height: ${height}; top: ${top}`
    }
  }
}
</script>
<style>
html {
  font-family: "Noto Sans", sans-serif;
  color: #424242;
}
.calender {
  margin: 0 auto;
  max-width: 1085px;
}
.title {
  font-size: 24px;
  font-weight: normal;
  margin: 60px 0;
}
.table, .table-th, .table-td {
  border-collapse: collapse;
  border: 1px solid lightgray
}
.table {
  max-width: 1085px;
}
.table-th {
  height: 60px;
  font-size: 24px;
  font-weight: normal;
}
.table-td {
  height: 108px;
}
.table-th-time {
  width: 185px;
}
.table-td-time {
  font-size: 24px;
  text-align: center;
}
.table-td-schedule {
  position: relative;
  width: 300px;
}
.meet-content {
  position: absolute;
  background-color: #49b5a9;
  width: 100%;
}
.meet-summary {
  margin: 0;
  padding: 20px;
  font-size: 24px;
  color: #ffffff;
}
</style>
