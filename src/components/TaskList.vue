<template>
  <transition-group name="list" tag="section" class="task-list">
    <task-list-item
        v-for="task in sortedTasks"
        :key="task.id"
        :task="task"
        @updateTask="updateTask"
        @toggleDone="toggleDone"
        @deleteTask="deleteTask" />
  </transition-group>
</template>

<script>
import TaskListItem from '@/components/TaskListItem'

export default {
  components: { TaskListItem },
  props: ['tasks', 'selectedPriority', 'selectedCategory', 'sortAscending'],
  computed: {
    tasksSortByDue () {
      let sortedTasks = [ ...this.filteredTasks ]
      return sortedTasks.sort((a, b) => new Date(a.dueAt) - new Date(b.dueAt))
    },
    filteredPriorityTasks () {
      return (!this.selectedPriority)
        ? this.tasks
        : this.tasks.filter(task => task.priority === this.selectedPriority)
    },
    filteredCategoryTasks () {
      console.log(this.selectedCategory)
      return (this.selectedCategory === '')
        ? this.tasks
        : this.tasks.filter(task => task.category === this.selectedCategory)
    },
    filteredTasks () {
      const combined = [
        ...this.filteredCategoryTasks,
        ...this.filteredPriorityTasks
      ]
      return combined.reduce((accumulator, task) => {
        if (this.filteredPriorityTasks.includes(task) &&
        this.filteredCategoryTasks.includes(task) &&
        !accumulator.includes(task)) {
          accumulator.push(task)
        }
        return accumulator
      }, [])
    },
    sortedTasks () {
      let tasks = [...this.filteredTasks]
      return tasks.sort((a, b) => {
        const dateOne = new Date(a.dueAt)
        const dateTwo = new Date(b.dueAt)
        return this.sortAscending ? (dateOne - dateTwo) : (dateTwo - dateOne)
      })
    }
  },
  methods: {
    toggleDone (task) {
      this.$emit('toggleDone', task)
    },
    deleteTask (task) {
      this.$emit('deleteTask', task)
    },
    updateTask (task) {
      this.$emit('updateTask', task)
    }
  }
}
</script>

<style>
.list-move {
  transition: all 400ms ease-in-out 50ms;
}
.list-enter {
  opacity: 0;
  transform: translateY(-2em);
}
.list-leave-to {
  opacity: 0;
  transform: translateY(-2em);
}
.list-leave-active {
  position: absolute;
}
</style>
