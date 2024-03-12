<template>
  <div>
    <div v-if="user">
      <h1>Welcome</h1>
      <p>Email: {{ user.email }}</p>
      <button @click="logout">Logout</button>

      <p>{{ user.uid }}</p>
    </div>
    <div v-else-if="state == 'login'">
      <h1>Login</h1>
      <form @submit.prevent="login">
        <div class="field">
          <input v-model="email" type="email" placeholder="Email" required />
        </div>
        <div class="field">
          <input
            v-model="password"
            type="password"
            placeholder="Password"
            required
          />
        </div>
        <button type="submit">Login</button>
        <button @click="state = 'register'">Register</button>
      </form>
    </div>
    <div v-else-if="state == 'register'">
      <h1>Register</h1>
      <form @submit.prevent="register">
        <div class="field">
          <input v-model="email" type="email" placeholder="Email" required />
        </div>
        <div class="field">
          <input
            v-model="password"
            type="password"
            placeholder="Password"
            required
          />
        </div>
        <div class="field">
          <input
            v-model="confirmPassword"
            type="password"
            placeholder="Confirm password"
            required
          />
        </div>
        <button type="submit">Register</button>
        <button @click="state = 'login'">Login</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue"
import { initializeApp } from "firebase/app"
import {
  getAuth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  onAuthStateChanged,
  signOut
} from "firebase/auth"

const config = useRuntimeConfig()

const firebaseConfig = {
  apiKey: config.public.firebaseApiKey,
  authDomain: config.public.firebaseAuthDomain,
  projectId: config.public.firebaseProjectId,
  storageBucket: config.public.firebaseStorageBucket,
  messagingSenderId: config.public.firebaseMessagingSenderId,
  appId: config.public.firebaseApiId
}

initializeApp(firebaseConfig)
const email = ref("")
const password = ref("")
const confirmPassword = ref("")
const user = ref(null)
const state = ref("login")
const auth = getAuth()

onMounted(() => {
  // Check for stored login state on component mount
  if (localStorage.getItem("loggedIn")) {
    user.value = auth.currentUser
  }
})

const login = async () => {
  try {
    const credentials = await signInWithEmailAndPassword(
      auth,
      email.value,
      password.value
    )
    // Redirect or do something after successful login
    user.value = credentials.user
    localStorage.setItem("loggedIn", "true")
  } catch (error) {
    console.error("Error logging in:", error)
  }
}

const register = async () => {
  if (password.value !== confirmPassword.value) {
    console.log("Passwords dont match")
  } else {
    try {
      const credentials = await createUserWithEmailAndPassword(
        auth,
        email.value,
        password.value
      )
      // Redirect or do something after successful login
      user.value = credentials.user
      localStorage.setItem("loggedIn", "true")
    } catch (error) {
      console.error("Error logging in:", error)
      // Handle error, e.g., show error message
    }
  }
}

const logout = async () => {
  await signOut(auth)
  user.value = null
  // Clear login state
  localStorage.removeItem("loggedIn")
}

// Listen for authentication state changes
onAuthStateChanged(auth, (currentUser) => {
  user.value = currentUser
})
</script>
