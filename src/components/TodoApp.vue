<template>
  <div class="container">
    <h2 class="text-center mt-5">My Vue TodoApp</h2>

    <div class="d-flex">
      <input type="text" v-model="task" placeholder="Enter task" class="w-100 form-control">
      <button @click="submitTask" class="btn btn-warning rounded-0">SUBMIT</button>
    </div>

    <table class="table table-bordered mt-5">
      <thead>
        <tr>
          <th scope="col">Task</th>
          <th scope="col">Status</th>
          <th scope="col" class="text-center">#</th>
          <th scope="col" class="text-center">#</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(task, index) in tasks" :key="index.id">
          <td>
            <span :class="{ 'line-through': task.status === 'finished' }">
              {{ task.name }}
            </span>
          </td>

          <td style="width: 120px;">
            <span
              class="pointer noselect"
              @click="changeStatus(index)"
              :class="{
                'text-danger': task.status === 'to-do',
                'text-success': task.status === 'finished',
                'text-warning': task.status === 'in-progress',
              }"
            >
              {{ capitalizeFirstChar(task.status) }}
            </span>
          </td>
          <td class="text-center">
            <div @click="deleteTask(index)">
              <span class="fa fa-trash pointer"></span>
            </div>
          </td>
          <td class="text-center">
            <div @click="editTask(index)">
              <p class="fa fa-pen pointer"></p>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      task: '',
      editedTask: null,
      statuses: ['to-do', 'in-progress', 'finished'],
      tasks: [],
    };
  },
  created() {
    this.fetchTasks();
  },
  methods: {
    capitalizeFirstChar(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    },
    changeStatus(index) {
      let newIndex = this.statuses.indexOf(this.tasks[index].status);
      if (++newIndex > 2) newIndex = 0;
      this.tasks[index].status = this.statuses[newIndex];
      this.updateTask(this.tasks[index]);
    },
    deleteTask(index) {
      const taskId = this.tasks[index].id;
      axios.delete(`http://localhost:3000/tasks/${taskId}`)
        .then(() => {
          this.tasks.splice(index, 1);
        })
        .catch(error => {
          console.error(error);
        });
    },
    editTask(index) {
      this.task = this.tasks[index].name;
      this.editedTask = index;
    },
    submitTask() {
      if (this.task.length === 0) return;
      // add new task to the list of tasks
      if (this.editedTask != null) {
        this.tasks[this.editedTask].name = this.task;
        this.editedTask = null;
      } 

       else {
        axios.post('http://localhost:3000/tasks', {
          name: this.task,
           status: 'to-do',
          })
          .then(response => {
            this.tasks.push(response.data);
          })
          .catch(error => {
            console.error(error);
          });
      }

      this.task = '';
    },
    fetchTasks() {
      axios.get('http://localhost:3000/tasks')
        .then(response => {
          this.tasks = response.data;
        })
        .catch(error => {
          console.error(error);
        });
    },
    updateTask(task) {
      axios.put(`http://localhost:3000/tasks/${task.id}`, task)
        .then(() => {
          console.log('Task updated:', task);
        })
        .catch(error => {
          console.error(error);
        });
    },
  },
};
</script>

<style scoped>
.line-through {
  text-decoration: line-through;
}
.pointer {
  cursor: pointer;
}
</style>
