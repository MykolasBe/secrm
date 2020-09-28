<template>
  <div>
    <div>
      <form
            @submit="onSubmit"
            @reset="onReset"
            method="post"
            v-if="show"
        >
            <label>
                <span>Search for:</span>
                <input type="text" v-model="form.search" required>
            </label>
            <label>
                <span>Search field:</span>
                <select v-model="form.searchField" required>
                    <option value="project">Project</option>
                    <option value="task_title">Task title</option>
                    <option value="creator_name">Creator name</option>
                </select>
            </label>
            <button type="submit">Search</button>
            <button type="reset">Reset</button>
        </form>
    </div>
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
        <tr v-for="task in output" :key="task.id">
          <td>{{ task.task_title }}</td>
          <td>{{ task.project }}</td>
          <td>100000$</td>
          <td>{{ spentTime(task, task.time_logs) }} / {{ Number(( estimateTime(task)).toFixed(2)) }} / {{ soldTime(task, task.time_logs) }}</td>
          <td>{{ remainingTime(estimateTime(task), spentTime(task, task.time_logs)) }}</td>
          <td>{{ $moment(task.created_at, "YYYY-MM-DD").fromNow() }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import tasksquery from '@/apollo/queries/fetchMainTask.gql'

export default {
  data() {
    return {
      form: {
          search: '',
          searchField: null,
      },
      output: this.$apollo.tasks,
      show: true,
    }
  },
  apollo: {
    tasks: {
      query: tasksquery
    },
  },
  methods: {
    spentTime:function (task, time_logs) {
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
      let estimate_time = task.task_time_estimate

      task.sub_task.map(function (sub_task){
        estimate_time += sub_task.task_time_estimate
      })

      return estimate_time / 60
    },

    remainingTime:function (estimated,spent){
      const remainingTime = estimated - spent

      return Number((remainingTime >= 0 ? remainingTime : 0).toFixed(2))
    },

    onSubmit(evt) {
      evt.preventDefault();
      let currentObj = this;
      
      // this.$apollo.mutate({
      //   mutation: taskMutation,
      //   variables: { objects: tasks } ,
      //   // prefetch: true,
      // })
      searchResult = this.$apollo.query({
        query:` 
          query {
            tasks(where: {parent_id: {_is_null: true}, ${this.form}: {_similar: "${this.form}"}}) {
            project
            parent_id
            created_at
            task_description
            task_time_estimate
            task_title
            time_log_parent_task {
              billed
              time_decimal
            }
            time_logs {
              time_decimal
              billed
            }
            sub_task {
              task_time_estimate
            }
          }
        }
      `
      })
     
      // return axios.post('jobs/searchJob', this.form)
      //     .then(function (response) {
      //         currentObj.output = response.data
      //     })
      //     .catch(error => {
      //         let er = error.response.data.errors;
      //         let ov = Object.values(er);
      //         alert(ov);
      //     })
    },
    onReset(evt) {
      evt.preventDefault();
      this.form.search = ''
      this.form.searchField = null
      this.show = false
      this.getAll();
      this.$nextTick(() => {
          this.show = true
      })
    },
    
    getAll() {
      let currentObj = this;

      return currentObj.output = this.tasks
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
