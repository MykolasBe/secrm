<template>
  <div>
    <h3>Import tasks</h3>
    <button v-on:click.stop="addTasks()"> Import! </button>

    <h3>Import time logs</h3>
    <button v-on:click.stop="addTime()"> Import! </button>
  </div>
</template>

<script>
// Dummy seed Hasura DB to have some data
// Needs to be Hasura actions/events to fetch data
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
    "task_time_estimate": task["estimated-minutes"] ,
    "created_at": task["created-on"],
    "parent_id": parseInt(task.parentTaskId),
    "creator_name": task["creator-firstname"] + ' ' + task["creator-lastname"]
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
    "time_normal": 'hours: ' + timeLog.hours + ' minutes: ' + timeLog.minutes,
    "billed": timeLog.isbilled === "1",
    "parent_task_id": parseInt(timeLog["parentTaskId"])
  }
})

export default {
  methods: {
    addTasks() {
      this.$apollo.mutate({
        mutation: taskMutation,
        variables: { objects: tasks } ,
      })
    },
    addTime() {
      this.$apollo.mutate({
        mutation: timeMutation,
        variables: { objects: times },
      })
    }
  }
}
</script>

