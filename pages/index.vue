<template>
  <div>
    <table>
      <thead>
        <th>Order</th>
        <th>Project</th>
        <th>Budget</th>
        <th>Spnt./Est./Sold</th>
        <th>Rem. h.</th>
        <th>Placed</th>
      </thead>
      <tbody>
        <tr v-for="task in tasks" :key="task.id">
          <td>{{ task.task_title }}</td>
          <td>{{ task.project }}</td>
          <td>100000$</td>
          <td>{{ totalTime(task, task.time_logs) }} / {{ Number(( estimateTime(task)).toFixed(1)) }} / {{ soldTime(task, task.time_logs) }}</td>
          <td>{{ Number(( estimateTime(task)).toFixed(1)) - soldTime(task, task.time_logs) }}</td>
          <td>{{ $moment(task.created_at, "YYYY-MM-DD").fromNow() }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import tasks from '@/apollo/queries/fetchMainTask.gql'
export default {
  apollo: {
    tasks: {
      query: tasks
    },
  },
  // time calc methods, needs to be ported to hasura computed fields
  methods: {
    totalTime:function (task, time_logs) {
      let time_ttl = 0
      time_logs.map(function (time_log){
        time_ttl += time_log.time_decimal
      })
      task.time_log_parent_task.map(function (time_log){
        time_ttl += time_log.time_decimal
      })
      return time_ttl
    },
    soldTime: function (task, time_logs) {
      let sold_time = 0
      time_logs.map(function (time_log){
        if (time_log.billed) {
          sold_time += time_log.time_decimal
        }
      })
      task.time_log_parent_task.map(function (time_log){
        if (time_log.billed) {
          sold_time += time_log.time_decimal
        }
      })
      return sold_time
    },
    estimateTime: function (task) {
      let estimate_time = 0
      task.sub_task.map(function (sub_task){
        estimate_time += sub_task.task_time_estimate
      })
      return estimate_time / 60
    }
  },
  head: {
    title: 'Teamwork Tasks'
  }
}
</script>

<style>
table {
  margin: 20px 20px;
}
thead {
  width: 100%;
  background: lightslategrey;
}
th{
  padding: 15px;
}
th:first-of-type{
  width: 45%;
}
td{
  padding: 15px;
}
</style>