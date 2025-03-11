methods: {
    // Validate the form fields and separate empty errors from other errors.
    validateForm() {
      // Reset error objects.
      this.empties = {};
      this.errors = {};

      // Validate Name
      if (!this.formData.name || !this.formData.name.trim()) {
        this.empties.name = 'Name is required.';
      } else if (this.formData.name.trim().length < 3) {
        this.errors.name = 'Name must be at least 3 characters.';
      }

      // Validate Email
      if (!this.formData.email || !this.formData.email.trim()) {
        this.empties.email = 'Email is required.';
      } else {
        const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailPattern.test(this.formData.email.trim())) {
          this.errors.email = 'Please enter a valid email address.';
        }
      }

      // Validate Password
      if (!this.formData.password || !this.formData.password.trim()) {
        this.empties.password = 'Password is required.';
      } else if (this.formData.password.trim().length < 6) {
        this.errors.password = 'Password must be at least 6 characters.';
      }

      // Validate Age
      if (this.formData.age === '' || this.formData.age === null) {
        this.empties.age = 'Age is required.';
      } else if (isNaN(this.formData.age)) {
        this.errors.age = 'Age must be a valid number.';
      } else if (this.formData.age <= 18) {
        this.errors.age = 'Age must be greater than 18.';
      }

      // Return true if there are no errors in either object.
      return Object.keys(this.empties).length === 0 && Object.keys(this.errors).length === 0;
    },

    // Submit form data
    submitForm() {
      // Clear any previous success indicator.
      this.submitted = false;
      this.fadeOut = true;

      // Run validations.
      if (!this.validateForm()) {
        return;
      }

      this.loading = true;

      // Simulated POST request.
      axios
        .post('https://jsonplaceholder.typicode.com/users', { ...this.formData })
        .then((response) => {
          console.log('POST response:', response.data);

          // Mark submission as successful.
          this.submitted = true;

          // Reset form.
          this.formData = {
            name: '',
            email: '',
            password: '',
            age: ''
          };

          // Optionally re-fetch user list.
          this.fetchUsers();
        })
        .catch((error) => {
          console.error('Error submitting user:', error);
        })
        .finally(() => {
          this.loading = false;
          setTimeout(() => {
            this.fadeOut = false;
          }, 1000);
        });
    },

    // Fetch list of existing users from JSONPlaceholder.
    fetchUsers() {
      this.loadingUsers = true;

      axios
        .get('https://jsonplaceholder.typicode.com/users')
        .then((response) => {
          this.users = response.data;
        })
        .catch((error) => {
          console.error('Error fetching users:', error);
        })
        .finally(() => {
          this.loadingUsers = false;
        });
    }
  }
