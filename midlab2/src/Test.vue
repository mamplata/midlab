
<template>
  <div class="container py-4">
    <h1 class="mb-3">Vue.js User Registration</h1>

    <!-- Registration Form -->
    <form @submit.prevent="submitForm" class="card p-3 mb-4" novalidate>
      <!-- Name -->
      <div class="form-group mb-3">
        <label for="name">Name</label>
        <input v-model="formData.name" type="text" class="form-control" id="name" placeholder="Enter name" required />
        <small v-if="empties.name" class="text-warning">{{ empties.name }}</small>
        <small v-else-if="errors.name" class="text-danger">{{ errors.name }}</small>
      </div>

      <!-- Email -->
      <div class="form-group mb-3">
        <label for="email">Email</label>
        <input v-model="formData.email" type="email" class="form-control" id="email" placeholder="Enter email" required />
        <small v-if="empties.email" class="text-warning">{{ empties.email }}</small>
        <small v-else-if="errors.email" class="text-danger">{{ errors.email }}</small>
      </div>

      <!-- Password -->
      <div class="form-group mb-3">
        <label for="password">Password</label>
        <input v-model="formData.password" type="password" class="form-control" id="password" placeholder="Enter password" required />
        <small v-if="empties.password" class="text-warning">{{ empties.password }}</small>
        <small v-else-if="errors.password" class="text-danger">{{ errors.password }}</small>
      </div>

      <!-- Age -->
      <div class="form-group mb-3">
        <label for="age">Age</label>
        <input v-model.number="formData.age" type="number" class="form-control" id="age" placeholder="Enter age" required />
        <small v-if="empties.age" class="text-warning">{{ empties.age }}</small>
        <small v-else-if="errors.age" class="text-danger">{{ errors.age }}</small>
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

<script>
import axios from 'axios'

export default {
  name: 'App',
  data() {
    return {
      formData: {
        name: '',
        email: '',
        password: '',
        age: ''
      },

      empties: {},
      errors: {},
      submitted: false,
      users: [],
      loadingUsers: false,
      loading: false,
      fadeOut: false
    }
  },
  mounted() {
    this.fetchUsers()
  },
  methods: {
    validateForm() {
      this.empties = {}
      this.errors = {}

      if (!this.formData.name || !this.formData.name.trim()) {
        this.empties.name = 'Name is required.'
      } else if (this.formData.name.trim().length < 3) {
        this.errors.name = 'Name must be at least 3 characters.'
      }

      if (!this.formData.email || !this.formData.email.trim()) {
        this.empties.email = 'Email is required.'
      } else {
        const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!emailPattern.test(this.formData.email.trim())) {
          this.errors.email = 'Please enter a valid email address.'
        }
      }

      if (!this.formData.password || !this.formData.password.trim()) {
        this.empties.password = 'Password is required.'
      } else if (this.formData.password.trim().length < 6) {
        this.errors.password = 'Password must be at least 6 characters.'
      }

      if (this.formData.age === '' || this.formData.age === null) {
        this.empties.age = 'Age is required.'
      } else if (isNaN(this.formData.age)) {
        this.errors.age = 'Age must be a valid number.'
      } else if (this.formData.age <= 18) {
        this.errors.age = 'Age must be greater than 18.'
      }

      return Object.keys(this.empties).length === 0 && Object.keys(this.errors).length === 0
    },

    submitForm() {
      this.submitted = false
      this.fadeOut = true;

      if (!this.validateForm()) {
        return
      }

      this.loading = true

      axios.post('https://jsonplaceholder.typicode.com/users', { ...this.formData })
        .then(response => {
          console.log('POST response:', response.data)

          this.submitted = true

          this.formData = {
            name: '',
            email: '',
            password: '',
            age: ''
          }

          return this.fetchUsers()  // Fetch users after submission
        })
        .then(() => {
          setTimeout(() => {
            this.fadeOut = false;
          }, 1000);
        })
        .catch(error => {
          console.error('Error submitting user:', error)
        })
        .finally(() => {
          this.loading = false;
        })
    },

    fetchUsers() {
      this.loadingUsers = true
      return axios.get('https://jsonplaceholder.typicode.com/users')
        .then(response => {
          this.users = response.data
        })
        .catch(error => {
          console.error('Error fetching users:', error)
        })
        .finally(() => {
          this.loadingUsers = false
        })
    }
  }
}
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
