<template>
  <div class="container py-4">
    <h1 class="mb-3">Vue.js User Registration</h1>

    <!-- Registration Form -->
    <form @submit.prevent="submitForm" class="card p-3 mb-4" novalidate>
      <!-- Name -->
      <div class="form-group mb-3">
        <label for="name">Name</label>
        <input v-model="formData.name" type="text" class="form-control" id="name" placeholder="Enter name" required />
        <small v-if="empties.name" class="text-warning">
          {{ empties.name }}
        </small>
        <small v-else-if="errors.name" class="text-danger">
          {{ errors.name }}
        </small>
      </div>

      <!-- Email -->
      <div class="form-group mb-3">
        <label for="email">Email</label>
        <input v-model="formData.email" type="email" class="form-control" id="email" placeholder="Enter email"
          required />
        <small v-if="empties.email" class="text-warning">
          {{ empties.email }}
        </small>
        <small v-else-if="errors.email" class="text-danger">
          {{ errors.email }}
        </small>
      </div>

      <!-- Password -->
      <div class="form-group mb-3">
        <label for="password">Password</label>
        <input v-model="formData.password" type="password" class="form-control" id="password"
          placeholder="Enter password" required />
        <small v-if="empties.password" class="text-warning">
          {{ empties.password }}
        </small>
        <small v-else-if="errors.password" class="text-danger">
          {{ errors.password }}
        </small>
      </div>

      <!-- Age -->
      <div class="form-group mb-3">
        <label for="age">Age</label>
        <input v-model.number="formData.age" type="number" class="form-control" id="age" placeholder="Enter age"
          required />
        <small v-if="empties.age" class="text-warning">
          {{ empties.age }}
        </small>
        <small v-else-if="errors.age" class="text-danger">
          {{ errors.age }}
        </small>
      </div>

      <button class="btn btn-primary" type="submit" :disabled="loading">
        Submit
      </button>
    </form>

    <!-- Success Message -->
    <div v-if="submitted && fadeOut" class="alert alert-success">
      Registration successful!
    </div>


    <div v-if="loadingUsers && submitted" class="text-center">
      <div class="spinner-border" role="status"></div>
      <p>Loading users…</p>
    </div>

    <transition name="fade" mode="out-in">
      <div v-if="submitted">
        <h1>User List</h1>
        <ul v-if="users.length" key="user-list" class="list-group">
          <li v-for="user in users" :key="user.id" class="list-group-item">
            <strong>{{ user.name }}</strong> - {{ user.email }}
          </li>
        </ul>
      </div>
    </transition>
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
    // Validate the form fields and separate empty errors from other errors.
    validateForm() {
      // Reset error objects.
      this.empties = {}
      this.errors = {}

      // Validate Name
      if (!this.formData.name || !this.formData.name.trim()) {
        this.empties.name = 'Name is required.'
      } else if (this.formData.name.trim().length < 3) {
        this.errors.name = 'Name must be at least 3 characters.'
      }

      // Validate Email
      if (!this.formData.email || !this.formData.email.trim()) {
        this.empties.email = 'Email is required.'
      } else {
        const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!emailPattern.test(this.formData.email.trim())) {
          this.errors.email = 'Please enter a valid email address.'
        }
      }

      // Validate Password
      if (!this.formData.password || !this.formData.password.trim()) {
        this.empties.password = 'Password is required.'
      } else if (this.formData.password.trim().length < 6) {
        this.errors.password = 'Password must be at least 6 characters.'
      }

      // Validate Age
      if (this.formData.age === '' || this.formData.age === null) {
        this.empties.age = 'Age is required.'
      } else if (isNaN(this.formData.age)) {
        this.errors.age = 'Age must be a valid number.'
      } else if (this.formData.age <= 18) {
        this.errors.age = 'Age must be greater than 18.'
      }

      // Return true if there are no errors in either object.
      return Object.keys(this.empties).length === 0 && Object.keys(this.errors).length === 0
    },

    // Submit form data
    async submitForm() {
      // Clear any previous success indicator.
      this.submitted = false
      this.fadeOut = true;


      // Run validations.
      if (!this.validateForm()) {
        return
      }

      try {
        this.loading = true

        // Simulated POST request.
        const response = await axios.post(
          'https://jsonplaceholder.typicode.com/users',
          { ...this.formData }
        )
        console.log('POST response:', response.data)

        // Mark submission as successful.
        this.submitted = true

        // Reset form.
        this.formData = {
          name: '',
          email: '',
          password: '',
          age: ''
        }

        // Optionally re-fetch user list.
        this.fetchUsers()
      } catch (error) {
        console.error('Error submitting user:', error)
      } finally {
        this.loading = false;
        setTimeout(() => {
          this.fadeOut = false;
        }, 1000);

      };
    },

    // Fetch list of existing users from JSONPlaceholder.
    async fetchUsers() {
      this.loadingUsers = true
      try {
        const response = await axios.get('https://jsonplaceholder.typicode.com/users')
        this.users = response.data
      } catch (error) {
        console.error('Error fetching users:', error)
      } finally {
        this.loadingUsers = false
      }
    }
  }
}
</script>

<style scoped>
/* Simple fade transition */
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