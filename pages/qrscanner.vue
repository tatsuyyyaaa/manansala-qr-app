<template>
  <v-container>
    <!-- Button to start scanning -->
    <v-btn
      v-if="!isScanning && !loading"
      color="primary"
      @click="startScanner"
    >
      Start Scanner
    </v-btn>

    <!-- Loading indicator while scanner is initializing -->
    <v-progress-circular
      v-if="loading"
      indeterminate
      color="primary"
      class="ma-4"
    />

    <!-- Scanner view once active -->
    <div
      v-show="isScanning"
      id="scanner"
      style="width: 100%; height: 400px; border: 2px solid #000"
    ></div>
  </v-container>
</template>

<script>
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
      loading: false,
    };
  },
  beforeDestroy() {
    this.stopScanner();
  },
  methods: {
    async startScanner() {
      this.loading = true;

      try {
        const { Html5Qrcode } = await import("html5-qrcode");
        this.html5QrCode = new Html5Qrcode("scanner");

        const config = {
          fps: 10,
          qrbox: this.qrbox,
        };

        await this.html5QrCode.start(
          { facingMode: "environment" },
          config,
          (decodedText) => {
            console.log("Scanned result:", decodedText);
            this.$emit("scanned", decodedText);
          },
          (errorMessage) => {
            // Optionally log scan errors
            console.log("QR scan error:", errorMessage);
          }
        );

        this.isScanning = true;
      } catch (err) {
        console.error("Failed to start QR scanner:", err);
      } finally {
        this.loading = false;
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
