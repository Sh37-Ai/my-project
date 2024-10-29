<template>
  <div class="login-container">
    <h2 class="title is-4">Nice to see you again</h2>
    <form @submit.prevent="login">
      <div class="field">
        <label class="label" for="email">Email:</label>
        <div class="control">
          <input class="input" type="email" v-model="email" placeholder="Email or phone number" required />
        </div>
      </div>
      <div class="field">
        <label class="label" for="password">Password:</label>
        <div class="control">
          <input class="input" type="password" v-model="password" placeholder="Enter password" required />
        </div>
      </div>
      <div class="field">
        <div class="control">
          <button class="button is-primary" type="submit">Sign in</button>
        </div>
      </div>
    </form>
    <p v-if="user">Logged in as: {{ user.email }}</p>
  </div>
</template>

<script>
import { ref } from 'vue';
import { signInWithEmailAndPassword, onAuthStateChanged } from 'firebase/auth';
import { auth } from '../firebase.js';

export default {
  setup() {
    const email = ref('');
    const password = ref('');
    const user = ref(null);

    const login = async () => {
      try {
        await signInWithEmailAndPassword(auth, email.value, password.value);
      } catch (error) {
        console.error('Error logging in:', error);
      }
    };

    onAuthStateChanged(auth, (currentUser) => {
      user.value = currentUser;
    });

    return {
      email,
      password,
      user,
      login,
    };
  },
};
</script>

<style>
 
</style>







