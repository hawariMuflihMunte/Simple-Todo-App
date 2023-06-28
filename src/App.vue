<template>
  <nav>
    <ul>
      <li>
        <button @click="toggleAddTaskDialog">Add Task</button>
      </li>
    </ul>
  </nav>

  <dialog ref="addTask" @click.self="toggleAddTaskDialog">
    <div>
      <template v-if="showError">
        <label class="error"><small>Empty Task Title</small></label>
      </template>
      <input v-model="newData.title" @keydown.enter="addTask" type="text" placeholder="Title" />
      <input v-model="newData.dueDate" type="date" placeholder="Due Date" />
      <textarea v-model="newData.description" placeholder="Description"></textarea>
      <button class="full-width" @click="addTask">Add Task</button>
    </div>
  </dialog>

  <button class="display-toggle" @click="toggleDisplay">
    {{ showAll ? 'Hide Completed' : 'Show All' }}
  </button>

  <section class="table-wrapper">
    <table>
      <thead>
        <tr>
          <th class="number">#</th>
          <th @dblclick="sortByTask">Task</th>
          <th @dblclick="sortByDueDate">Due Date</th>
          <th @dblclick="sortByDesc">Description</th>
          <th>Option</th>
        </tr>
      </thead>
      <tbody v-if="data.length > 0">
        <template v-if="showAll">
          <tr v-for="(d, index) in data" :key="index" :class="d.completed ? 'done' : ''">
            <td>{{ index + 1 }}</td>
            <td>
              <template v-if="d.editing">
                <input
                  v-model="d.name"
                  type="text"
                  autofocus
                  @keydown.enter="editingSave(d)"
                  @keydown.esc="editingCancel(d)"
                />
              </template>
              <template v-else>
                {{ d.name }}
              </template>
            </td>
            <td>
              <template v-if="d.editing">
                <input
                  v-model="d.dueDate"
                  type="date"
                  autofocus
                  @keydown.enter="editingSave(d)"
                  @keydown.esc="editingCancel(d)"
                />
              </template>
              <template v-else>
                {{ d.dueDate }}
              </template>
            </td>
            <td>
              <template v-if="d.editing">
                <textarea
                  v-model="d.description"
                  type="text"
                  autofocus
                  @keydown.enter="editingSave(d)"
                  @keydown.esc="editingCancel(d)"
                ></textarea>
              </template>
              <template v-else>
                {{ d.description }}
              </template>
            </td>
            <td>
              <button @click="toggleCompleted(d)" v-if="!d.editing">
                {{ d.completed ? 'Undone' : 'Done' }}
              </button>
              <button @click="editingStart(d)" v-if="!d.editing">Edit</button>
              <button @click="deleteTask(d.id)" v-if="!d.editing">Delete</button>
              <button @click="editingSave(d)" v-if="d.editing">Save</button>
              <button @click="editingCancel(d)" v-if="d.editing">Cancel</button>
            </td>
          </tr>
        </template>
        <template v-else>
          <tr v-for="(d, index) in hideCompleted" :key="index" :class="d.completed ? 'done' : ''">
            <td>{{ index + 1 }}</td>
            <td>
              <template v-if="d.editing">
                <input
                  v-model="d.name"
                  type="text"
                  autofocus
                  @keydown.enter="editingSave(d)"
                  @keydown.esc="editingCancel(d)"
                />
              </template>
              <template v-else>
                {{ d.name }}
              </template>
            </td>
            <td>
              <template v-if="d.editing">
                <input
                  v-model="d.dueDate"
                  type="date"
                  autofocus
                  @keydown.enter="editingSave(d)"
                  @keydown.esc="editingCancel(d)"
                />
              </template>
              <template v-else>
                {{ d.dueDate }}
              </template>
            </td>
            <td>
              <template v-if="d.editing">
                <textarea
                  v-model="d.description"
                  type="text"
                  autofocus
                  @keydown.enter="editingSave(d)"
                  @keydown.esc="editingCancel(d)"
                ></textarea>
              </template>
              <template v-else>
                {{ d.description }}
              </template>
            </td>
            <td>
              <button @click="toggleCompleted(d)" v-if="!d.editing">
                {{ d.completed ? 'Undone' : 'Done' }}
              </button>
              <button @click="editingStart(d)" v-if="!d.editing">Edit</button>
              <button @click="deleteTask(d.id)" v-if="!d.editing">Delete</button>
              <button @click="editingSave(d)" v-if="d.editing">Save</button>
              <button @click="editingCancel(d)" v-if="d.editing">Cancel</button>
            </td>
          </tr>
        </template>
      </tbody>
      <tbody v-else>
        <tr>
          <td colspan="5">No task(s) available</td>
        </tr>
      </tbody>
    </table>
  </section>
