<template>
  <IonPage>
    <IonHeader>
      <IonToolbar color="primary">
        <IonTitle>Photo Gallery</IonTitle>
      </IonToolbar>
    </IonHeader>

    <IonContent class="ion-padding">

      <!-- Take Photo Button -->
      <IonFab vertical="bottom" horizontal="center" slot="fixed">
        <IonFabButton @click="takePhoto">
          <IonIcon :icon="camera" />
        </IonFabButton>
      </IonFab>

      <!-- Photo Grid -->
      <IonGrid>
        <IonRow>
          <IonCol
            v-for="(photo, index) in photos"
            :key="index"
            size="6"
          >
            <div class="photo-card" @click="showActionSheet(photo, index)">
              <img :src="photo.webviewPath" alt="Captured photo" />
            </div>
          </IonCol>
        </IonRow>
      </IonGrid>

      <!-- Empty State -->
      <div v-if="photos.length === 0" class="empty-state">
        <IonIcon :icon="imagesOutline" class="empty-icon" />
        <h2>No Photos Yet</h2>
        <p>Tap the camera button below to take your first photo</p>
      </div>

    </IonContent>
  </IonPage>
</template>


<script setup lang="ts">
import { ref, onMounted } from 'vue'
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonGrid,
  IonRow,
  IonCol,
  IonFab,
  IonFabButton,
  IonIcon,
  actionSheetController
} from '@ionic/vue'
import { camera, imagesOutline, trash, close } from 'ionicons/icons'
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera'
import { Preferences } from '@capacitor/preferences'

interface UserPhoto {
  filepath: string
  webviewPath: string
}

const PHOTO_STORAGE_KEY = 'photos'
const photos = ref<UserPhoto[]>([])

async function takePhoto() {
  try {
    const image = await Camera.getPhoto({
      resultType: CameraResultType.Uri,
      source: CameraSource.Camera,
      quality: 90
    })

    const fileName = `photo_${Date.now()}.${image.format}`
    const newPhoto: UserPhoto = {
      filepath: fileName,
      webviewPath: image.webPath!
    }

    photos.value = [newPhoto, ...photos.value]
    await savePhotos()
  } catch {
    // User cancelled the camera — do nothing
  }
}

async function deletePhoto(index: number) {
  photos.value.splice(index, 1)
  await savePhotos()
}

async function showActionSheet(photo: UserPhoto, index: number) {
  const actionSheet = await actionSheetController.create({
    header: 'Photo Options',
    buttons: [
      {
        text: 'Delete',
        role: 'destructive',
        icon: trash,
        handler: () => deletePhoto(index)
      },
      {
        text: 'Cancel',
        icon: close,
        role: 'cancel'
      }
    ]
  })
  await actionSheet.present()
}

async function savePhotos() {
  await Preferences.set({
    key: PHOTO_STORAGE_KEY,
    value: JSON.stringify(photos.value)
  })
}

async function loadSavedPhotos() {
  const { value } = await Preferences.get({ key: PHOTO_STORAGE_KEY })
  if (value) {
    photos.value = JSON.parse(value) as UserPhoto[]
  }
}

onMounted(() => {
  loadSavedPhotos()
})
</script>


<style scoped>
ion-content {
  --background: #f4f5f8;
}

.photo-card {
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
  cursor: pointer;
  transition: transform 0.2s;
}

.photo-card:active {
  transform: scale(0.96);
}

.photo-card img {
  width: 100%;
  height: 180px;
  object-fit: cover;
  display: block;
}

.empty-state {
  text-align: center;
  padding: 60px 24px;
  color: #92949c;
}

.empty-icon {
  font-size: 72px;
  margin-bottom: 16px;
  color: #c5cad3;
}

.empty-state h2 {
  margin: 0 0 8px;
  font-size: 1.3rem;
  font-weight: 600;
  color: #555;
}

.empty-state p {
  margin: 0;
  font-size: 0.95rem;
}

ion-fab-button {
  --background: #3880ff;
  --background-activated: #2667d8;
}

ion-col {
  padding: 6px;
}
</style>
