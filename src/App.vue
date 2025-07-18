<template>
  <div>
    <h1>To Do List - Vue</h1>

    <!-- Input for entering a task -->
    <input
      type="text"
      v-model="task"
      placeholder="Enter task"
    />

    <!-- Input for searching tasks -->
    <input
      type="text"
      v-model="searchQuery"
      placeholder="Search tasks"
    />

    <!-- Checkbox for filtering completed tasks -->
    <label>
      <input type="checkbox" v-model="showCompleted" />
      Show only completed tasks
    </label>

    <!-- Button to add a new task -->
    <button @click="addTask">Add</button>

    <!-- List of tasks -->
    <ul>
      <li v-for="(t, i) in filteredTasks" :key="i" style="margin-bottom: 0.5rem">

        <!-- Show input if the current index is in edit mode -->
        <template v-if="editIndex === i">
          <input
            v-model="editValue"
            @blur="saveEdit(i)"
            @keydown.enter="saveEdit(i)"
            autofocus
          />
        </template>

        <!-- Otherwise show task text, clickable to enter edit mode -->
        <template v-else>
          <span
            @click="() => {
              editIndex = i;
              editValue = t.text;
            }"
            :style="{
              textDecoration: t.completed ? 'line-through' : 'none',
              cursor: 'pointer',
              marginRight: '10px'
            }"
          >
            {{ t.text }}
          </span>
        </template>

        <!-- Task creation date -->
        <span style="font-size: 0.8rem; color: gray; margin-right: 10px;">
          {{ new Date(t.createdAt).toLocaleDateString() }}
        </span>

        <!-- Button to toggle completed state -->
        <button @click="toggleCompleted(i)">✔</button>

        <!-- Button to delete the task -->
        <button @click.stop="removeTask(i)">Delete</button>
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue';

// Define Task type
type Task = {
  text: string;
  completed: boolean;
  createdAt: number;
};

const task = ref("");            // State for current task input
const tasks = ref<Task[]>([]);   // State array for all tasks

// Load tasks from localStorage on component mount
onMounted(() => {
  try {
    const stored = localStorage.getItem("tasks");
    if (stored) {
      tasks.value = JSON.parse(stored);
    }
  } catch (error) {
    console.error("Error loading tasks from localStorage:", error);
    localStorage.removeItem("tasks"); // Clear corrupted data
  }
});

// Watch tasks array for changes to save in localStorage
watch(
  tasks,
  (newTasks) => {
    localStorage.setItem("tasks", JSON.stringify(newTasks));
  },
  { deep: true }
);

const showCompleted = ref(false);         // Whether to show only completed tasks
const editIndex = ref<number | null>(null); // Index of task currently in edit mode
const editValue = ref<string>("");          // Current edit input value
const searchQuery = ref<string>("");        // Search input

// Add a new task to the list
function addTask() {
  if (task.value.trim()) {
    tasks.value.push({
      text: task.value.trim(),
      completed: false,
      createdAt: Date.now(),
    });
    task.value = "";
  }
}

// Remove a task by index
function removeTask(index: number) {
  tasks.value.splice(index, 1);
}

// Toggle the completed state of a task
function toggleCompleted(index: number) {
  tasks.value[index].completed = !tasks.value[index].completed;
}

// Save the edited task text and exit edit mode
function saveEdit(index: number) {
  if (editValue.value.trim()) {
    tasks.value[index].text = editValue.value.trim();
    editIndex.value = null;
    editValue.value = "";
  }
}

// Compute filtered tasks based on completion filter and search query
const filteredTasks = computed(() => {
  const filtered = showCompleted.value
    ? tasks.value.filter(t => t.completed)
    : tasks.value;

  return filtered.filter(t =>
    t.text.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});
</script>

<style>
body {
  margin: 0;
  font-family: sans-serif;
  padding: 2rem;
}
</style>
