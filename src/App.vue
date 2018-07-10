<template>
  <div id="app">
    <the-header v-once />
    <new-task-form @addTask="addTask" />

    <div id="nav" class="tabs">
      <router-link to="/active">Active Tasks</router-link>
      <router-link to="/completed">Completed Tasks</router-link>
    </div>

    <transition-group name="tab" :duration="{ enter: 600, leave: 300 }" appear>
      <router-view
        :tasks="overdueTasks"
        :class="{ 'overdue': !!overdueTasks.length }"
        name="overdue"
        key="overdue"
        @updateTask="updateTask"
        @toggleDone="toggleDone"
        @deleteTask="deleteTask"
      />
      <router-view
        :tasks="upcomingTasks"
        name="upcoming"
        key="upcoming"
        @updateTask="updateTask"
        @toggleDone="toggleDone"
        @deleteTask="deleteTask"
      />

      <router-view
        :tasks="completedTasks"
        :class="{ 'completed': !!completedTasks.length }"
        name="completed"
        key="completed"
        @updateTask="updateTask"
        @toggleDone="toggleDone"
        @deleteTask="deleteTask"
      />
    </transition-group>
  </div>
</template>

<script>
import NewTaskForm from '@/components/NewTaskForm'
import TaskList from '@/components/TaskList'
import TheHeader from '@/components/TheHeader'

export default {
  // Global Awareness
  name: 'app',

  // Template Dependencies
  components: { NewTaskForm, TaskList, TheHeader },

  // Local State
  data: () => ({
    newTask: {},
    tasks: [
      // We are now overriding these in the created() event method
    ]
  }),
  computed: {
    activeTasks () {
      return this.tasks.filter(task => !task.isComplete)
    },
    completedTasks () {
      return this.tasks.filter(task => task.isComplete)
    },
    overdueTasks () {
      return this.activeTasks.filter(
        // if you want to make things due today, add time to the dueAt to midnight tonight +23:59:59
        task => new Date(task.dueAt) < Date.now() && !task.isComplete
      )
    },
    upcomingTasks () {
      return this.activeTasks.filter(task => new Date(task.dueAt) >= Date.now())
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
    this.tasks = JSON.parse(localStorage.getItem('taskList')) || []
  },

  // Non-Reactive Properties
  methods: {
    addTask (task) {
      this.tasks.push(task)
    },
    deleteTask (task) {
      const index = this.tasks.findIndex(t => t.id === task.id)
      this.tasks.splice(index, 1)
    },
    syncTasks () {
      localStorage.setItem('taskList', JSON.stringify(this.tasks))
    },
    toggleDone (task) {
      task.isComplete = !task.isComplete
    },
    updateTask (task) {
      // Instead of duplicating the code for delete and add, it is better to
      // just call those existing methods:
      // this.deleteTask(task)
      // this.addTask(task)

      // Alternatively, now that we are running the deep watch on our tasks
      // array, we WILL now have reactivity if we directly update a task
      // object's properties ...
      let target = this.tasks.find(t => t.id === task.id) // eslint-disable-line no-unused-vars
      target = Object.assign(target, task)
    }
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
  &.completed {
    // border-top: 1px solid hsl(153, 47%, 49%);
    background: hsl(153, 30%, 98.5%);
    color: hsl(153, 16%, 40%);
  }
  &.overdue {
    border-bottom: 1px solid hsl(348, 83%, 47%);
    background: hsl(348, 50%, 98.5%);
    color: hsl(348, 20%, 40%);
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
      border-bottom-color: #42b983;
      color: #42b983;
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
</style>
