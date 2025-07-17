<template>
  <v-container>
    <!-- Scanner Interface -->
    <v-btn
      v-if="!isScanning && !loading"
      color="primary"
      @click="startScanner"
      :disabled="cameraUnavailable"
      block
      large
    >
      <v-icon left>mdi-camera</v-icon>
      {{ cameraUnavailable ? 'Camera Unavailable' : 'Start Scanner' }}
    </v-btn>

    <!-- Error Display -->
    <v-alert
      v-if="errorMessage"
      type="error"
      class="mt-4"
      outlined
    >
      {{ errorMessage }}
      <v-btn
        v-if="showRetry"
        small
        color="primary"
        @click="startScanner"
        class="ml-2"
      >
        Retry
      </v-btn>
    </v-alert>

    <!-- Scanner View -->
    <div
      id="scanner"
      v-show="isScanning"
      class="scanner-container"
    ></div>

    <!-- Debug Information (remove in production) -->
    <v-expansion-panels v-if="debugInfo" class="mt-4">
      <v-expansion-panel>
        <v-expansion-panel-header>Debug Information</v-expansion-panel-header>
        <v-expansion-panel-content>
          <pre>{{ debugInfo }}</pre>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>
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
    debug: {
      type: Boolean,
      default: false,
    }
  },
  data() {
    return {
      html5QrCode: null,
      isScanning: false,
      loading: false,
      errorMessage: '',
      cameraUnavailable: false,
      showRetry: false,
      debugInfo: null
    };
  },
  beforeDestroy() {
    this.cleanupScanner();
  },
  methods: {
    async startScanner() {
      this.loading = true;
      this.errorMessage = '';
      this.showRetry = false;
      this.debugInfo = this.debug ? {} : null;

      try {
        // Step 1: Verify browser support
        this.verifyBrowserSupport();
        
        // Step 2: Test basic camera access
        await this.testCameraAccess();
        
        // Step 3: Initialize scanner
        await this.initializeScanner();
        
        this.isScanning = true;
        this.$emit('started');
      } catch (err) {
        this.handleScannerError(err);
      } finally {
        this.loading = false;
      }
    },

    verifyBrowserSupport() {
      if (this.debug) this.debugInfo.browserSupport = {
        mediaDevices: !!navigator.mediaDevices,
        getUserMedia: !!navigator.mediaDevices?.getUserMedia,
        enumerateDevices: !!navigator.mediaDevices?.enumerateDevices
      };

      if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        throw new Error('BrowserDoesNotSupport', 'Your browser doesn\'t support camera access');
      }
    },

    async testCameraAccess() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        if (this.debug) this.debugInfo.cameraTest = { stream: !!stream };
        
        // Get available devices
        const devices = await navigator.mediaDevices.enumerateDevices();
        const cameras = devices.filter(d => d.kind === 'videoinput');
        
        if (cameras.length === 0) {
          throw new Error('NoCamerasFound', 'No cameras detected on this device');
        }
        
        // Release the stream
        stream.getTracks().forEach(track => track.stop());
      } catch (err) {
        if (this.debug) this.debugInfo.cameraTest = { error: err.message };
        throw err;
      }
    },

    async initializeScanner() {
      try {
        // Load the scanner library
        const Html5Qrcode = await this.$getHtml5Qrcode();
        this.html5Qrcode = new Html5Qrcode("scanner");

        if (this.debug) this.debugInfo.scannerInit = {
          libraryLoaded: !!Html5Qrcode,
          scannerInstance: !!this.html5Qrcode
        };

        // Start scanning
        const config = {
          fps: 10,
          qrbox: this.qrbox,
          aspectRatio: 1.0,
          disableFlip: false
        };

        const startResult = await this.html5Qrcode.start(
          { facingMode: "environment" },
          config,
          this.onScanSuccess,
          this.onScanError
        );

        if (this.debug) this.debugInfo.scannerStart = startResult;
      } catch (err) {
        if (this.debug) this.debugInfo.scannerError = err;
        throw err;
      }
    },

    onScanSuccess(decodedText) {
      this.$emit("scanned", decodedText);
      if (this.debug) this.debugInfo.lastScan = decodedText;
    },

    onScanError(errorMessage) {
      console.warn("QR scan error:", errorMessage);
      if (this.debug) this.debugInfo.scanErrors = this.debugInfo.scanErrors || [];
      this.debugInfo.scanErrors.push(errorMessage);
    },

    handleScannerError(err) {
      console.error("Scanner error:", err);
      
      let userMessage = 'Scanner error: Unknown error occurred';
      this.showRetry = true;
      this.cameraUnavailable = true;

      if (err.name === 'NotAllowedError') {
        userMessage = 'Camera access denied. Please allow camera permissions in your browser settings.';
      } else if (err.name === 'NotFoundError' || err.name === 'NoCamerasFound') {
        userMessage = 'No camera found on this device.';
      } else if (err.name === 'BrowserDoesNotSupport') {
        userMessage = 'Your browser doesn\'t support camera access. Try Chrome or Firefox.';
      } else if (err.message.includes('html5-qrcode not supported')) {
        userMessage = 'QR scanning not supported in this browser.';
      } else {
        userMessage = `Scanner error: ${err.message || 'Unknown error'}`;
      }

      this.errorMessage = userMessage;
      this.$emit('error', { 
        error: err, 
        userMessage,
        debugInfo: this.debugInfo
      });
    },

    cleanupScanner() {
      if (this.html5Qrcode && this.isScanning) {
        this.html5Qrcode.stop().catch(err => {
          console.warn("Error stopping scanner:", err);
        });
      }
      this.html5Qrcode = null;
      this.isScanning = false;
    }
  }
};
</script>

<style scoped>
.scanner-container {
  width: 100%;
  height: 400px;
  border: 2px solid #000;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  background-color: #f5f5f5;
  margin: 16px 0;
}

.scanner-container::before {
  content: "Camera Preview";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #999;
  font-size: 1.2rem;
  z-index: 0;
}

.scanner-container video {
  position: relative;
  z-index: 1;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>