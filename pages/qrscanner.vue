<template>
  <v-container>
    <!-- QR Scanner Container -->
    <div
      id="qr-reader"
      style="width: 100%; min-height: 400px; border: 1px solid #ccc; border-radius: 8px; margin-top: 1rem;"
    ></div>

    <!-- Display the Scanned QR Code Result -->
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
    // Initialize the QR Code Scanner
    async initScanner() {
      try {
        // Dynamically load the html5-qrcode library
        const Html5Qrcode = await this.$getHtml5Qrcode();

        // Ensure the container is present
        const container = document.getElementById("qr-reader");
        if (!container) {
          throw new Error("QR Reader container not found.");
        }

        // Configure the QR scanner settings
        const config = { fps: 10, qrbox: 250 };

        // Initialize the QR code scanner
        this.html5QrcodeScanner = new Html5QrcodeScanner(
          "qr-reader", // ID of the container div
          config,
          false // disable verbose logging
        );

        // Start rendering the QR code scanner and handle scanned results
        this.html5QrcodeScanner.render(
          (decodedText) => {
            this.scanResult = decodedText;
            this.$emit('scanned', decodedText);
          },
          (errorMessage) => {
            console.debug("Scan failed:", errorMessage);
          }
        );
      } catch (err) {
        console.error("Error initializing scanner:", err);
        this.$emit("error", err);
      }
    }
  }
};
</script>

<style scoped>
#qr-reader {
  /* Ensure the QR scanner container has some visible size */
  min-height: 400px;
  width: 100%;
  border: 1px solid #ccc;
  border-radius: 8px;
  margin-top: 1rem;
}
</style>
