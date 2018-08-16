<template>
  <div v-if="isEditing" class="task-list-item is-editing" @keyup.esc="cancel">
    <form @submit.prevent="save">
      <label>
        Title
        <input v-focus v-model.trim="editTask.title" type="text" required/>
      </label>
      <label>
        Description
        <input v-model.trim="editTask.description" type="text">
      </label>
      <label>
        <p>Set a priority:</p>
        <select v-model="editTask.priority.id">
          <option value="1">Low</option>
          <option value="2">Medium</option>
          <option value="3">High</option>
        </select>
      </label>
      <label>
        <p>Select a category:</p>
        <select v-model="editTask.category.id">
          <option value="5">None</option>
          <option value="1">Home</option>
          <option value="2">School</option>
          <option value="3">Work</option>
        </select>
      </label>
      <label>
        Due at
        <input v-model="editTask.dueAt" type="date" />
      </label>
      <span class="action-buttons">
        <button class="button isOutline" type="button" @click="cancel">cancel</button>
        <button class="button" type="submit">save</button>
      </span>
    </form>
  </div>
  <div v-else class="task-list-item taskcontain" >
    <div class="title">{{ task.title }}</div>
    <div class="complete"><font-awesome-icon :icon="statusIcon" @click="$emit('toggleDone', task)" /></div>
    <div class="desc">{{ task.description }}</div>
    <div class="date">{{ task.dueAt.split(' ')[0] }}</div>
    <div class="category">{{ categoryOutput }}</div>
    <div class="edit"  @click="edit"><img class="icon" src="../assets/edit.png"/> &nbsp;<font-awesome-icon :icon="deleteIcon" @click="$emit('deleteTask', task)" /></div>
    <div class="priority">{{ task.priority.name }}</div>
  </div>
</template>

<script>
import FontAwesomeIcon from '@fortawesome/vue-fontawesome'
import faCircle from '@fortawesome/fontawesome-free-regular/faCircle'
import faCheckCircle from '@fortawesome/fontawesome-free-regular/faCheckCircle'
import faTrashAlt from '@fortawesome/fontawesome-free-regular/faTrashAlt'

export default {
  components: { FontAwesomeIcon },
  directives: {
    focus: {
      inserted (el) {
        el.focus()
      }
    }
  },
  props: ['task'],
  data: () => ({
    isEditing: false,
    editTask: {}
  }),
  computed: {
    // categoryOutput allows us to not output the name of any null items
    categoryOutput () {
      if (this.task.category) {
        return this.task.category.name
      }
    },
    statusIcon () {
      return this.task.isComplete ? faCheckCircle : faCircle
    },
    deleteIcon () {
      return faTrashAlt
    }
  },
  methods: {
    cancel () {
      this.isEditing = false
    },
    edit () {
      this.editTask = Object.assign({}, this.task)
      if (this.editTask.category) {
        // if this task does have a category, proceed as normal
      } else {
        // if the task id is null, give it an object with an ID so it does not throw an error
        this.editTask.category = {id: 5}
      }
      this.editTask.dueAt = this.editTask.dueAt.split(' ')[0]
      this.isEditing = true
    },
    save () {
      // following line resets time to be at the end of the day so the task is not late until the day is over
      this.editTask.dueAt = this.editTask.dueAt + ' 23:59:59'
      this.$emit('updateTask', this.editTask)
      this.isEditing = false
    }
  }
}
</script>

<style lang="scss">
.task-list-item {
  align-items: center;
  margin: 0.5rem 0;
  padding: 0.5rem;
  transition: all 0.4s;
}

.taskcontain {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  grid-gap: 10px;
  padding: 10px;
  color: black;
}

.title {
  grid-column: 2/7;
  grid-row: 1;
  font-weight: 800;
}
.complete {
  grid-column: 1;
  grid-row: 1/4;
}
.desc {
  grid-column: 2/8;
  grid-row: 2;
}
.date {
  grid-column: 4/7;
  grid-row: 3;
  color: #999999;
}
.category {
  grid-column: 2/4;
  grid-row: 3;
  color: #999999;
}
.edit {
  text-align: right;
  grid-column: 7;
  grid-row: 1;
}
.priority{
  grid-column: 7;
  grid-row: 3;
  color: #999999;
}

.is-editing {
  margin: 1rem 0;
  background: hsl(0, 0%, 99%);
  border-radius: 0.25rem;
  box-shadow: 0px 1px 5px 0px hsl(0, 0%, 84%);
  padding-top: 1rem;
  padding-bottom: 1rem;

  form {
    display: flex;
    flex-direction: column;
  }

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

  .action-buttons {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 1rem;
  }

  .button {
    background: hsl(197, 90%, 43%);
    border: 1px solid hsl(197, 90%, 48%);
    border-radius: 0.25rem;
    padding: 0.5rem 0.5rem;
    color: hsl(0, 0%, 99%);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    font-weight: 500;
    font-size: 1.1rem;
    box-shadow: 0px 1px 3px 0px rgba(0, 0, 0, 0.25);
    outline: none;

    &:active {
      box-shadow: none;
      background: hsl(197, 90%, 33%);
      border: 1px solid hsl(197, 90%, 38%);
    }

    &.isOutline {
      background: transparent;
      color: hsl(197, 90%, 43%);
      box-shadow: none;
    }
  }
}
.edit-enter,
.edit-leave-to {
  opacity: 0;
  transform: translateY(2em);
}
.edit-leave-active {
  position: absolute;
}
.icon{
  width: 15px;
  height: 15px;
}

p{
  margin-top: 0;
  margin-bottom: 3px;
}

</style>
