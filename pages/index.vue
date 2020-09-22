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
          <td></td>
        </tr>
      </tbody>
    </table>
    <ul>
      <li v-for="item in tasks" :key="item.id">
        <nuxt-link :to="`/article/${item.id}`">
<!--          {{ item.task_title }}-->
        </nuxt-link>
      </li>
    </ul>
  </div>
</template>

<script>
import tasks from '~/apollo/queries/fetchTask.gql'
import times from '~/apollo/queries/fetchTime.gql'

export default {
  apollo: {
    tasks: {
      prefetch: true,
      query: tasks
    },
    times: {
      prefetch: true,
      query: times
    }
  },
  head: {
    title: 'Teamwork Tasks'
  }
}
</script>

<style>
ul {
  list-style-type: none;
  margin: 0;
  padding: 0;
  line-height: 1.6;
}
a {
  text-decoration: none;
  color: #3498DB;
}
a:hover {
  border-bottom: 1px solid;
}
</style>
