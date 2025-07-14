<template>
  <div>
    <div id="scanner" style="width: 100%; height: 400px;"></div>
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
  mounted() {
    this.startScanner();
  },
  beforeDestroy() {
    this.stopScanner();
  },
  methods: {
    startScanner() {
      const config = {
        fps: 10,
        qrbox: this.qrbox,
      };

      this.html5QrCode = new Html5Qrcode("scanner");

      this.html5QrCode
        .start(
          { facingMode: "environment" },
          config,
          (decodedText) => {
            console.log("Scanned result:", decodedText);
            this.$emit("scanned", decodedText);

            // Optional: stop after successful scan
            // this.stopScanner();
          },
          (errorMessage) => {
            // Comment this out to reduce spam
            // console.warn("QR scan error:", errorMessage);
          }
        )
        .then(() => {
          this.isScanning = true;
        })
        .catch((err) => {
          console.error("Failed to start QR scanner:", err);
        });
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
