<template>
  <div>
    <div v-if="!isScanning">
      <v-btn color="primary" @click="startScanner">
        Start Scanner
      </v-btn>
    </div>

    <div
      v-show="isScanning"
      id="scanner"
      style="width: 100%; height: 400px;"
    ></div>
  </div>
</template>

<script>
import { Html5Qrcode } from "html5-qrcode";

export default {
  name: "QrScanner",
  props: {
    qrbox: {
      type: Number,
      default: 250,
    },
  },
  data() {
    return {
      html5QrCode: null,
      isScanning: false,
    };
  },
  beforeDestroy() {
    this.stopScanner();
  },
  methods: {
    async startScanner() {
      this.isScanning = true;
      const config = {
        fps: 10,
        qrbox: this.qrbox,
      };

      try {
        this.html5QrCode = new Html5Qrcode("scanner");
        await this.html5QrCode.start(
          { facingMode: "environment" },
          config,
          (decodedText) => {
            console.log("Scanned result:", decodedText);
            this.$emit("scanned", decodedText);
          },
          (errorMessage) => {
            // You can log error or ignore
          }
        );
      } catch (err) {
        console.error("Failed to start QR scanner:", err);
      }
    },

    stopScanner() {
      if (this.html5QrCode && this.isScanning) {
        this.html5QrCode
          .stop()
          .then(() => {
            this.html5QrCode.clear();
            this.isScanning = false;
          })
          .catch((err) => {
            console.error("Error stopping QR scanner:", err);
          });
      }
    },
  },
};
</script>

<style scoped>
#scanner {
  border: 2px solid #000;
}
</style>
