<template>
    <div id="app">
        <v-app>
            <v-layout row>
                <v-flex xs12 sm12 md12 lg6 offset-lg3>
                        <v-card>
                        <!--Main toolbar-->
                            <v-toolbar class="light-blue" extended light>
                            <v-toolbar-side-icon></v-toolbar-side-icon>
                              <v-btn
                                class="cyan accent-2"
                                fab
                                dark
                                absolute
                                bottom
                                right
                                @click.native.stop="dialogActions()"
                              >
                                <v-icon>add</v-icon>
                              </v-btn>
                            <v-toolbar-title slot="extension" class="white--text">My TODO's</v-toolbar-title>
                          </v-toolbar>
                        <!--Listing all todos-->
                            <v-list subheader>
                            <v-subheader>TODO items</v-subheader>
                                <v-list-tile avatar v-for="todo in todos" :key="todo._id">
                                  <v-list-tile-avatar>
                                    <i class="material-icons">work</i>
                                  </v-list-tile-avatar>
                                  <v-list-tile-content>
                                    <v-list-tile-title
                                            v-html="todo.todo"
                                            :class="{ completed: todo.done == 'true' }">
                                    </v-list-tile-title>
                                  </v-list-tile-content>
                                  <v-list-tile-action>
                                      <span>
                                          <v-btn v-if="todo.done" @click="undoTask(todo)" icon class="material-icons my-green">undo</v-btn>
                                          <v-btn v-if="!todo.done" @click="doneTask(todo)" icon class="material-icons my-green">done</v-btn>
                                          <v-btn @click.stop="editTask(todo)" icon class="material-icons my-orange">mode_edit</v-btn>
                                          <v-btn @click="deleteTask(todo)" icon class="material-icons my-red">delete</v-btn>
                                      </span>
                                  </v-list-tile-action>
                                </v-list-tile>
                          </v-list>
                        <!--Dialog part-->
                            <v-dialog v-model="dialog">
                          <v-card>
                           <v-card-text>
                               <v-text-field ref="inputTodo" v-model="addedTask" label="Add a task" @keyup.13="addTask(addedTask)"></v-text-field>
                              <small class="grey--text">* This will be saved to DB as well.</small>
                           </v-card-text>
                           <v-card-actions>
                             <v-spacer></v-spacer>
                             <v-btn flat primary @click.native="addTask(addedTask)">Submit</v-btn>
                           </v-card-actions>
                          </v-card>
                        </v-dialog>
                        <!--Edit Dialog part-->
                            <v-dialog v-model="editDialog">
                              <v-card>
                               <v-card-text>
                                   <v-text-field v-model="editedTask.todo" autofocus="" label="Edit task" @keyup.13="updateTask(editedTask)"></v-text-field>
                                  <small class="grey--text">* This will be saved to DB as well.</small>
                               </v-card-text>
                               <v-card-actions>
                                 <v-spacer></v-spacer>
                                 <v-btn flat primary @click.native="updateTask(editedTask)">Submit</v-btn>
                               </v-card-actions>
                              </v-card>
                        </v-dialog>
                        <!--Bottom navigation part-->
                            <v-bottom-nav absolute  class="transparent">
                          <v-btn flat class="teal--text" value="allTasks">
                            <span>All tasks</span>
                            <v-icon>history</v-icon>
                          </v-btn>
                          <v-btn flat class="teal--text" value="todoTasks">
                            <span>Todo tasks</span>
                            <i class="material-icons">work</i>
                          </v-btn>
                          <v-btn flat class="teal--text" value="doneTasks">
                            <span>Done tasks</span>
                            <i icon class="material-icons">done</i>
                          </v-btn>
                        </v-bottom-nav>
                        </v-card>
                  </v-flex>
                </v-layout>
            </v-app>
        </div>
  </template>

<style>
    .material-icons.my-orange { color: #E8B30C !important; }
    .material-icons.my-green { color: #02530D !important; }
    .material-icons.my-red { color: #FF0000 !important; }
    .completed{text-decoration: line-through;}
</style>

<script>
import axios from 'axios'
const serverURL = "https://simple-todo-crud.herokuapp.com/todos/";

export default {
    name: 'app',
  data(){
      return {
          filteredTodos: [],
          todos: [],
          addedTask: '',
          editedTask: {},
          editDialog:false,
          dialog: false,
      }
  },
  created() {
      axios.get(serverURL)
        .then(response => this.todos = response.data)
        .catch(error => console.log(error));
  },
  methods: {
      doneTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos[foundIndex].done = "true";
          axios.put( serverURL + todo._id, {done: todo.done})
              .then(response => console.log("Task DONE!"))
              .catch(error => console.log(error));
      },
      editTask: function (todo) {
        this.editedTask = todo;
        this.editDialog = !this.editDialog;
      },
      updateTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos[foundIndex].todo = todo.todo;
          this.editDialog = false;
          axios.put( serverURL + todo._id, {todo: todo.todo})
              .then(response => console.log("Task EDITED!"))
              .catch(error => console.log(error));
      },
      deleteTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos.splice(foundIndex, 1);
          axios.delete(serverURL + todo._id)
              .then(response => {console.log("Task DELETED!")})
              .catch(error => console.log(error));;
      },
      undoTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos[foundIndex].done = false;
          axios.put( serverURL + todo._id, {done: todo.done})
              .then(response => console.log("Task UNDONE!"))
              .catch(error => console.log(error));
      },
      addTask: function (todo) {
          this.dialog = false;
          this.addedTask = '';
          axios.post( serverURL , {todo: todo})
              .then(response => {
                  let todoItem = {
                      todo: todo,
                      done: false,
                      _id: response.data._id,
                  }
                  this.todos.push(todoItem)
              })
              .catch(error => console.log(error));
      },
      dialogActions(){
          this.dialog = !this.dialog;
      }
  }
}
</script>