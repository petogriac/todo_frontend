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
                                @click.native.stop="dialog = !dialog"
                              >
                                <v-icon>add</v-icon>
                              </v-btn>
                            <v-toolbar-title slot="extension" class="white--text">My TODO's</v-toolbar-title>
                          </v-toolbar>
                        <!--Listing all todos-->
                            <v-list subheader>
                            <v-subheader>TODO items</v-subheader>
                                <v-list-tile avatar v-for="todo in todos">
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
                                          <v-btn v-if="todo.done == 'true'" @click="undoTask(todo)" icon class="material-icons my-green">undo</v-btn>
                                          <v-btn v-if="todo.done == 'false'" @click="doneTask(todo)" icon class="material-icons my-green">done</v-btn>
                                          <v-btn @click="editTask(todo)" icon class="material-icons my-orange">mode_edit</v-btn>
                                          <v-btn @click="deleteTask(todo)" icon class="material-icons my-red">delete</v-btn>
                                      </span>
                                  </v-list-tile-action>
                                </v-list-tile>
                          </v-list>
                        <!--Dialog part-->
                            <v-dialog v-model="dialog">
                          <v-card>
                           <v-card-text>
                              <v-text-field label="Add a task"></v-text-field>
                              <small class="grey--text">* This doesn't actually save.</small>
                           </v-card-text>
                           <v-card-actions>
                             <v-spacer></v-spacer>
                             <v-btn flat primary @click.native="dialog = false">Submit</v-btn>
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

export default {
    name: 'app',
  data(){
      return {
          dialog: false,
          todos: []
      }
  },
  created() {
      axios.get('http://localhost:3000/todos')
          .then(response => this.todos = response.data)
          .catch(error => console.log(error));
  },
  methods: {
      doneTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos[foundIndex].done = "true";
      },
      editTask: function (todo) {

      },
      deleteTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos.splice(foundIndex, 1);
      },
      undoTask: function (todo) {
          let foundIndex = this.todos.findIndex(x => x._id == todo._id);
          this.todos[foundIndex].done = "false";
      }

  }
}
</script>