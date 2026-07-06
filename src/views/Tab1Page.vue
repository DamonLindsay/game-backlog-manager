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

        <h2 class="ion-margin-top">Your Games</h2>
        <ion-list>
          <ion-item v-for="game in games" :key="game.id">
            <ion-label>
              <h3>{{ game.title }}</h3>
              <p>{{ game.platform }} — {{ game.status }}</p>
            </ion-label>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
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
  IonList,
} from '@ionic/vue'
import { collection, addDoc, Timestamp, query, where } from 'firebase/firestore'
import { getAuth } from 'firebase/auth'
import { db } from '@/firebase'
import firebaseApp from '@/firebase'
import { useCollection } from 'vuefire'

const title = ref('')
const platform = ref('')
const status = ref<'backlog' | 'playing' | 'completed'>('backlog')
const notes = ref('')
const errorMessage = ref('')

const auth = getAuth(firebaseApp)

const gamesQuery = computed(() => {
  const user = auth.currentUser
  if (!user) return null
  return query(collection(db, 'games'), where('userId', '==', user.uid))
})

const games = useCollection(gamesQuery)

async function handleAddGame() {
  errorMessage.value = ''

  if (!title.value.trim()) {
    errorMessage.value = 'Title is required.'
    return
  }

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