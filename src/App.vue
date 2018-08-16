<template>
  <div id="app" v-if="isLoggedIn">
    <the-header v-once />

    <div class="logout pointer" @click="logoutUser">
    <h3>Logout</h3>
    </div>

    <div class="sortOptions">
      <span @click="sortAscending = !sortAscending">Sort Date <img src="@/assets/arrow.png" class="icon pointer" /></span>
      <select name="priority-filter" id="priority-filter" v-model="selectedPriority">
      <option value="">Priority</option>
        <option v-for="option in priorityOptions" :value="option.id" :key="option.id">{{ option.name }}</option>
      </select>
      <select name="category-filter" id="category-filter" v-model="selectedCategory">
      <option value="">Category</option>
        <option v-for="option in categoryOptions" :value="option.id" :key="option.id">{{ option.name }}</option>
      </select>
    </div>

    <div id="nav" class="tabs pointer">
      <router-link to="/active">Active Tasks</router-link>
      <router-link to="/completed">Completed Tasks</router-link>
      <router-link to="/addTask">+ Task</router-link>
    </div>

    <transition-group name="tab" :duration="{ enter: 600, leave: 300 }" appear>
      <router-view
        :tasks="overdueTasks"
        :class="{ 'overdue': !!overdueTasks.length }"
        name="overdue"
        key="overdue"
        :selectedPriority="selectedPriority"
        :selectedCategory="selectedCategory"
        :sortAscending="sortAscending"
        @updateTask="updateTask"
        @toggleDone="toggleDone"
        @deleteTask="deleteTask"
      />
      <router-view
        :tasks="upcomingTasks"
        class="upcoming"
        name="upcoming"
        key="upcoming"
        :selectedPriority="selectedPriority"
        :selectedCategory="selectedCategory"
        :sortAscending="sortAscending"
        @updateTask="updateTask"
        @toggleDone="toggleDone"
        @deleteTask="deleteTask"
      />

      <router-view
        :tasks="completedTasks"
        :class="{ 'completed': !!completedTasks.length }"
        name="completed"
        key="completed"
        :selectedPriority="selectedPriority"
        :selectedCategory="selectedCategory"
        :sortAscending="sortAscending"
        @updateTask="updateTask"
        @toggleDone="toggleDone"
        @deleteTask="deleteTask"
      />

      <router-view
      key="newTask"
      @addTask="addTask" @updateTask="updateTask"/>

    </transition-group>

  </div>

    <login-form v-else @userDidAuthenticate="loginUser" />

</template>

<script>
import NewTaskForm from '@/components/NewTaskForm'
import TaskList from '@/components/TaskList'
import TheHeader from '@/components/TheHeader'
import axios from 'axios'
import moment from 'moment'
import LoginForm from '@/components/loginForm'

