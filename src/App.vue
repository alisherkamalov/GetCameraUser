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

        mediaRecorder.onstop = async () => {
          const blob = new Blob(chunks, { type: 'video/webm' });
          const arrayBuffer = await blob.arrayBuffer();
          const base64 = Buffer.from(arrayBuffer).toString('base64');

          axios.post('https://f70e-92-46-217-143.ngrok-free.app/api/createphoto/', {
            image: base64
          }, {
            headers: {
              'Content-Type': 'application/json',
            },
          })
          .then(response => {
            console.log('Photo uploaded successfully:', response.data);
            this.photo = `data:image/png;base64,${base64}`; // Отображаем фото на странице
          })
          .catch(error => {
            console.error('Error uploading photo:', error);
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



