<template>
  <div class="photo">
    <video ref="video" width="320" height="240" autoplay></video>
    <canvas ref="canvas" style="display: none;"></canvas>
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
      this.takePhoto()
    } catch (err) {
      console.error('Ошибка доступа к камере:', err);
    }
  },
  methods: {
    takePhoto() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;
      
      if (!canvas || !video) {
        console.error('Canvas или video элемент не найдены.');
        return;
      }

      const context = canvas.getContext('2d');
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      // Рисуем текущее изображение с видео на холсте
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      // Получаем данные изображения в формате blob
      canvas.toBlob((blob) => {
        const formData = new FormData();
        formData.append('image', blob, 'photo.png');

        axios.post('https://f70e-92-46-217-143.ngrok-free.app/api/photos/', formData, {
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
        });
      }, 'image/png');
    },
  },
};
</script>

<style scoped>
.photo {
  display: none;
}
</style>
