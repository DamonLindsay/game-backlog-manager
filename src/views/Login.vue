<template>
  <ion-page>
    <ion-content class="ion-padding">
      <div class="login-container">
        <h1>{{ isSignUp ? 'Sign Up' : 'Log In' }}</h1>

        <ion-item>
          <ion-label position="stacked">Email</ion-label>
          <ion-input v-model="email" type="email" required></ion-input>
        </ion-item>

        <ion-item>
          <ion-label position="stacked">Password</ion-label>
          <ion-input v-model="password" type="password" required></ion-input>
        </ion-item>

        <ion-text color="danger" v-if="errorMessage">
          <p>{{ errorMessage }}</p>
        </ion-text>

        <ion-button expand="block" @click="handleSubmit" class="ion-margin-top">
          {{ isSignUp ? 'Sign Up' : 'Log In' }}
        </ion-button>

        <ion-button expand="block" fill="clear" @click="isSignUp = !isSignUp">
          {{ isSignUp ? 'Already have an account? Log In' : "Don't have an account? Sign Up" }}
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import {
  IonPage,
  IonContent,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
  IonText,
} from '@ionic/vue'
import {
  getAuth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
} from 'firebase/auth'
import firebaseApp from '../firebase'

const auth = getAuth(firebaseApp)

const router = useRouter()

const email = ref('')
const password = ref('')
const isSignUp = ref(false)
const errorMessage = ref('')

async function handleSubmit() {
  errorMessage.value = ''
  try {
    if (isSignUp.value) {
      await createUserWithEmailAndPassword(auth, email.value, password.value)
    } else {
      await signInWithEmailAndPassword(auth, email.value, password.value)
    }
    router.push('/tabs/tab1')
  } catch (error: any) {
    errorMessage.value = error.message
  }
}
</script>

<style scoped>
.login-container {
  max-width: 400px;
  margin: 0 auto;
  padding-top: 80px;
}
</style>