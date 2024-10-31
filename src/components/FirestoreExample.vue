<template>
  <div class="login-container">
    
    <form @submit.prevent="login">
      <div class="field">
        <label class="label" for="email">Email: </label>
        <div class="control">
          <input class="input" type="email" v-model="email" placeholder="Enter your email" required />
        </div>
      </div>
      <div class="field">
        <label class="label" for="password">Password:</label>
        <div class="control">
          <input class="input" type="password" v-model="password" placeholder="Enter your password" required />
        </div>
      </div>
      <div class="field">
        <div class="control">
          <button class="button is-primary" type="submit">Login</button>
        </div>
      </div>
    </form>
    <p v-if="user" class="notification is-info">Logged in as: {{ user.email }}</p>
  </div>
</template>


<script>
import { ref } from 'vue';
import { signInWithEmailAndPassword, onAuthStateChanged } from 'firebase/auth';
import { getDoc, doc } from 'firebase/firestore';
import { auth, db } from '../firebase.js';

export default {
  name: 'AuthExample',
  setup() {
    const email = ref('');
    const password = ref('');
    const user = ref(null);

    const login = async () => {
      try {
        // Référence au document utilisateur dans Firestore
        const userDocRef = doc(db, 'users', email.value);
        const userDoc = await getDoc(userDocRef);

        if (userDoc.exists()) {
          console.log('Utilisateur trouvé dans Firestore');
          // Si l'utilisateur existe dans Firestore, essaye de le connecter
          await signInWithEmailAndPassword(auth, email.value, password.value);
          console.log('Utilisateur connecté avec succès');
        } else {
          console.error('Utilisateur non trouvé dans la base de données.');
        }
      } catch (error) {
        console.error('Erreur lors de la connexion:', error);
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



<style scoped>
.login-container {
  max-width: 400px;
  margin: auto;
  padding: 20px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
</style>


<style scoped>
/* AuthExample.css */
body {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f0f0f0;
  margin: 0;
}

.login-container {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.login-container h2 {
  margin-bottom: 20px;
}

.login-container input[type="email"],
.login-container input[type="password"] {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.login-container input[type="checkbox"] {
  margin-right: 10px;
}

.login-container .remember-me {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.login-container .forgot-password {
  color: #007bff;
  text-decoration: none;
}

.login-container .forgot-password:hover {
  text-decoration: underline;
}

.login-container button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.login-container button:hover {
  background-color: #0056b3;
}


</style>
