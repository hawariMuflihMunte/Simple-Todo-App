<template>
  <nav>
    <a href="./">Home</a>
    <ul>
      <li>
        <a href="#" @click="toggleShowTask">Add</a>
      </li>
    </ul>
  </nav>

  <div v-if="showAddTask" ref="addTaskForm" @click="handleAddTaskForm">
    <AddTaskForm @addTask="handleAddTask" />
  </div>
</template>

<script>
import AddTaskForm from './AddTaskForm.vue'

export default {
  emits: ['addedTask'],
  data() {
    return {
      showAddTask: false
    }
  },
  methods: {
    toggleShowTask() {
      this.showAddTask = !this.showAddTask
    },
    handleAddTaskForm(e) {
      const target = e.target

      const addTaskForm = this.$refs.addTaskForm

      if (target === addTaskForm) {
        this.showAddTask = !this.showAddTask
      }
    },
    handleAddTask(data) {
      this.toggleShowTask()
      this.$emit('addedTask', data)
    }
  },
  components: { AddTaskForm }
}
</script>

<style scoped></style>
