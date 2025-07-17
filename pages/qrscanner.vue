<template>
  <v-container class="scanner-container">
    <!-- Scanner Controls -->
    <v-btn
      v-if="!isScanning"
      color="primary"
      @click="startScanner"
      :loading="loading"
      block
      large
    >
      <v-icon left>mdi-camera</v-icon>
      Start Scanner
    </v-btn>

    <!-- Scanner View -->
    <div
      id="qr-scanner-container"
      ref="scannerContainer"
      v-show="isScanning"
      class="scanner-view"
    ></div>

    <!-- Status Messages -->
    <v-alert
      v-if="statusMessage"
      :type="statusType"
      class="mt-4"
    >
      {{ statusMessage }}
    </v-alert>

    <!-- Scan Results -->
    <v-card
      v-if="scanResult"
      class="mt-4"
    >
      <v-card-title>Scan Result</v-card-title>
      <v-card-text>
        <pre>{{ scanResult }}</pre>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "QrScanner",
  data() {
    return {
      html5QrCode: null,
      isScanning: false,
      loading: false,
      scanResult: null,
      statusMessage: '',
      statusType: 'info'
    };
  },
  beforeDestroy() {
    this.stopScanner();
  },
  methods: {
    async startScanner() {
      this.loading = true;
      this.statusMessage = 'Initializing scanner...';
      this.statusType = 'info';
      this.scanResult = null;

      try {
        // 1. Load the scanner library
        const Html5Qrcode = await this.loadLibrary();
        
        // 2. Create scanner instance
        this.html5QrCode = new Html5Qrcode("qr-scanner-container");
        
        // 3. Configure scanner
        const config = {
          fps: 10,
          qrbox: 250,
          aspectRatio: 1.0,
          disableFlip: false
        };

        // 4. Start scanning
        this.statusMessage = 'Accessing camera...';
        await this.html5QrCode.start(
          { facingMode: "environment" },
          config,
          this.onScanSuccess,
          this.onScanError
        );

        this.isScanning = true;
        this.statusMessage = 'Scanner ready - point at a QR code';
      } catch (err) {
        this.handleError(err);
      } finally {
        this.loading = false;
      }
    },

    async loadLibrary() {
      try {
        // Try plugin method first
        if (this.$getHtml5Qrcode) {
          const lib = await this.$getHtml5Qrcode();
          if (lib) return lib;
        }
        
        // Fallback to direct import
        const { Html5Qrcode } = await import('html5-qrcode');
        return Html5Qrcode;
      } catch (err) {
        throw new Error(`Failed to load scanner library: ${err.message}`);
      }
    },

    onScanSuccess(decodedText) {
      this.scanResult = decodedText;
      this.statusMessage = 'QR code detected!';
      this.statusType = 'success';
      this.$emit('scanned', decodedText);
    },

    onScanError(errorMessage) {
      // Continue scanning despite errors
      console.warn('Scan error:', errorMessage);
    },

    handleError(err) {
      console.error('Scanner error:', err);
      this.statusType = 'error';
      
      if (err.name === 'NotAllowedError') {
        this.statusMessage = 'Camera access denied. Please allow camera permissions.';
      } else if (err.name === 'NotFoundError') {
        this.statusMessage = 'No camera found on this device.';
      } else {
        this.statusMessage = `Scanner error: ${err.message || 'Unknown error'}`;
      }
    },

    async stopScanner() {
      if (this.html5QrCode && this.isScanning) {
        try {
          await this.html5QrCode.stop();
          this.html5QrCode.clear();
        } catch (err) {
          console.warn('Error stopping scanner:', err);
        }
      }
      this.isScanning = false;
      this.html5QrCode = null;
      this.statusMessage = 'Scanner stopped';
      this.statusType = 'info';
    }
  }
};
</script>

<style scoped>
.scanner-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 16px;
}

.scanner-view {
  width: 100%;
  height: 400px;
  border: 2px solid #1976d2;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  background-color: #000; /* Black background for better contrast */
}

.scanner-view video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>