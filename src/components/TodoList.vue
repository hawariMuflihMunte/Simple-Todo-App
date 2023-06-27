<template>
  <section>
    <button @click="toggleDisplay">{{ showAll ? 'Hide Completed' : 'Show All' }}</button>

    <table>
      <thead>
        <tr>
          <th :class="sorted ? 'number' : 'number boldize'">#</th>
          <th @dblclick="sortByTask">Task</th>
          <th @dblclick="sortByDueDate">Due Date</th>
          <th @dblclick="sortByDesc">Description</th>
          <th>Option</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(d, index) in filteredData" :key="index" :class="d.completed ? 'done' : ''">
          <td>{{ index + 1 }}</td>
          <td>
            <template v-if="d.editing">
              <input v-model="d.name" autofocus @keydown.enter="editingSave(d)" @keydown.esc="editingCancel(d)">
            </template>
            <template v-else>
              {{ d.name }}
            </template>
          </td>
          <td>
            <template v-if="d.editing">
              <input v-model="d.dueDate" autofocus @keydown.enter="editingSave(d)" @keydown.esc="editingCancel(d)">
            </template>
            <template v-else>
              {{ d.dueDate }}
            </template>
          </td>
          <td>
            <template v-if="d.editing">
              <textarea v-model="d.description" autofocus @keydown.enter="editingSave(d)" @keydown.esc="editingCancel(d)"></textarea>
            </template>
            <template v-else>
              {{ d.description }}
            </template>
          </td>
          <td>
            <button @click="toggleCompleted(d)" v-if="!d.editing">{{ d.completed ? 'Undone' : 'Done' }}</button>
            <button @click="editingStart(d)" v-if="!d.editing">Edit</button>
            <button @click="deleteTask(d.id)" v-if="!d.editing">Delete</button>
            <button @click="editingCancel(d)" v-if="d.editing">Cancel</button>
          </td>
        </tr>
      </tbody>
    </table>
  </section>
</template>

<script>
import { ref, watchEffect, computed } from 'vue'

export default {
  setup(props) {
    const sorted = ref(null)
    const showAll = ref(true)
    const displayedData = ref([])

    watchEffect(() => {
      try {
        displayedData.value = props.data
      } catch (error) {
        console.error(error)
      }
    })

    const filteredData = computed(() => {
      if (showAll.value) {
        return displayedData.value // Return all tasks
      } else {
        // Filter out completed tasks
        return displayedData.value.filter(t => !t.completed)
      }
    })

    function editingStart(task) {
      task.editing = true
      updateSave()
    }

    function editingSave(task) {
      task.editing = false
      this.$emit('updateSave', this.displayedData)
    }

    function editingCancel(task) {
      task.editing = false
    }

    function deleteTask(id) {
      if (confirm('Are you sure want to delete this task?')) {
        const updatedData = displayedData.value.filter(d => d.id !== id)
        update(updatedData)
      }
    }

    function toggleCompleted(task) {
      task.completed = !task.completed
      updateSave()
    }

    function toggleDisplay() {
      showAll.value = !showAll.value
    }

    function updateSave() {
      const updatedData = displayedData.value.map(t => ({ ...t }))
      props.onUpdateSave(updatedData)
    }

    function update(updatedData) {
      props.onUpdate(updatedData)
    }

    return {
      sorted,
      showAll,
      displayedData,
      filteredData,
      editingStart,
      editingSave,
      editingCancel,
      deleteTask,
      toggleCompleted,
      toggleDisplay
    }
  },

  props: {
    data: Array,
    onUpdate: Function,
    onUpdateSave: Function
  }
}
</script>

<style scoped>
section {
  width: 100%;
  overflow-x: auto;
  margin-top: 60px;
}

table {
  width: 100%;
  table-layout: fixed;
}

table th,
table td {
  padding: 8px;
  text-align: left;
}

table thead tr th {
  border-bottom: 3px solid hsla(160, 100%, 37%, 1);
}

th.number {
  width: 60px;
  text-align: center;
}

tbody tr:nth-child(even) {
  background-color: #0da16d;
  color: #181818;
}

tbody tr td:first-child {
  text-align: center;
}

tbody tr:nth-child(even) button {
  background-color: #181818;
  color: hsla(160, 100%, 37%, 1);
}

td {
  overflow: hidden;
}

input,
textarea {
  width: 100%;
  min-height: 100%;
  border: 0;
  padding: 9px;
  font-family: inherit;
}

textarea {
  max-width: 100%;
  max-height: 100%;
  resize: vertical;
}

.done {
  text-decoration: line-through;
  font-style: italic;
  opacity: 0.9;
  font-size: smaller;
}

.boldize {
  font-weight: bold;
}
</style>
