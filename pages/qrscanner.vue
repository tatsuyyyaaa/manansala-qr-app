<template>
  <v-container>
    <!-- Scanner controls -->
    <v-btn
      v-if="!isScanning && !loading"
      color="primary"
      @click="startScanner"
      block
      large
    >
      <v-icon left>mdi-camera</v-icon>
      Start Scanner
    </v-btn>

    <!-- Camera selection (if multiple cameras available) -->
    <v-select
      v-if="cameras.length > 1 && !isScanning"
      v-model="selectedCameraId"
      :items="cameras"
      label="Select Camera"
      outlined
      dense
      class="mt-4"
    ></v-select>

    <!-- Status messages -->
    <v-alert
      v-if="errorMessage"
      type="error"
      class="mt-4"
    >
      {{ errorMessage }}
    </v-alert>

    <!-- Loading indicator -->
    <v-progress-circular
      v-if="loading"
      indeterminate
      color="primary"
      size="64"
      class="ma-4 d-block mx-auto"
    />

    <!-- Scanner view -->
    <div
      v-show="isScanning"
      id="scanner"
      class="scanner-container"
    ></div>

    <!-- Stop button when scanning -->
    <v-btn
      v-if="isScanning"
      color="error"
      @click="stopScanner"
      block
      large
      class="mt-4"
    >
      <v-icon left>mdi-stop</v-icon>
      Stop Scanner
    </v-btn>
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
      errorMessage: '',
      cameras: [],
      selectedCameraId: null,
      currentStream: null
    };
  },
  beforeDestroy() {
    this.stopScanner();
  },
  mounted() {
    this.detectCameras();
  },
  methods: {
    async detectCameras() {
      try {
        const devices = await navigator.mediaDevices.enumerateDevices();
        this.cameras = devices
          .filter(device => device.kind === 'videoinput')
          .map((camera, index) => ({
            text: camera.label || `Camera ${index + 1}`,
            value: camera.deviceId
          }));
        
        if (this.cameras.length > 0) {
          this.selectedCameraId = this.cameras[0].value;
        }
      } catch (err) {
        console.warn('Camera detection failed:', err);
      }
    },

    async startScanner() {
      this.loading = true;
      this.errorMessage = '';
      this.isScanning = false;

      try {
        // Test basic camera access first
        const testStream = await navigator.mediaDevices.getUserMedia({ 
          video: { 
            deviceId: this.selectedCameraId ? { exact: this.selectedCameraId } : undefined
          } 
        });
        testStream.getTracks().forEach(track => track.stop());

        // Load scanner library
        const Html5Qrcode = await this.$getHtml5Qrcode();
        this.html5QrCode = new Html5Qrcode("scanner");

        // Scanner configuration
        const config = {
          fps: 10,
          qrbox: this.qrbox,
          aspectRatio: 1.0,
          disableFlip: false
        };

        // Camera constraints
        const constraints = {
          deviceId: this.selectedCameraId ? { exact: this.selectedCameraId } : undefined,
          facingMode: "environment"
        };

        await this.html5QrCode.start(
          constraints,
          config,
          this.onScanSuccess,
          this.onScanError
        );

        this.isScanning = true;
      } catch (err) {
        this.handleScannerError(err);
      } finally {
        this.loading = false;
      }
    },

    onScanSuccess(decodedText) {
      console.log("Scanned result:", decodedText);
      this.$emit("scanned", decodedText);
      this.stopScanner(); // Optional: stop after successful scan
    },

    onScanError(errorMessage) {
      console.warn("QR scan error:", errorMessage);
      // Continue scanning despite errors
    },

    handleScannerError(err) {
      console.error("Scanner error:", err);
      
      if (err.name === 'NotAllowedError') {
        this.errorMessage = 'Camera access was denied. Please allow camera permissions.';
      } else if (err.name === 'NotFoundError') {
        this.errorMessage = 'No camera found on this device.';
      } else if (err.name === 'NotSupportedError') {
        this.errorMessage = 'QR scanning is not supported in your browser.';
      } else {
        this.errorMessage = `Scanner error: ${err.message || 'Unknown error'}`;
      }
      
      this.$emit('error', err);
    },

    async stopScanner() {
      if (this.html5QrCode && this.isScanning) {
        try {
          await this.html5QrCode.stop();
          this.html5QrCode.clear();
        } catch (err) {
          console.error("Error stopping scanner:", err);
        } finally {
          this.isScanning = false;
          this.html5QrCode = null;
        }
      }
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
}
</style>