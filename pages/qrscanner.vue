<template>
  <v-container>
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
      id="qr-reader"
      v-show="isScanning"
      style="width: 100%; min-height: 300px;"
    ></div>

    <!-- Status Messages -->
    <v-alert
      v-if="statusMessage"
      :type="alertType"
      class="mt-4"
    >
      {{ statusMessage }}
    </v-alert>
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
      statusMessage: '',
      alertType: 'info'
    };
  },
  beforeDestroy() {
    this.stopScanner();
  },
  methods: {
    async startScanner() {
      this.loading = true;
      this.statusMessage = 'Initializing scanner...';
      this.alertType = 'info';

      try {
        // Load the scanner library
        const { Html5Qrcode } = await import('html5-qrcode');
        
        // Initialize scanner - IMPORTANT: Use 'qr-reader' as the container ID
        this.html5QrCode = new Html5Qrcode("qr-reader");
        
        // Configuration
        const config = {
          fps: 10,
          qrbox: 250,
          aspectRatio: 1.0,
          disableFlip: false
        };

        // Start scanning
        this.statusMessage = 'Accessing camera...';
        await this.html5QrCode.start(
          { facingMode: "environment" },
          config,
          (decodedText) => {
            this.handleScanSuccess(decodedText);
          },
          (errorMessage) => {
            console.warn('Scan error:', errorMessage);
          }
        );

        this.isScanning = true;
        this.statusMessage = 'Scanner ready - point camera at QR code';
        this.alertType = 'success';
      } catch (err) {
        this.handleError(err);
      } finally {
        this.loading = false;
      }
    },

    handleScanSuccess(decodedText) {
      console.log('Scan result:', decodedText);
      this.statusMessage = `Scanned: ${decodedText}`;
      this.alertType = 'success';
      this.$emit('scanned', decodedText);
      
      // Optional: Stop after successful scan
      // this.stopScanner();
    },

    handleError(err) {
      console.error('Scanner error:', err);
      this.alertType = 'error';
      
      if (err.name === 'NotAllowedError') {
        this.statusMessage = 'Camera access denied. Please allow camera permissions in your browser settings.';
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
        } catch (err) {
          console.warn('Error stopping scanner:', err);
        }
      }
      this.isScanning = false;
      this.html5QrCode = null;
      this.statusMessage = 'Scanner stopped';
      this.alertType = 'info';
    }
  }
};
</script>