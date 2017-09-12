<template>
  <div id="app">

    <h2>TODO APP</h2>

    <ul id="example-1">
      <li v-for="todo in todos">
        {{ todo.todo }}

        <i class="material-icons">edit</i>
        <i class="material-icons">delete</i>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'app',
  data(){
      return {
          todos: []
      }
  },
  created() {
      axios.get('http://localhost:3000/todos')
          .then(response => this.todos = response.data)
          .catch(error => console.log(error));
  }
}
</script>