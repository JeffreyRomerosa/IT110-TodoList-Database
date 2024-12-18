<template>
    <div class="todo-container">
        <h1 class="todo-title"><i class="fa fa-list"></i> Todo List</h1>

        <!-- Form to add a new task -->
        <form @submit.prevent="addTask" class="todo-form">
            <div class="form-group">
                <input
                    v-model="title"
                    type="text"
                    name="title"
                    placeholder="Enter a task (max 50 characters)"
                    class="form-control"
                />
            </div>
            <button type="submit" class="btn btn-primary add-btn">Add Task</button>
        </form>

        <!-- Display tasks -->
        <ul class="task-list">
            <li
                v-for="task in tasks"
                :key="task.id"
                class="task-item"
            >
                <div class="task-content">
                    <!-- Checkbox to mark the task as completed -->
                    <input
                        type="checkbox"
                        v-model="task.completed"
                        @change="updateTask(task, true)"
                    />

                    <!-- Show task title or input field when editing -->
                    <span
                        v-if="!task.isEditing"
                        :class="{ completed: task.completed }"
                    >
                        {{ task.title }}
                    </span>
                    <input
                        v-else
                        v-model="task.title"
                        type="text"
                        class="edit-input"
                        @blur="toggleEditMode(task)"
                        @keyup.enter="toggleEditMode(task)"
                    />
                </div>

                <!-- Action buttons -->
                <div class="button-container">
                    <button
                        class="update-btn"
                        @click="toggleEditMode(task)"
                    >
                        <i class="fa" :class="task.isEditing ? 'fa-save' : 'fa-edit'"></i>
                    </button>
                    <button
                        class="delete-btn"
                        @click="deleteTask(task.id)"
                    >
                        <i class="fa fa-trash"></i>
                    </button>
                </div>
            </li>
        </ul>
    </div>
</template>

<script>
import axios from "axios";

export default {
    data() {
        return {
            title: "", // Stores the title for the new task
            tasks: []  // Stores the list of tasks
        };
    },
    mounted() {
        this.fetchTasks(); // Fetch tasks when the component mounts
    },
    methods: {
        fetchTasks() {
            axios
                .get("http://localhost:8000/api/tasks")
                .then((response) => {
                    this.tasks = response.data || []; // Ensure tasks is always an array
                })
                .catch((error) =>
                    console.error("Error fetching tasks:", error)
                );
        },
        addTask() {
            const maxLength = 50;
            const validTitle = /^[a-zA-Z\s]+$/;

            if (this.title.trim() === "") {
                alert("Task title cannot be empty!");
                return;
            }

            if (this.title.length > maxLength) {
                alert(`Task title cannot exceed ${maxLength} characters`);
                return;
            }

            if (!validTitle.test(this.title)) {
                alert("Task title can only contain letters and spaces");
                return;
            }

            axios
                .post("http://localhost:8000/api/tasks", { title: this.title })
                .then((response) => {
                    this.tasks.push({
                        ...response.data,
                        isEditing: false,
                        completed: false
                    });
                    this.title = ""; // Clear the input field
                })
                .catch((error) =>
                    console.error("Error adding task:", error)
                );
        },
        updateTask(task, isCheckboxChange = false) {
            const updatedTask = {
                title: task.title,
                completed: task.completed
            };

            axios
                .put(`http://localhost:8000/api/tasks/${task.id}`, updatedTask)
                .then((response) => {
                    const index = this.tasks.findIndex((t) => t.id === task.id);
                    if (index !== -1) {
                        this.tasks[index] = {
                            ...response.data,
                            isEditing: false
                        };
                    }
                    if (!isCheckboxChange) {
                        alert("Task updated successfully!");
                    }
                })
                .catch((error) =>
                    console.error("Error updating task:", error)
                );
        },
        toggleEditMode(task) {
            if (task.isEditing) {
                if (task.title.trim() === "") {
                    alert("Task title cannot be empty!");
                    return;
                }
                this.updateTask(task);
            }
            task.isEditing = !task.isEditing;
        },
        deleteTask(id) {
            if (confirm("Are you sure you want to delete this task?")) {
                axios
                    .delete(`http://localhost:8000/api/tasks/${id}`)
                    .then(() => {
                        this.tasks = this.tasks.filter(
                            (task) => task.id !== id
                        );
                    })
                    .catch((error) =>
                        console.error("Error deleting task:", error)
                    );
            }
        }
    }
};
</script>

