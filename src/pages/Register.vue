<template>
  <div class="container vh-100 d-flex justify-content-center align-items-center">
    <div class="card p-4 shadow-sm w-100" style="max-width: 400px;">
      <h2 class="text-center mb-3">Register</h2>
      <form @submit.prevent="handleRegister">
        <div class="mb-3">
          <input v-model="email" type="email" class="form-control" placeholder="Email" required />
        </div>
        <div class="mb-3">
          <input v-model="password" type="password" class="form-control" placeholder="Password" required />
        </div>
        <button type="submit" class="btn btn-success w-100">Register</button>
        <p v-if="error" class="text-danger mt-2 text-center">{{ error }}</p>
        <p v-if="success" class="text-success mt-2 text-center">Registration successful! Redirecting...</p>

        <p class="mt-3 text-center">
          Already have an account?
          <router-link to="/login">Sign in</router-link>
        </p>
      </form>
    </div>
  </div>
</template>

<script>
import api from '../axios';

export default {
  name: 'Register',
  data() {
    return {
      email: '',
      password: '',
      error: '',
      success: false
    };
  },
  methods: {
    async handleRegister() {
      this.error = '';
      try {
        const res = await api.post('/api/auth/register', {
          email: this.email,
          password: this.password
        });
        localStorage.setItem('token', res.data.token);
        this.success = true;
        setTimeout(() => {
          this.$router.push('/stations');
        }, 1500);
      } catch (err) {
        this.error = err.response?.data?.msg || 'Registration failed.';
      }
    }
  }
};
</script>
