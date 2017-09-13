<template>
    <div id="app">
        <v-app>
            <v-layout row>
                <v-flex xs12 sm12 md12 lg6 offset-lg3>
                    <v-card class="my-top-margin">
                        <!--Main toolbar-->
                        <v-toolbar class="my-toolbar" extended light>
                                <v-btn v-if="authenticated" icon
                                       @click="logout()" v-tooltip:bottom="{ html: 'Logout' }">
                                    <v-icon>vpn_key</v-icon>
                                </v-btn>
                                <v-btn v-if="authenticated" class="my-plus" fab dark absolute bottom right
                                       @click.native.stop="dialogActions()" v-tooltip:top="{ html: 'Add a task' }">
                                    <v-icon>add</v-icon>
                                </v-btn>
                                <v-toolbar-title v-if="authenticated" slot="extension" class="white--text">My To-Do list</v-toolbar-title>
                                <v-toolbar-title v-if="!authenticated" slot="extension" class="white--text">Please login</v-toolbar-title>
                            </v-toolbar>
                            <div v-if="authenticated">
                            <!--Listing all todos-->
                            <v-list subheader>
                                <v-subheader></v-subheader>
                                <v-list-tile avatar v-for="todo in filterTodos" :key="todo._id">
                                    <v-list-tile-avatar>
                                        <i class="material-icons">work</i>
                                    </v-list-tile-avatar>
                                    <v-list-tile-content>
                                        <v-list-tile-title
                                            v-html="todo.todo"
                                            :class="{ completed: todo.done == true }">
                                        </v-list-tile-title>
                                    </v-list-tile-content>
                                    <v-list-tile-action>
                                        <span>
                                            <v-btn v-if="todo.done" @click="undoTask(todo)" icon
                                                   class="material-icons my-green">undo</v-btn>
                                            <v-btn v-if="!todo.done" @click="doneTask(todo)" icon
                                                   class="material-icons my-green">done</v-btn>
                                            <v-btn @click.stop="editTask(todo)" icon
                                                   class="material-icons my-orange">mode_edit</v-btn>
                                            <v-btn @click="deleteTask(todo)" icon
                                                   class="material-icons my-red">delete</v-btn>
                                        </span>
                                    </v-list-tile-action>
                                </v-list-tile>
                            </v-list>
                            <!--Dialog part-->
                            <v-dialog v-model="dialog">
                                <v-card>
                                    <v-card-text>
                                        <v-text-field ref="inputTodo" v-model="addedTask" label="Add a task"
                                                      @keyup.13="addTask(addedTask)"></v-text-field>
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
                                        <v-text-field v-model="editedTask.todo" label="Edit task" @keyup.13="updateTask(editedTask)"></v-text-field>
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
                                <v-btn flat class="teal--text my-top-margin"  @click="status = 'all'" value="allTasks">
                                    <span>All tasks</span>
                                    <v-icon v-badge="{value: todos.length, right: true}">history</v-icon>
                                </v-btn>
                                <v-btn flat class="teal--text my-top-margin" @click="status = false" value="todoTasks">
                                    <span>To-do tasks</span>
                                    <i icon v-badge="{value: todoTodos.length, right: true}" class="material-icons">work</i>
                                </v-btn>
                                    <v-btn flat class="teal--text my-top-margin" @click="status = true"  value="doneTasks">
                                    <span>Done tasks</span>
                                    <i icon v-badge="{value: doneTodos.length, right: true}" class="material-icons">done</i>
                                </v-btn>
                            </v-bottom-nav>
                        </div>
                        <div v-if="!authenticated">
                            <div class="my-all-margin">
                                <v-alert info>Please Login first:</v-alert>
                                <v-form>
                                    <v-text-field
                                      label="Username"
                                      v-model="username"
                                      required
                                    ></v-text-field>
                                    <v-text-field
                                      label="Password"
                                      v-model="password"
                                      required
                                      type="password"
                                      @keyup.13="login()"
                                    ></v-text-field>
                                    <div class="center-button">
                                        <v-btn outline class="my-bottom-margin" @click="login()">Login</v-btn>
                                    </div>
                                </v-form>
                            </div>
                        </div>
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
    .my-toolbar { background-color: #E8B30C !important; }
    .my-plus { background-color: #02530D !important; }
    .my-top-margin {margin-top: 5px !important;}
    .my-bottom-margin {margin-bottom: 20px !important;}
    .my-all-margin {margin: 20px;}
    .toolbar__title {font-size: 30px !important;}
    .center-button {  display: flex; justify-content: center;}
</style>

<script>
import axios from 'axios';

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
            status: 'all',
            authenticated: localStorage['token'] || false,
            username: '',
            password: ''
        }
    },
    created() {
      axios.get(serverURL)
        .then(response => this.todos = response.data)
        .catch(error => console.log(error));
    },
    computed:{
        doneTodos: function () {
            return this.todos.filter(todo => todo.done == true);
        },
        todoTodos: function () {
            return this.todos.filter(todo => todo.done == false);
        },
        filterTodos: function () {
          if(this.status == 'all'){
            return this.todos;
          }else{
            return this.todos.filter(todo => todo.done == this.status);
          }
        }
    },
    methods: {
        doneTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos[foundIndex].done = true;
          axios.put( serverURL + todo._id, {done: true})
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
              .then(() => {console.log("Task DELETED!")})
              .catch(error => console.log(error));;
        },
        undoTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos[foundIndex].done = false;
          axios.put( serverURL + todo._id, {done: false})
              .then(() => console.log("Task UNDONE!"))
              .catch(error => console.log(error));
        },
        addTask: function (todo) {
          this.addedTask = '';
          axios.post( serverURL , {todo: todo})
              .then(response => {
                  let todoItem = {
                      todo: todo,
                      done: false,
                      _id: response.data._id,
                  };
                  this.todos.push(todoItem);
                  this.dialog = false;
              })
              .catch(error => console.log(error));
        },
        dialogActions(){
          this.dialog = !this.dialog;
        },
        login(){
            this.authenticated = true;
            localStorage['token'] = true;
        },
        logout(){
            this.authenticated = false;
            localStorage['token'] = '';
        }
    }
}
</script>