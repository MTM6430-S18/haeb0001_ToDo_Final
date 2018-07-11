<template>
  <form class="new-task-form" @submit.prevent="addTask">
    <label>
      Title
      <input v-model.trim="newTask.title" type="text" required>
    </label>
    <label>
      Description
      <input v-model.trim="newTask.desc" type="text">
    </label>
    <label>
      <select v-model="newTask.priority">
        <option value="high">High</option>
        <option value="medium" selected>Medium</option>
        <option value="low">Low</option>
      </select>
    </label>
    <label>
      <select v-model="newTask.category">
        <option value="">None</option>
        <option value="Homework">Homework</option>
        <option value="Work">Work</option>
        <option value="Chores">Chores</option>
        <option value="Family">Family</option>
      </select>
    </label>
    <label>
      Due At
      <input v-model="newTask.dueAt" type="date">
    </label>
    <button type="submit">Add Task</button>
  </form>
</template>

<script>
export default {
  data: () => ({
    newTask: {}
  }),

  created () {
    this.resetNewTask()
  },

  methods: {
    addTask () {
      this.$emit('addTask', this.newTask)
      this.resetNewTask()
      this.$router.push('/active')
    },

    resetNewTask () {
      this.newTask = {
        id: Date.now(),
        title: '',
        desc: '',
        priority: 'medium',
        category: '',
        dueAt: (new Date()).toISOString().split('T')[0],
        isComplete: false
      }
    }
  }
}
</script>

<style lang="scss">
.new-task-form {
  margin-bottom: 2rem;

  label {
    display: flex;
    flex-direction: column;
    margin-bottom: 1rem;
  }

  input {
    background: #F8FBFF;
    border: 1px solid hsl(0, 0%, 93%);
    font-size: 1.1rem;
    padding: 0.4em;
    border-radius: 0.25rem;
  }

  button {
    background: #239CCC;
    border: 1px solid #239CCC;
    border-radius: 0.25rem;
    padding: 0.5rem 0.5rem;
    color: hsl(0, 0%, 99%);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    font-weight: 500;
    font-size: 1.1rem;
    width: 100%;
    box-shadow: 0px 1px 3px 0px rgba(0, 0, 0, 0.25);
    outline: none;

    &:active {
      box-shadow: none;
      background: #239CCC;
      border: 1px solid #239CCC;
    }
  }
  input:focus, select:focus {
    outline: 1px solid #239CCC;
  }

  select{
    font-size: 1rem;
  }
}
</style>
