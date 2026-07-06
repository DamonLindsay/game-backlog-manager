<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Game Library</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Game Library</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="ion-padding">
        <h2>Add a Game</h2>

        <ion-item>
          <ion-label position="stacked">Title</ion-label>
          <ion-input v-model="title" required></ion-input>
        </ion-item>

        <ion-item>
          <ion-label position="stacked">Platform</ion-label>
          <ion-input v-model="platform"></ion-input>
        </ion-item>

        <ion-item>
          <ion-label position="stacked">Status</ion-label>
          <ion-select v-model="status" interface="popover">
            <ion-select-option value="backlog">Backlog</ion-select-option>
            <ion-select-option value="playing">Playing</ion-select-option>
            <ion-select-option value="completed">Completed</ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item>
          <ion-label position="stacked">Notes</ion-label>
          <ion-textarea v-model="notes"></ion-textarea>
        </ion-item>

        <ion-text color="danger" v-if="errorMessage">
          <p>{{ errorMessage }}</p>
        </ion-text>

        <ion-button expand="block" class="ion-margin-top" @click="handleAddGame">
          Add Game
        </ion-button>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonItem,
  IonLabel,
  IonInput,
  IonSelect,
  IonSelectOption,
  IonTextarea,
  IonButton,
  IonText,
} from '@ionic/vue'
import { collection, addDoc, Timestamp } from 'firebase/firestore'
import { getAuth } from 'firebase/auth'
import { db } from '@/firebase'
import firebaseApp from '@/firebase'

const title = ref('')
const platform = ref('')
const status = ref<'backlog' | 'playing' | 'completed'>('backlog')
const notes = ref('')
const errorMessage = ref('')

async function handleAddGame() {
  errorMessage.value = ''

  if (!title.value.trim()) {
    errorMessage.value = 'Title is required.'
    return
  }

  const auth = getAuth(firebaseApp)
  const user = auth.currentUser

  if (!user) {
    errorMessage.value = 'You must be logged in to add a game.'
    return
  }

  try {
    await addDoc(collection(db, 'games'), {
      title: title.value,
      platform: platform.value,
      status: status.value,
      notes: notes.value,
      dateAdded: Timestamp.now(),
      userId: user.uid,
    })

    title.value = ''
    platform.value = ''
    status.value = 'backlog'
    notes.value = ''
  } catch (error: any) {
    errorMessage.value = error.message
  }
}
</script>