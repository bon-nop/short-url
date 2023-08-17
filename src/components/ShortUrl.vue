<script>
import axios from 'axios';
import QRCode from 'qrcode-generator';

const backendURL = 'https://bon-nop.github.io/short-url';

export default {
  data() {
    return {
      originalUrl: '',
      shortCode: null,
      qrCodeImageUrl: null,
      clickCount: null,
    };
  },
  methods: {
    async shortenUrl() {
      try {
        const response = await axios.post(`${backendURL}/shorten`, { originalUrl: this.originalUrl });
        this.shortCode = response.data.shortCode;
        this.clickCount = 0;

        // Generate QR code on the client side
        const qrCode = QRCode(0, 'H');
        qrCode.addData(this.getShortUrl());
        qrCode.make();

        // Set the QR code image URL
        this.qrCodeImageUrl = qrCode.createDataURL();
      } catch (error) {
        console.error('Error shortening URL:', error);
      }
    },
    getShortUrl() {
      return backendURL + '/' + this.shortCode;
    },
  },
};
</script>

<template>
  <div>
    <input v-model="originalUrl" class="p-3" placeholder="Enter link here">
    <button @click="shortenUrl">Shorten</button>
  </div>

  <div v-if="shortCode">
    <p>Short URL: <a :href="getShortUrl()" target="_blank">{{ getShortUrl() }}</a></p>
    <img :src="qrCodeImageUrl" alt="QR Code">
  </div>
</template>