<template>
  <div class="container vh-100 d-flex justify-content-center align-items-center">
    <div class="card p-4 shadow-sm w-100" style="max-width: 400px;">
      <h2 class="text-center mb-3">Login</h2>
      <form @submit.prevent="handleLogin">
        <div class="mb-3">
          <input v-model="email" type="email" class="form-control" placeholder="Email" required />
        </div>
        <div class="mb-3">
          <input v-model="password" type="password" class="form-control" placeholder="Password" required />
        </div>
        <button type="submit" class="btn btn-primary w-100">Login</button>
        <p v-if="error" class="text-danger mt-2 text-center">{{ error }}</p>
        <p class="mt-3 text-center">
          Don't have an account? 
          <router-link to="/register">Register</router-link>
        </p>
      </form>
    </div>
  </div>
</template>

<script>
import api from '../axios';

export default {
  name: 'Login',
  data() {
    return {
      email: '',
      password: '',
      error: ''
    };
  },
  methods: {
    async handleLogin() {
      this.error = '';
      try {
        const res = await api.post('/api/auth/login', {
          email: this.email,
          password: this.password
        });
        localStorage.setItem('token', res.data.token);
        this.$router.push('/stations');
      } catch (err) {
        this.error = err.response?.data?.msg || 'Login failed.';
      }
    }
  }
};
</script>
