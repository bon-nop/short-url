<script>
import axios from 'axios';
import QRCode from 'qrcode-generator';

const backendURL = 'https://web-api-20e2.onrender.com';
// const backendURL = 'http://localhost:3000';

export default {
  data() {
    return {
      originalUrl: '',
      shortCode: null,
      qrCodeImageUrl: null,
      clickCount: null,
      msgError: null,
      items: []
    };
  },
  methods: {
    isValidUrl(url) {
      // Regular expression to check URL format
      const urlPattern = /^(http:\/\/www\.|https:\/\/www\.|http:\/\/|https:\/\/)?[a-z0-9]+([\-\.]{1}[a-z0-9]+)*\.[a-z]{2,5}(:[0-9]{1,5})?(\/.*)?$/i;
      return urlPattern.test(url);
    },

    async shortenUrl() {
      if (!this.isValidUrl(this.originalUrl)) {
        // Display an error message or handle invalid URL case
        this.msgError = 'Invalid URL format';
        return;
      }

      try {
        const response = await axios.post(`${backendURL}/shorten`, { originalUrl: this.originalUrl });
        this.shortCode = response.data.shortCode;
        this.clickCount = 0;
        this.msgError = null;

        // Generate QR code on the client side
        const qrCode = QRCode(10, 'M');
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
    async fetchDataFromAPI() {
      try {
        const response = await axios.get(`${backendURL}/getAllUrls`);
        
        this.items = response.data.map(item => {
        return {
          ...item,
          completeUrl: backendURL + '/' + item.shortCode,
        };
      });
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    }
  }, 
  mounted() {
    this.fetchDataFromAPI(); // Call the method when the component is mounted
  }
};
</script>

<template>
  <div class="container">
    <h2 class="p-2 text-center mb-5">Kho San San</h2>
    <div class="g-3 row justify-content-center">
      <div class="col-7">
        <input v-model="originalUrl" type="text" class="form-control" placeholder="Enter link here" required>
        <div v-if="msgError" class="alert alert-danger" role="alert">
          {{ msgError }}
        </div>
      </div>
      <div class="col-auto">
        <button type="submit" @click="shortenUrl" class="btn btn-success btn-lg mb-3 text-white">Shorten URL</button>
      </div>
    </div>
    
    <div v-if="shortCode" class="card text-center my-3 card-gen-url">
      <img :src="qrCodeImageUrl" class="card-img-top mt-5" alt="QR Code">
      <div class="card-body">
        <a :href="getShortUrl()" target="_blank" class="card-text">
          <h5>{{ getShortUrl() }}</h5>
        </a>
      </div>
    </div>

    <hr />
    <h4 class="text-center bg-light p-2 mt-5">List URL</h4>
    <div class="g-3 row justify-content-center">
      <div v-for="item in items" :key="item.id" class="col-5">
        <div class="card">
          <div class="card-body">
            <a :href="item.completeUrl" target="_blank" class="card-title">
              <h5>{{ item.completeUrl }}</h5>
            </a>
            <h6 class="card-subtitle mb-2 text-body-secondary">{{ item.originalUrl }}</h6>
            <p class="card-text text-body-secondary">{{ `Visit count : ${item.clickCount}` }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>