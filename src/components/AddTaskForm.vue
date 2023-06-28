<template>
  <section>
    <form @submit.prevent="addTask">
      <input type="text" v-model="taskName" />
      <input type="text" v-model="dueDate" />
      <textarea v-model="description"></textarea>
      <button type="submit">Add Task</button>
    </form>
  </section>
</template>

<script>
export default {
  emits: ['addTask'],
  data() {
    return {
      taskName: '',
      dueDate: '',
      description: ''
    }
  },
  methods: {
    addTask() {
      // Validate Task Name
      if (this.taskName === '') {
        alert('Empty Task Name')
        return
      }

      // Validate Task Due Date
      if (this.dueDate === '') {
        alert('Empty Task Due Date')
        return
      }

      const id = +new Date()
      const raw = {
        id: id,
        name: this.taskName,
        description: this.description,
        dueDate: this.dueDate
      }

      this.$emit('addTask', raw)
    }
  }
}
</script>

<style scoped>
section {
  position: fixed;
  top: 12%;
  left: 50%;
  transform: translateX(-50%);
  min-width: 240px;
  max-width: 380px;
  z-index: 10000;
}

section form {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

input,
textarea {
  border: 0;
  padding: 9px;
  font-family: inherit;
}
</style>
