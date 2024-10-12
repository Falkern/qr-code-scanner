<template>
  <div class="qr-scanner">
    <h1>QR Code Scanner</h1>
    <div
      class="drop-zone"
      @dragover.prevent
      @drop.prevent="onDrop"
      @click="$refs.fileInput.click()"
    >
      <input
        type="file"
        accept="image/*"
        @change="onFileChange"
        ref="fileInput"
        style="display: none;"
      />
      <p>Drag & drop an image here, or click to select one.</p>
    </div>
    <canvas ref="canvas" style="display: none;"></canvas>
    <img
      v-if="imgSrc"
      :src="imgSrc"
      alt="Uploaded Image"
      style="max-width: 100%; margin-top: 20px;"
    />
    <p v-if="result">
      QR Code Result:
      <span v-if="isValidURL(result)">
        <a :href="result" target="_blank" rel="noopener noreferrer">{{ result }}</a>
      </span>
      <span v-else>{{ result }}</span>
    </p>
    <footer class="footer">
      <a href="https://github.com/FALKERN" target="_blank" class="glowing-text">
        Made by FALKERN
      </a>
    </footer>
  </div>
</template>

<script>
import jsQR from "jsqr";

export default {
  data() {
    return {
      result: null,
      imgSrc: null,
    };
  },
  methods: {
    onFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.processImage(file);
      }
    },
    onDrop(event) {
      const file = event.dataTransfer.files[0];
      if (file) {
        this.processImage(file);
      }
    },
    processImage(file) {
      const reader = new FileReader();
      
      reader.onload = (e) => {
        this.imgSrc = e.target.result;
        const img = new Image();
        img.src = e.target.result;
        img.onload = () => {
          this.decodeQRCode(img);
        };
      };
      reader.readAsDataURL(file);
    },
    decodeQRCode(img) {
      const canvas = this.$refs.canvas;
      const context = canvas.getContext("2d");
      canvas.width = img.width;
      canvas.height = img.height;
      context.drawImage(img, 0, 0, img.width, img.height);
      const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, canvas.width, canvas.height);

      if (code) {
        this.result = code.data;
      } else {
        this.result = "No QR code found.";
      }
    },
    isValidURL(string) {
      const pattern = new RegExp('^(https?:\\/\\/)?' + // protocol
        '((([a-z0-9]+(-[a-z0-9]+)*\\.)+[a-z]{2,})|' + // domain name
        'localhost|' + // localhost
        '\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}|' + // IP address
        '\\[?[a-f0-9]*:[a-f0-9:]+\\]?)' + // IPv6
        '(\\:\\d+)?(\\/[-a-z0-9+&@#\\/%=~_|$?!:.]*)(\\?[;&a-z0-9+&@#\\/%=~_|$?!:.]*)?' + // path
        '(\\#[-a-z0-9+&@#\\/%=~_|$?!:.]*)?$','i'); // fragment locator
      return !!pattern.test(string);
    }
  },
};
</script>

<style scoped>
.qr-scanner {
  text-align: center;
  margin: 20px;
}

.drop-zone {
  border: 2px dashed #ccc;
  border-radius: 10px;
  padding: 20px;
  margin: 20px 0;
  cursor: pointer;
  transition: background-color 0.3s;
}

.footer {
  text-align: center;
  margin-top: 20px;
}

.glowing-text {
  color: #fff;
  text-decoration: none;
  font-size: 18px;
  font-weight: bold;
  animation: glow 1.5s ease-in-out infinite alternate;
}

@keyframes glow {
  from {
    text-shadow: 0 0 1px #ffffff, 0 0 2px #ffffff, 0 0 3px #ffffff, 0 0 4px #ffffff, 0 0 5px #ffffff;
  }
  to {
    text-shadow: 0 0 2px #d2d2d2, 0 0 4px #d2d2d2, 0 0 6px #d2d2d2, 0 0 8px #d2d2d2, 0 0 10px #d2d2d2;
  }
}
</style>