<style scoped>
/* Main container */
.todo-container {
    max-width: 600px;
    margin: 2rem auto;
    background: #fef9ef; /* Soft bright background */
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.todo-container:hover {
    transform: scale(1.02);
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

/* Title */
.todo-title {
    text-align: center;
    color: #ff6f61; /* Soft bright coral */
    font-size: 2rem;
    margin-bottom: 20px;
    transition: color 0.3s ease;
}

.todo-title:hover {
    color: #ff9478; /* Slightly brighter coral */
}

/* Form styling */
.todo-form {
    display: flex;
    flex-wrap: nowrap;
    gap: 10px;
    margin-bottom: 20px;
    justify-content: space-between;
    align-items: center;
}

.form-control {
    flex: 1;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 8px; /* Rounded corners */
    font-size: 1rem;
    transition: box-shadow 0.3s ease, border-color 0.3s ease;
}

.form-control:focus {
    outline: none;
    box-shadow: 0 0 8px rgba(255, 111, 97, 0.5); /* Coral shadow on focus */
    border-color: #ff6f61; /* Coral border on focus */
}

.add-btn {
    background: #ff6f61; /* Soft bright coral */
    color: #fff;
    border: none;
    padding: 10px 20px;
    border-radius: 8px; /* Rounded corners */
    cursor: pointer;
    font-size: 1rem;
    font-weight: bold;
    transition: transform 0.3s ease, background-color 0.3s ease;
}

.add-btn:hover {
    background: #ff9478; /* Slightly brighter coral */
    transform: scale(1.05);
}

/* Task list */
.task-list {
    list-style: none;
    padding: 0;
    margin: 0;
}

.task-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #ffffff; /* White background for tasks */
    padding: 10px 20px;
    border-radius: 8px; /* Rounded corners */
    margin-bottom: 10px;
    border: 1px solid #ddd; /* Soft grey border */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s ease, background-color 0.3s ease, border-color 0.3s ease;
}

.task-item:hover {
    transform: scale(1.02);
    background-color: #fce4dc; /* Light coral background */
    border-color: #ff9478; /* Lighter coral border on hover */
}

/* Task content */
.task-content {
    display: flex;
    align-items: center;
    gap: 10px;
}

input[type="checkbox"] {
    transform: scale(1.2);
    width: 20px;
    height: 20px;
    border: 1px solid #ddd;
    border-radius: 50%; /* Circular checkbox */
    appearance: none;
    outline: none;
    transition: background-color 0.3s ease, box-shadow 0.3s ease;
    cursor: pointer;
}

input[type="checkbox"]:checked {
    background-color: #ff6f61; /* Coral when checked */
    box-shadow: 0 0 5px rgba(255, 111, 97, 0.5);
}

input[type="checkbox"]:hover {
    background-color: #ffe0db; /* Light coral on hover */
    box-shadow: 0 0 4px rgba(0, 0, 0, 0.2);
}

.completed {
    text-decoration: line-through;
    color: #888;
    transition: color 0.3s ease;
}

/* Edit input */
.edit-input {
    flex: 1;
    padding: 8px;
    font-size: 1rem;
    border-radius: 8px;
    border: 1px solid #ddd;
    transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.edit-input:focus {
    outline: none;
    border-color: #4caf50; /* Green border on focus */
    box-shadow: 0 0 8px rgba(76, 175, 80, 0.5); /* Green shadow on focus */
}

/* Buttons */
.button-container {
    display: flex;
    gap: 10px;
}

.update-btn,
.delete-btn {
    background: transparent;
    border: none;
    cursor: pointer;
    font-size: 1.2rem;
    transition: transform 0.3s ease, color 0.3s ease;
}

.update-btn:hover {
    transform: scale(1.1);
    color: #4caf50; /* Green for edit/save button */
}

.delete-btn:hover {
    transform: scale(1.1);
    color: #f44336; /* Red for delete button */
}

</style>