export default {
  // Global Awareness
  name: 'app',

  // Template Dependencies
  components: { NewTaskForm, TaskList, TheHeader, LoginForm },

  // Local State
  data: () => ({
    api: {
      accessToken: '',
      expiresAt: ''
    },
    newTask: {},
    taskList: {},
    priorityOptions: [
      {id: 1, name: 'High'},
      {id: 2, name: 'Medium'},
      {id: 3, name: 'Low'}
    ],
    selectedPriority: '',
    categoryOptions: [
      {id: 5, name: 'None'},
      {id: 1, name: 'Homework'},
      {id: 2, name: 'Chores'},
      {id: 3, name: 'Work'},
      {id: 4, name: 'Family'}
    ],
    selectedCategory: '',
    sortAscending: true
  }),
  computed: {
    activeTasks () {
      return Object.values(this.taskList).filter(task => !task.isComplete)
      // return this.tasks.filter(task => !task.isComplete)
    },
    completedTasks () {
      return Object.values(this.taskList).filter(task => task.isComplete)
    },
    overdueTasks () {
      return this.activeTasks.filter(
        // if you want to make things due today, add time to the dueAt to midnight tonight +23:59:59
        task => new Date(task.dueAt) < Date.now() && !task.isComplete
      )
    },
    upcomingTasks () {
      return this.activeTasks.filter(task => new Date(task.dueAt) >= Date.now())
    },
    isLoggedIn () {
      return this.api.accessToken && moment(this.api.expiresAt).isAfter()
    },
    axiosOptions () {
      return {
        baseURL: 'https://vue-todos.robertmckenney.ca/api',
        headers: { 'Authorization': `Bearer ${this.api.accessToken}` }
      }
    }
  },
  // Reactive and lifcycle Events
  watch: {
    tasks: {
      handler: function () { this.syncTasks() },
      deep: true
    }
  },
  created () {
    this.loadCachedData()
  },
  // Non-Reactive Properties
  methods: {
    syncTasks () {
      localStorage.setItem('taskList', JSON.stringify(this.tasks))
    },
    toggleDone (task) {
      task.isComplete = !task.isComplete
      this.updateTask(task)
    },
    updateTask (task) {
      axios.put(`/tasks/${task.id}`, task, this.axiosOptions)
        .then(({data: {data: t}}) => {
          // t in this scenario is data: data
          this.$set(this.taskList, t.id, t)
          this.refreshTasks()
        })
        .catch(error => this.handleAPIErrors(error))
      // let target = this.tasks.find(t => t.id === task.id) // eslint-disable-line no-unused-vars
      // target = Object.assign(target, task)
    },
    // This method is called when the LoginForm emits the saveApiTokens event
    saveApiTokens (apiTokens) {
      this.api.accessToken = apiTokens.access_token
      this.api.expiresAt = apiTokens.expires_at
      localStorage.setItem('todoApiTokens', JSON.stringify(apiTokens))
      // We just logged-in a new user. We better refresh our task list.
      this.refreshTasks()
    },
    refreshTasks () {
      axios.get('/tasks', this.axiosOptions)
        // we can destructure the response object to get the nested data payload
        .then(({data: {data}}) => {
          // Assign the array of tasks from the API response payload
          // this.taskList = data
          // This version transforms the array into a dictionary object with
          // the task.id as the keys
          this.taskList = Object.assign({}, ...data.map(t => ({ [t.id]: t })))
        })
        .catch(error => this.handleAPIErrors(error))
    },
    addTask (task) {
      task.dueAt = task.dueAt + ' 23:59:59'
      console.log(task)
      axios.post('/tasks', task, this.axiosOptions)
        // when destructuring, we can also assign a different name (e.g. "t")
        .then(({data: {data: t}}) => {
          // We would expect to be able to simply add the new object property
          // and value, like this ...
          // this.taskList[t.id] = t
          // However, Vue is not able to correctly observe this action, and
          // the values of our task properties (e.g. title, dueAt, etc) will
          // not be reactive when edited. See https://vuejs.org/v2/api/#Vue-set
          // So, we need to use Vue's special $set() method ...
          this.$set(this.taskList, t.id, t)
          console.log(t)
        })
        .catch(error => this.handleAPIErrors(error))
    },
    deleteTask (task) {
      axios.delete(`/todos/${task.id}`, this.axiosOptions)
        .then(response => {
          // We would expect to be able to simply delete the object property
          // cooresponding to our dictionary key, like this ...
          // delete this.taskList[task.id]
          // However, Vue is not able to directly observe this action, and
          // we will not see the reactivity system update our list.
          // So, we need to use a special $delete() method on our Vue
          // component instance ...
          this.$delete(this.taskList, task.id)
        })
        .catch(error => this.handleAPIErrors(error))
    },
    loadCachedData () {
      let apiTokens = localStorage.getItem('todoApiTokens')
      if (apiTokens === undefined) return
      apiTokens = JSON.parse(apiTokens)
      this.loginUser(apiTokens)
    },
    loginUser (apiTokens) {
      this.api.accessToken = apiTokens.access_token
      this.api.expiresAt = apiTokens.expires_at
      this.saveApiTokens(apiTokens)
      this.refreshTasks()
    },
    logoutUser () {
      this.api.accessToken = ''
      this.api.expiresAt = ''
      localStorage.removeItem('todoApiTokens')
    },
    // These are leftover methods from last week's exercise. But you can use
    // as guides to how to update your todo app methods.
    // Retrive collection of objects
    fetchUsers () {
      axios
        .get(`${this.baseUrl}/users`)
        .then(response => {
          // The response object has several properties:
          // { config, headers, data, request, status, statusText }
          // Our requested payload is in the "data" key, and for this
          // resource request it will be an array of user objects.
          // this.users = response.data
          // We need to do this differently if this.users is a
          // dictionary object rather than an array ...
          // [see](https://medium.com/dailyjs/rewriting-javascript-converting-an-array-of-objects-to-an-object-ec579cafbfc7)
          this.users = Object.assign({}, ...response.data.map(user => ({[user.id]: user})))
        })
        .catch(error => this.handleAPIErrors(error))
    },
    // Retrive a single object
    getUser (id) {
      axios
        .get(`${this.baseUrl}/users/${id}`)
        .then(response => console.log(response.data))
        .catch(error => this.handleAPIErrors(error))
    },
    // Create a new object
    createUser (newUser) {
      //  validation
      // this.users.push(newUser)
      // this.newUser = { name: '' }
      axios
        .post(`${this.baseUrl}/users`, newUser)
        // We can use object destructuring to get the data property
        .then(({ data }) => {
          // this.users.push(data)
          this.users[data.id] = data
          this.newUser = { name: '' }
        })
        .catch(error => this.handleAPIErrors(error))
    },
    // Change the properties of an object
    updateUser (user) {
      axios
        .patch(`${this.baseUrl}/users/${user.id}`, user)
        .then(response => {
          // const index = this.users.findIndex(user)
          // this.users.splice(index, 1)
          // this.users.push(result.data)
          this.users[user.id] = response.data
        })
        .catch(error => this.handleAPIErrors(error))
    },
    // Remove an object
    destroyUser (user) {
      axios
        .delete(`${this.baseUrl}/users/${user.id}`)
        .then(response => {
          // const index = this.users.findIndex(user)
          // this.users.splice(index, 1)
          delete this.users[user.id]
        })
        .catch(error => this.handleAPIErrors(error))
    },
    // A common error handler function
    handleAPIErrors (error) {
      // We will make this more robust net week
      console.log(error.message)
    }
    // fetchCategories(){
    //   axios.get('/categories', this.axiosOptions)
    //     .then (response => {
    //       this.categoryOptions = response.data.data
    //     })
    // }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 1rem;
}

.task-list {
  margin: 1rem 0;
  &.upcoming{
    > div{
      box-shadow: 0px 0px 1px #DEF1FF;
    }
  }
  &.completed {
    background: #F8FBFF;
    > div{
      box-shadow: 0px 0px 1px #B0D0FF;
    }
  }
  &.overdue {
    border-bottom: 1px solid hsl(348, 83%, 47%);
    background: hsl(348, 50%, 98.5%);
    color: hsl(348, 20%, 40%);
    box-shadow: 0px 0px 1px #FF5555;
  }
}

.tabs {
  display: flex;
  justify-content: space-between;
  margin-bottom: 1rem;

  a {
    border-bottom: 1px solid hsl(0, 0%, 88%);
    color: #2c3e50;
    flex: 1;
    font-weight: bold;
    letter-spacing: 0.5px;
    padding: 0.5rem 1rem;
    text-decoration: none;
    text-transform: uppercase;
    transition: all 0.6s ease-in-out;

    &.router-link-exact-active {
      border-bottom-color: #239CCC;
      color: #239CCC;
    }
  }
}

// Tab transition styles
.tab-enter {
  opacity: 0;
  transform: translateY(-200%);
}
.tab-leave-to {
  opacity: 0;
  transform: translateY(200%);
}
.tab-enter-active {
  transition: all 0.4s ease;
}
.tab-leave-active {
  transition: all 0.4s cubic-bezier(1, 0.5, 0.8, 1);
}
.sortOptions{
  max-width: 600px;
  margin: 0 auto;
  margin-top: 20px;
  margin-bottom: 20px;
  display: flex;
  justify-content: space-between;
  font-size: .9rem;
  & > select{
    font-size: .9rem;
    width: 100px;
  }
}
body{
  max-width: 800px;
  margin: 0 auto;
}

.pointer{
  cursor: pointer;
}

.logout{
  height: 30px;
  border-radius: 5px;
  width: 100px;
  text-align: center;
  background-color: #239CCC;
  margin-left:auto;
  margin-right:0;
}

.logout h3{
  position: relative;
  top: 50%;
  -webkit-transform: translateY(-50%);
  -ms-transform: translateY(-50%);
  transform: translateY(-50%);
}
</style>
