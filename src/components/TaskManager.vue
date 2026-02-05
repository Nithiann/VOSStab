<script setup>
import { ref, onMounted, watch } from 'vue';

const props = defineProps(['isOpen']);
const emit = defineEmits(['close']);

const tasks = ref([]);
const newTaskText = ref('');

onMounted(() => {
  const savedTasks = localStorage.getItem('vosstab-tasks');
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks);
  }
});

watch(tasks, (newTasks) => {
  localStorage.setItem('vosstab-tasks', JSON.stringify(newTasks));
}, { deep: true });

const addTask = () => {
  if (newTaskText.value.trim()) {
    tasks.value.push({
      id: Date.now(),
      text: newTaskText.value.trim(),
      completed: false
    });
    newTaskText.value = '';
  }
};

const removeTask = (id) => {
  tasks.value = tasks.value.filter(t => t.id !== id);
};

const completeChecked = () => {
  tasks.value = tasks.value.filter(t => !t.completed);
};
</script>

<template>
  <div class="task-manager-popover" :class="{ open: isOpen }">
    <div class="header">
      <h3><i class="fas fa-tasks"></i> Tasks</h3>
      <button class="close-btn" @click="$emit('close')">&times;</button>
    </div>

    <div class="input-container">
      <input 
        type="text" 
        v-model="newTaskText" 
        @keyup.enter="addTask" 
        placeholder="Add a new task..."
        class="task-input"
      >
      <button @click="addTask" class="add-btn">
        <i class="fas fa-plus"></i>
      </button>
    </div>

    <div class="task-list-container">
      <ul v-if="tasks.length > 0" class="task-list">
        <li v-for="task in tasks" :key="task.id" class="task-item">
          <label class="task-label">
            <input type="checkbox" v-model="task.completed">
            <span class="checkmark"></span>
            <span class="task-text" :class="{ completed: task.completed }">{{ task.text }}</span>
          </label>
          <button @click="removeTask(task.id)" class="delete-btn">
            <i class="fas fa-trash"></i>
          </button>
        </li>
      </ul>
      <div v-else class="empty-state">
        No tasks yet!
      </div>
    </div>

    <div class="footer" v-if="tasks.some(t => t.completed)">
      <button @click="completeChecked" class="complete-all-btn">
        Complete Checked Tasks
      </button>
    </div>
  </div>
</template>

<style scoped>
.task-manager-popover {
  position: fixed;
  top: 70px;
  left: 20px;
  width: 300px;
  max-height: 500px;
  background: var(--glass-bg);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid var(--glass-border);
  border-radius: 16px;
  box-shadow: var(--glass-shadow);
  z-index: 1000;
  display: flex;
  flex-direction: column;
  padding: 16px;
  transform: translateY(-10px);
  opacity: 0;
  visibility: hidden;
  transition: all 0.3s ease;
}

.task-manager-popover.open {
  transform: translateY(0);
  opacity: 1;
  visibility: visible;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.header h3 {
  margin: 0;
  font-weight: 500;
  color: var(--on-background);
  font-size: 1.1rem;
}

.header h3 i {
  color: var(--primary-color);
  margin-right: 8px;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.5rem;
  color: var(--on-background);
  cursor: pointer;
  opacity: 0.6;
}

.close-btn:hover {
  opacity: 1;
}

.input-container {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
}

.task-input {
  flex: 1;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid var(--glass-border);
  border-radius: 8px;
  padding: 8px 12px;
  color: var(--on-background);
  outline: none;
}

.task-input:focus {
  border-color: var(--primary-color);
}

.add-btn {
  background: var(--primary-color);
  color: var(--on-primary);
  border: none;
  border-radius: 8px;
  width: 36px;
  height: 36px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s;
}

.add-btn:hover {
  transform: scale(1.05);
}

.task-list-container {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 16px;
  max-height: 300px;
}

.task-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid var(--glass-border);
}

.task-item:last-child {
  border-bottom: none;
}

.task-label {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  flex: 1;
  position: relative;
}

.task-label input {
  display: none;
}

.checkmark {
  width: 18px;
  height: 18px;
  border: 2px solid var(--primary-color);
  border-radius: 4px;
  position: relative;
  transition: all 0.2s;
}

.task-label input:checked + .checkmark {
  background: var(--primary-color);
}

.checkmark:after {
  content: "";
  position: absolute;
  display: none;
  left: 5px;
  top: 1px;
  width: 4px;
  height: 8px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

.task-label input:checked + .checkmark:after {
  display: block;
}

.task-text {
  color: var(--on-background);
  font-size: 0.95rem;
  transition: opacity 0.2s;
}

.task-text.completed {
  text-decoration: line-through;
  opacity: 0.5;
}

.delete-btn {
  background: none;
  border: none;
  color: #ff4d4d;
  cursor: pointer;
  opacity: 0;
  transition: opacity 0.2s;
  padding: 4px;
}

.task-item:hover .delete-btn {
  opacity: 0.6;
}

.delete-btn:hover {
  opacity: 1 !important;
}

.empty-state {
  text-align: center;
  color: var(--on-background);
  opacity: 0.5;
  padding: 20px 0;
  font-size: 0.9rem;
}

.footer {
  padding-top: 12px;
  border-top: 1px solid var(--glass-border);
}

.complete-all-btn {
  width: 100%;
  padding: 10px;
  background: var(--primary-color);
  color: var(--on-primary);
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: opacity 0.2s;
}

.complete-all-btn:hover {
  opacity: 0.9;
}

/* Custom Scrollbar */
.task-list-container::-webkit-scrollbar {
  width: 6px;
}

.task-list-container::-webkit-scrollbar-track {
  background: transparent;
}

.task-list-container::-webkit-scrollbar-thumb {
  background: var(--glass-border);
  border-radius: 10px;
}

.task-list-container::-webkit-scrollbar-thumb:hover {
  background: var(--primary-color);
}
</style>
