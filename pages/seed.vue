<template>
  <div>
    <h3>Import tasks</h3>
    <button v-on:click.stop="addTasks()"> Import! </button>

    <h3>Import time logs</h3>
    <button v-on:click.stop="addTime()"> Import! </button>
  </div>
</template>

<script>
import taskMutation from '~/apollo/queries/upsertTasks.gql'
import timeMutation from '~/apollo/queries/upsertTimes.gql'

import taskData from '~/static/DB seed/Tasks.json'
import timeData from "~/static/DB seed/Time.json"

const tasks = taskData["todo-items"].map(function(task){
  return {
    "id": task.id,
    "project": task["project-name"],
    "task_description": task.description,
    "task_title": task.content,
    "task_time_estimate": parseInt(task.estimatedTime)
  }
})

const times = timeData["time-entries"].map(function(timeLog){
  return {
    "id": parseInt(timeLog.id, 10),
    "project": timeLog["project-name"],
    "task_title": timeLog["todo-item-name"],
    "description": timeLog.description,
    "first_name": timeLog["person-first-name"],
    "last_name": timeLog["person-last-name"],
    "task_id": parseInt(timeLog["todo-item-id"], 10),
    "time_decimal": timeLog.hoursDecimal,
    "time_normal": 'hours: ' + timeLog.hours + ' minutes: ' + timeLog.minutes
  }
})

export default {
  methods: {
    addTasks() {
      this.$apollo.mutate({
        mutation: taskMutation,
        variables: { objects: tasks } ,
        // prefetch: true,
      })
    },
    addTime() {
      this.$apollo.mutate({
        mutation: timeMutation,
        variables: { objects: times },
        // prefetch: true,
      })
    }
  }
}
</script>

