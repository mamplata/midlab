<template>
  <div class="container py-4">
    <h1 class="mb-3">Vue.js User Registration</h1>

    <!-- Registration Form -->
    <form @submit.prevent="submitForm" class="card p-3 mb-4" novalidate>
      <!-- Name -->
      <div class="form-group mb-3">
        <label for="name">Name</label>
        <input 
          v-model="formData.name" 
          type="text" 
          class="form-control" 
          :class="{ 'is-invalid': errors.name }"
          id="name" 
          placeholder="Enter name" 
          required 
        />
        <div v-if="errors.name" class="invalid-feedback">{{ errors.name }}</div>
      </div>

      <!-- Email -->
      <div class="form-group mb-3">
        <label for="email">Email</label>
        <input 
          v-model="formData.email" 
          type="email" 
          class="form-control" 
          :class="{ 'is-invalid': errors.email }"
          id="email" 
          placeholder="Enter email" 
          required 
        />
        <div v-if="errors.email" class="invalid-feedback">{{ errors.email }}</div>
      </div>

      <!-- Password -->
      <div class="form-group mb-3">
        <label for="password">Password</label>
        <input 
          v-model="formData.password" 
          type="password" 
          class="form-control" 
          :class="{ 'is-invalid': errors.password }"
          id="password" 
          placeholder="Enter password" 
          required 
        />
        <div v-if="errors.password" class="invalid-feedback">{{ errors.password }}</div>
      </div>

      <!-- Age -->
      <div class="form-group mb-3">
        <label for="age">Age</label>
        <input 
          v-model.number="formData.age" 
          type="number" 
          class="form-control" 
          :class="{ 'is-invalid': errors.age }"
          id="age" 
          placeholder="Enter age" 
          required 
        />
        <div v-if="errors.age" class="invalid-feedback">{{ errors.age }}</div>
      </div>

      <button class="btn btn-primary" type="submit" :disabled="loading">
        Submit
      </button>
    </form>

    <!-- Success Message -->
    <transition name="fade">
      <div v-if="submitted && fadeOut" class="alert alert-success">
        Registration successful!
      </div>
    </transition>

    <!-- Loading Indicator -->
    <div v-if="loadingUsers && submitted" class="text-center">
      <div class="spinner-border" role="status"></div>
      <p>Loading users…</p>
    </div>

    <!-- User List with Transition Group -->
    <div v-if="submitted">
      <h1>User List</h1>
      <transition-group name="fade" tag="ul" class="list-group">
        <li v-for="user in users" :key="user.id" class="list-group-item">
          <strong>{{ user.name }}</strong> - {{ user.email }}
        </li>
      </transition-group>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const formData = ref({
  name: '',
  email: '',
  password: '',
  age: ''
})

const errors = ref({})
const submitted = ref(false)
const users = ref([])
const loadingUsers = ref(false)
const loading = ref(false)
const fadeOut = ref(false)

const validateForm = () => {
  errors.value = {}

  if (!formData.value.name.trim()) {
    errors.value.name = 'Name is required.'
  } else if (formData.value.name.trim().length < 3) {
    errors.value.name = 'Name must be at least 3 characters.'
  }

  if (!formData.value.email.trim()) {
    errors.value.email = 'Email is required.'
  } else {
    const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!emailPattern.test(formData.value.email.trim())) {
      errors.value.email = 'Please enter a valid email address.'
    }
  }

  if (!formData.value.password.trim()) {
    errors.value.password = 'Password is required.'
  } else if (formData.value.password.trim().length < 6) {
    errors.value.password = 'Password must be at least 6 characters.'
  }

  if (formData.value.age === '' || formData.value.age === null) {
    errors.value.age = 'Age is required.'
  } else if (isNaN(formData.value.age)) {
    errors.value.age = 'Age must be a valid number.'
  } else if (formData.value.age <= 18) {
    errors.value.age = 'Age must be greater than 18.'
  }

  return Object.keys(errors.value).length === 0
}

const submitForm = () => {
  submitted.value = false
  fadeOut.value = true

  if (!validateForm()) {
    return
  }

  loading.value = true

  axios.post('https://jsonplaceholder.typicode.com/users', { ...formData.value })
    .then(response => {
      console.log('POST response:', response.data)

      submitted.value = true

      formData.value = {
        name: '',
        email: '',
        password: '',
        age: ''
      }

      return fetchUsers() // Fetch users after submission
    })
    .then(() => {
      setTimeout(() => {
        fadeOut.value = false
      }, 1000)
    })
    .catch(error => {
      console.error('Error submitting user:', error)
    })
    .finally(() => {
      loading.value = false
    })
}

const fetchUsers = () => {
  loadingUsers.value = true
  return axios.get('https://jsonplaceholder.typicode.com/users')
    .then(response => {
      users.value = response.data
    })
    .catch(error => {
      console.error('Error fetching users:', error)
    })
    .finally(() => {
      loadingUsers.value = false
    })
}

onMounted(fetchUsers)
</script>

<style scoped>
/* Fade transition styles */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Adjust the spinner spacing */
.spinner-border {
  margin: 1rem 0;
}
</style>
