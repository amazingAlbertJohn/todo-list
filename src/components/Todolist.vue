<template>
  <v-app>
    <!-- Title -->
    <v-card-title class="text-center">
      <h1>TO DO LIST</h1>

      
    </v-card-title>
    <v-container fluid fill-height>
      <v-row class="fill-height" align="center" justify="center">
        <v-col cols="12" md="5">
          <!-- Outer Card with Glass Effect -->
          <v-card class="glass-card fill-height">

            <!-- Task Table -->
            <v-data-table
              :headers="headers"
              :items="tasks"
              item-key="note"
              class="pa-4"
              hide-default-footer
            >
              <template v-slot:item="props">
                <tr :class="{ 'task-completed': props.item.isCompleted }">
                  <td>
                    <v-checkbox
                      v-model="props.item.isCompleted"
                      @change="toggleDeleteButton(props.index)"
                      hide-details
                    ></v-checkbox>
                  </td>
                  <td>
                    <span :class="{ 'text-underlined': props.item.isCompleted }">
                      {{ props.item.note }}
                    </span>
                  </td>
                  <td class="text-right">
                    <!-- Edit Button -->
                    <v-btn icon @click="openEditDialog(props.item, props.index)" class="edit-button">
                      <v-icon>mdi-pencil</v-icon>
                    </v-btn>
                    <!-- Delete Button -->
                    <v-btn
                      v-if="props.item.showDeleteButton"
                      icon
                      @click="deleteTask(props.index)"
                      class="delete-button"
                    >
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>
                  </td>
                </tr>
              </template>
            </v-data-table>
          </v-card>
        </v-col>
      </v-row>
    </v-container>

    <!-- Task Counts -->
    <v-row justify="center" class="mt-4 text-center">
      <v-col cols="10" md="5">
        <v-card class="pa-4 glass-card">
          <v-row>
            <v-col>
              <v-chip color="green" outlined>
                Finished Tasks: {{ finishedCount }}
              </v-chip>
            </v-col>
            <v-col>
              <v-chip color="red" outlined>
                Unfinished Tasks: {{ unfinishedCount }}
              </v-chip>
            </v-col>
          </v-row>
        </v-card>
      </v-col>
    </v-row>

    <!-- Add Task Button -->
    <v-row justify="center">
      <v-col cols="10" md="5" mt="none" class="text-center">
        <v-btn color="primary" @click="dialog = true">Add Task</v-btn>
      </v-col>
    </v-row>

    <!-- Task Form in Dialog -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title class="headline">Add New Task</v-card-title>
        <v-card-text>
          <v-form @submit.prevent="addTask">
            <v-textarea
              v-model="newTask.note"
              label="Write task"
              rows="2"
              required
              class="mb-4"
            ></v-textarea>
            <v-btn type="submit" color="primary">Add</v-btn>
          </v-form>
        </v-card-text>
      </v-card>
    </v-dialog>

    <!-- Edit Task Dialog -->
    <v-dialog v-model="editDialog" max-width="500px">
      <v-card>
        <v-card-title class="headline">Edit Task</v-card-title>
        <v-card-text>
          <v-form @submit.prevent="updateTask">
            <v-textarea
              v-model="currentTask.note"
              label="Edit task"
              rows="2"
              required
              class="mb-4"
            ></v-textarea>
            <v-btn type="submit" color="primary">Save</v-btn>
          </v-form>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      tasks: [], // Initialize with an empty array
      newTask: {
        note: '' // Only include the note field
      },
      currentTask: {
        note: '', // Task being edited
        index: null // Index of the task being edited
      },
      dialog: false, // Dialog visibility state for adding tasks
      editDialog: false, // Dialog visibility state for editing tasks
      headers: [
        { text: '', value: 'checkbox' },
        { text: 'Task Note', value: 'note' },
        { text: '', value: 'actions', align: 'end' }
      ]
    };
  },
  computed: {
    finishedCount() {
      // Return the count of finished tasks
      return this.tasks.filter(task => task.isCompleted).length;
    },
    unfinishedCount() {
      // Return the count of unfinished tasks
      return this.tasks.filter(task => !task.isCompleted).length;
    }
  },
  methods: {
    addTask() {
      if (this.newTask.note.trim()) {
        this.tasks.push({ 
          note: this.newTask.note.trim(), 
          isCompleted: false,
          showDeleteButton: false // Initially hide the delete button
        });
        this.updateLocalStorage(); // Update local storage after adding a task
        this.newTask.note = ''; // Clear the input field
        this.dialog = false; // Close the dialog after adding the task
      }
    },
    deleteTask(index) {
      this.tasks.splice(index, 1);
      this.updateLocalStorage(); // Update local storage after deleting a task
    },
    toggleDeleteButton(index) {
      // Toggle delete button visibility based on the checkbox state
      this.tasks[index].showDeleteButton = this.tasks[index].isCompleted;
      this.updateLocalStorage(); // Update local storage after changing task status
    },
    openEditDialog(task, index) {
      // Open the edit dialog and set currentTask
      this.currentTask = { ...task, index };
      this.editDialog = true;
    },
    updateTask() {
      // Update the task note
      if (this.currentTask.note.trim() && this.currentTask.index !== null) {
        this.tasks[this.currentTask.index].note = this.currentTask.note.trim();
        this.updateLocalStorage(); // Update local storage after editing a task
        this.editDialog = false; // Close the edit dialog
      }
    },
    updateLocalStorage() {
      // Save tasks to local storage
      localStorage.setItem('tasks', JSON.stringify(this.tasks));
    },
    loadTasksFromLocalStorage() {
      // Load tasks from local storage
      const savedTasks = localStorage.getItem('tasks');
      if (savedTasks) {
        this.tasks = JSON.parse(savedTasks);
      }
    }
  },
  created() {
    this.loadTasksFromLocalStorage(); // Load tasks when the component is created
  }
};
</script>

<style scoped>
.task-completed {
  padding: 0;
  color: rgb(13, 255, 0);
  opacity: 0.6;
}

.text-underlined {
  text-decoration: line-through;
  color: white;
  display: inline-block; /* Ensures consistent text display */
}

.edit-button, .delete-button {
  width: 30px; /* Adjust width as needed */
  height: 30px; /* Adjust height as needed */
  font-size: 15px; /* Adjust font size as needed */
  padding: 0; /* Remove padding if needed */
  color: #eeeeee;
  box-shadow: none; 
  background: none; 
}

.delete-button{
  width: 30px; /* Adjust width as needed */
  height: 30px; /* Adjust height as needed */
  font-size: 15px; /* Adjust font size as needed */
  padding: 0; /* Remove padding if needed */
  color: #f40606;
  box-shadow: none; 
  background: none; 
}

.glass-card {
  background: rgba(255, 255, 255, 0.164);
  border-radius: 16px;
  backdrop-filter: blur(5px);
  -webkit-backdrop-filter: blur(5px);
  border: 1px solid rgba(255, 255, 255, 0.77);
}

.fill-height {
  height: 90%;
}

.v-container {
  height: 90vh;
}

.v-row {
  height: 100%;
  margin-bottom: none;
}

.text-center {
  text-align: center;
}

h1 {
  margin: 0;
  font-size: 24px; /* Adjust font size as needed */
  color: #ffffff; /* Adjust text color if needed */
}
</style>
