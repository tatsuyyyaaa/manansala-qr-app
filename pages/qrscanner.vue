<template>
  <v-container>
    <div id="qr-reader" style="width: 100%;"></div>
    <v-alert
      v-if="scanResult"
      type="success"
      class="mt-4"
      dense
    >
      Scanned: {{ scanResult }}
    </v-alert>
  </v-container>
</template>

<script>
export default {
  name: "QrScanner",
  data() {
    return {
      html5QrcodeScanner: null,
      scanResult: null
    };
  },
  mounted() {
    this.initScanner();
  },
  beforeDestroy() {
    if (this.html5QrcodeScanner) {
      this.html5QrcodeScanner.clear();
    }
  },
  methods: {
    async initScanner() {
      try {
        // Dynamically load Html5Qrcode using your globally available function
        const Html5Qrcode = await this.$getHtml5Qrcode();
        
        // Configure the scanner
        const config = { fps: 10, qrbox: 250 };
        this.html5QrcodeScanner = new Html5QrcodeScanner(
          "qr-reader",
          config,
          /* verbose= */ false
        );

        // Render the scanner
        this.html5QrcodeScanner.render(
          (decodedText) => {
            this.scanResult = decodedText;
            this.$emit('scanned', decodedText);
            // Optionally stop after the first scan:
            // this.html5QrcodeScanner.clear();
          },
          (errorMessage) => {
            console.debug("Scan failed:", errorMessage);
          }
        );
      } catch (err) {
        console.error('Error initializing scanner:', err);
        this.$emit('error', err);
      }
    }
  }
};
</script>

<style scoped>
#qr-reader {
  margin: 1rem auto;
  max-width: 500px;
}
</style>