</template>

<script>
import tasks from './data/data'

export default {
  data() {
    return {
      data: tasks,
      showAll: true,
      showAddTaskDialog: false,
      newData: {
        title: '',
        dueDate: '',
        description: ''
      },
      showError: false
    }
  },
  watch: {
    'newData.title'(newTitle, oldTitle) {
      if (newTitle !== oldTitle) {
        this.showError = false
      }
    },
    data: {
      handler(newData) {
        localStorage.setItem('tasks', JSON.stringify(newData))
      },
      deep: true
    }
  },
  computed: {
    hideCompleted() {
      return this.data.filter((d) => !d.completed)
    }
  },
  methods: {
    toggleAddTaskDialog() {
      this.showAddTaskDialog = !this.showAddTaskDialog

      const dialog = this.$refs.addTask
      if (this.showAddTaskDialog) {
        dialog.showModal()
      } else {
        dialog.close()
      }
    },
    toggleCompleted(task) {
      task.completed = !task.completed
    },
    toggleDisplay() {
      this.showAll = !this.showAll
    },
    addTask() {
      if (this.newData.title !== '') {
        this.showError = false

        const id = +new Date()
        const newDataToBeInsertedIntoDataArray = {
          id,
          name: this.newData.title,
          dueDate: this.newData.dueDate === '' ? 'N/A' : this.newData.dueDate,
          description: this.newData.description === '' ? 'N/A' : this.newData.description
        }

        this.data.push(newDataToBeInsertedIntoDataArray)
        console.log(this.data)

        setTimeout(() => {
          // Perform data clearance
          this.newData.title = ''
          this.newData.dueDate = ''
          this.newData.description = ''

          this.toggleAddTaskDialog()
        }, 200)
      } else {
        this.showError = true
      }
    },
    deleteTask(id) {
      if (confirm('Are you sure want to delete this task?')) {
        const index = this.data.findIndex((d) => d.id === id)

        if (index !== -1) {
          this.data.splice(index, 1)
          console.log(this.data)
        }
      }
    },
    editingStart(task) {
      task.editing = true

      if (task.dueDate === '' || task.dueDate === 'N/A') {
        task.dueDate = ''
      }

      if (task.description === '' || task.description === 'N/A') {
        task.description = ''
      }
    },
    editingSave(task) {
      task.editing = false

      this.data.forEach((d) => {
        if (d.id === task.id) {
          d.name = task.name
          d.dueDate = task.dueDate === '' ? 'N/A' : task.dueDate
          d.description = task.description === '' ? 'N/A' : task.description
        }
      })
    },
    editingCancel(task) {
      task.editing = false

      if (task.dueDate === '' || task.dueDate === 'N/A') {
        task.dueDate = 'N/A'
      }

      if (task.description === '' || task.description === 'N/A') {
        task.description = 'N/A'
      }
    }
  }
}
</script>

<style>
.full-width {
  width: 100%;
}

.error {
  color: crimson;
}

nav {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background-color: hsla(160, 100%, 37%, 0.2);
  padding: 15px 29px;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

nav ul {
  display: inline-flex;
  list-style: none;
}

.display-toggle {
  margin-top: 60px;
}

dialog {
  position: fixed;
  top: 10%;
  margin-left: auto;
  margin-right: auto;
  z-index: 99;
  min-width: 220px;
  max-width: 460px;
  border: 0;
  background-color: hsla(160, 100%, 37%, 0.2);
}

dialog div {
  display: flex;
  flex-direction: column;
  width: 100%;
  gap: 8px;
}

dialog input,
dialog textarea {
  border: 2px solid #8c8c8c;
  background-color: transparent;
  color: #e6e6e6;
}

dialog::backdrop {
  background-color: rgba(0, 0, 0, 0.7);
}

input,
textarea {
  outline: none;
  width: 100%;
  min-height: 100%;
  border: 0;
  padding: 9px;
  font-family: inherit;
  border: 2px solid rgba(0, 0, 0, 0.3);
  color: rgba(0, 0, 0, 0.75);
}

textarea {
  max-width: 100%;
  max-height: 100%;
  resize: vertical;
}
</style>
