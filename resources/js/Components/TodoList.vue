<template>
    <div class="todo-container">
        <h1 class="todo-title"><i class="fa fa-list"></i> Todo List</h1>

        <!-- Form to add a new task -->
        <form @submit.prevent="addTask">
            <div class="form-group">
                <input 
                    v-model="title" 
                    type="text" 
                    name="title" 
                    placeholder="Task Title" 
                    class="form-control"
                >
            </div>
            <div class="form1">
                <input type="submit" value="Add Task" class="btn btn-info form">
            </div>
        </form>

        <!-- Display tasks -->
        <ul class="task-list">
            <li v-for="task in tasks" :key="task.id" class="task-item">
                <!-- Flex container for checkbox and task title -->
                <div class="task-content">
                    <!-- Checkbox to mark the task as completed or incomplete -->
                    <input 
                        type="checkbox" 
                        v-model="task.completed" 
                        @change="updateTask(task, true)" 
                    >

                    <!-- Conditionally show input field for editing or task title -->
                    <span v-if="!task.isEditing" :class="{ completed: task.completed }">{{ task.title }}</span>
                    <input 
                        v-if="task.isEditing" 
                        v-model="task.title" 
                        type="text" 
                        class="edit-input"
                        @blur="updateTask(task)"
                    >
                </div>

                <!-- Button container for update and delete buttons -->
                <div class="button-container">
                    <!-- Update button with Font Awesome pencil icon -->
                    <button class="update-btn" @click="toggleEditMode(task)">
                        <i class="fa" :class="task.isEditing ? 'fa-save' : 'fa-edit'"></i>
                    </button>
                    <!-- Delete button with Font Awesome trash icon -->
                    <button class="delete-btn" @click="deleteTask(task.id)">
                        <i class="fa fa-trash"></i>
                    </button>
                </div>
            </li>
        </ul>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            title: '',  // Stores the title for the new task
            tasks: []   // Stores the list of tasks
        };
    },
    mounted() {
        this.fetchTasks(); // Fetch tasks when the component mounts
    },
    methods: {
        // Fetch all tasks from the API
        fetchTasks() {
            axios.get('http://localhost:8000/api/tasks')
                .then(response => {
                    this.tasks = response.data;
                })
                .catch(error => console.error('Error fetching tasks:', error));
        },

        // Add a new task
        addTask() {

            const validTitle = /^[a-zA-Z\s]+$/;
           // const validTitle = /^[a-zA-Z0-9\s]*$/; 

            if (this.title.trim() === ''){
                alert('Task title cannot be empty!');
                return;
            }

            if(!validTitle.test(this.title)){
                alert('Task title can only contain letters and spaces');
                return;
            }

            axios.post('http://localhost:8000/api/tasks', { title: this.title })
                .then(response => {
                    this.tasks.push(response.data); // Add the task to the local list
                    this.title = ''; // Clear the input field
                    alert('Task successfully added to the list!');
                })
                .catch(error => console.error('Error adding task:', error));
        },

        // Update a task's completion status or title
        updateTask(task, isCheckboxChange = false) {
            const updatedTask = {
                title: task.title,
                completed: task.completed
            };

            axios.put(`http://localhost:8000/api/tasks/${task.id}`, updatedTask)
                .then(response => {
                    // Update the task locally
                    const index = this.tasks.findIndex(t => t.id === task.id);
                    if (index !== -1) {
                        this.tasks[index] = response.data;
                    }

                    // Only show the alert if it's not a checkbox change
                    if (!isCheckboxChange) {
                        alert('Task updated successfully!');
                    }
                })
                .catch(error => console.error('Error updating task:', error));
        },

        // Toggle edit mode for a task
        toggleEditMode(task) {
            if (task.isEditing) {
                // If in editing mode, update the task
                this.updateTask(task);
            }
            // Toggle the editing state
            task.isEditing = !task.isEditing;
        },

        // Delete a task
        deleteTask(id) {
            if (confirm('Are you sure you want to delete this task?')) {
                axios.delete(`http://localhost:8000/api/tasks/${id}`)
                    .then(() => {
                        // Remove the task from the local list after successful deletion
                        this.tasks = this.tasks.filter(task => task.id !== id);
                        alert('Task deleted successfully!');
                    })
                    .catch(error => console.error('Error deleting task:', error));
            } else {
                alert('Task deletion cancelled.');
            }
        }
    }
};
</script>

<style scoped>
/* Styles for container, form, task list, etc. */
.todo-container {
    background-color: gray;
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
}
.todo-title {
    text-align: center;
    font-size: 2em;
    margin-bottom: 20px;
    color: #333;
}
.form-group {
    margin-bottom: 10px;
}

.form1{
    text-align: center;
}
.form:hover {
    background-color: rgb(53, 58, 58);
}
.form-control {
    width: 100%;
    padding: 10px;
    font-size: 1em;
    border-radius: 4px;
    border: 1px solid #ddd;
}
.btn-info {
    background-color: #4CAF50;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
}
.task-list {
    list-style: none;
    padding: 0;
    margin: 20px 0;
}
.task-item {
    display: flex;
    justify-content: space-between; /* Space between task content and buttons */
    align-items: center;
    padding: 10px;
    background-color: #fff;
    border-radius: 4px;
    margin-bottom: 10px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Container for checkbox and task title */
.task-content {
    margin-top: 10px;
    display: flex;
    align-items: center; /* Align checkbox and title in a row */
}

/* Styling for the checkbox */
input[type="checkbox"] {
    margin-right: 10px; /* Space between the checkbox and task title */
    transform: scale(1.2); /* Make the checkbox slightly larger */
}

/* Line-through style when the task is completed */
.completed {
    text-decoration: line-through;
    color: #888;
}

/* Flexbox container for buttons (update and delete) */
.button-container {
    display: flex;
    gap: 10px; /* Space between the buttons */
}

/* Styling for the buttons */
.delete-btn, .update-btn {
    padding: 6px 12px;
    border-radius: 4px;
    cursor: pointer;
    border: none;
    font-size: 14px;
}

/* Specific styles for Delete button */
.delete-btn {
    background-color: #ff4d4f;
    color: white;
}

/* Specific styles for Update button */
.update-btn {
    background-color: #4CAF50;
    color: white;
}

/* Styling for the input field while editing */
.edit-input {
    flex: 1; /* Allow the input to take up remaining space */
    padding: 8px;
    font-size: 1em;
    border-radius: 4px;
    border: 1px solid #ddd;
}
</style>
