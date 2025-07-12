<script setup>
import { ref, onMounted, watch } from 'vue'

const newReminderText = ref('')
const newReminderDate = ref('')
const reminders = ref([])
const STORAGE_KEY = 'vue-reminders-app'

// Load reminders from localStorage when the component is mounted
onMounted(() => {
  reminders.value = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')

  // Register service worker for PWA
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker
        .register('/sw.js')
        .then((registration) => {
          console.log('Service Worker registered: ', registration)
        })
        .catch((registrationError) => {
          console.log('Service Worker registration failed: ', registrationError)
        })
    })
  }
})

// Watch for changes in reminders and save to localStorage
watch(
  reminders,
  (newReminders) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(newReminders))
  },
  { deep: true },
)

function addReminder() {
  if (newReminderText.value.trim() && newReminderDate.value) {
    reminders.value.push({
      id: Date.now(),
      text: newReminderText.value,
      date: newReminderDate.value,
    })
    newReminderText.value = ''
    newReminderDate.value = ''
  }
}

function removeReminder(id) {
  reminders.value = reminders.value.filter((r) => r.id !== id)
}
</script>

<template>
  <div id="app-container">
    <header>
      <h1>Date Reminder App</h1>
    </header>
    <main>
      <form @submit.prevent="addReminder">
        <input
          type="text"
          v-model="newReminderText"
          placeholder="What do you need to remember?"
          required
        />
        <input type="date" v-model="newReminderDate" required />
        <button type="submit">Add Reminder</button>
      </form>

      <div class="reminders-list">
        <h2>Your Reminders</h2>
        <ul v-if="reminders.length > 0">
          <li v-for="reminder in reminders" :key="reminder.id">
            <span
              >{{ reminder.text }} - <strong>{{ reminder.date }}</strong></span
            >
            <button @click="removeReminder(reminder.id)">Remove</button>
          </li>
        </ul>
        <p v-else>You have no reminders yet.</p>
      </div>
    </main>
  </div>
</template>

<style scoped>
@keyframes text-pulse {
  0% {
    text-shadow: 0 0 5px rgba(66, 185, 131, 0.7);
  }
  50% {
    text-shadow: 0 0 10px rgba(66, 185, 131, 0.9);
  }
  100% {
    text-shadow: 0 0 5px rgba(66, 185, 131, 0.7);
  }
}

@keyframes scanline {
  0% {
    top: -10%;
  }
  100% {
    top: 110%;
  }
}

#app-container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 2rem;
  font-family: 'Press Start 2P', monospace;
  letter-spacing: -1px;
  color: #42b983;
  background-color: #0a0a0a;
  border: 2px solid #42b983;
  background-image: radial-gradient(rgba(66, 185, 131, 0.05) 1px, transparent 1px);
  background-size: 3px 3px;
  border-radius: 8px;
  box-shadow: 0 0 15px rgba(66, 185, 131, 0.5);
  text-shadow: 0 0 5px rgba(66, 185, 131, 0.7);
  position: relative;
  overflow: hidden;
  animation: text-pulse 3s ease-in-out infinite;
}

#app-container::before {
  content: ' ';
  display: block;
  position: absolute;
  left: 0;
  width: 100%;
  height: 2px;
  background: #42b983;
  opacity: 0.25;
  box-shadow: 0 0 5px rgba(66, 185, 131, 0.4);
  z-index: 3;
  animation: scanline 6s linear infinite;
}

#app-container::after {
  content: ' ';
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background:
    linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%),
    linear-gradient(90deg, rgba(255, 0, 0, 0.06), rgba(0, 255, 0, 0.02), rgba(0, 0, 255, 0.06));
  z-index: 1;
  background-size:
    100% 4px,
    3px 100%;
  pointer-events: none;
}

header {
  text-align: center;
  margin-bottom: 2rem;
  position: relative;
  z-index: 2;
}

h1,
h2 {
  color: #42b983;
  text-transform: uppercase;
}

h1 {
  font-size: 1.5rem;
}
h2 {
  font-size: 1.2rem;
}

form {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-bottom: 2rem;
  position: relative;
  z-index: 2;
}

input[type='text'] {
  flex: 1 1 300px;
}

input,
button {
  background-color: transparent;
  border: 1px solid #42b983;
  color: #42b983;
  font-family: inherit;
  font-size: 0.8rem;
  padding: 0.75rem 1rem;
  border-radius: 4px;
  box-sizing: border-box;
}

input::placeholder {
  color: #36a374;
  opacity: 0.6;
}

input[type='date']::-webkit-calendar-picker-indicator {
  cursor: pointer;
  /* This filter converts the black icon to the theme's green color */
  filter: invert(40%) sepia(100%) saturate(1000%) hue-rotate(90deg) brightness(90%) contrast(100%);
}

button {
  cursor: pointer;
  transition:
    background-color 0.2s,
    color 0.2s;
}

button:hover {
  background-color: #36a374;
  color: #0a0a0a;
}

.reminders-list ul {
  list-style: none;
  padding: 0;
  position: relative;
  z-index: 2;
}

.reminders-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  background-color: rgba(66, 185, 131, 0.1);
  border: 1px solid #36a374;
  border-radius: 4px;
  margin-bottom: 0.5rem;
  font-size: 0.8rem;
}

.reminders-list li button {
  background-color: transparent;
  border-color: #e74c3c;
  color: #e74c3c;
}

.reminders-list li button:hover {
  background-color: #e74c3c;
  color: #0a0a0a;
}

p {
  text-align: center;
  position: relative;
  z-index: 2;
}
</style>
