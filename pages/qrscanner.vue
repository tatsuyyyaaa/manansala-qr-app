<template>
  <v-container class="scanner-wrapper">
    <!-- Scanner Controls -->
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

    <!-- Scanner View -->
    <div
      id="qr-scanner"
      v-show="isScanning"
      class="scanner-view"
    ></div>

    <!-- Scan Results -->
    <v-card
      v-if="lastResult"
      class="mt-4"
    >
      <v-card-title>Scan Result</v-card-title>
      <v-card-text>
        <pre>{{ lastResult }}</pre>
      </v-card-text>
    </v-card>

    <!-- Debug Info -->
    <v-expansion-panels v-if="debug" class="mt-4">
      <v-expansion-panel>
        <v-expansion-panel-header>Debug Info</v-expansion-panel-header>
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
    debug: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      html5QrCode: null,
      isScanning: false,
      loading: false,
      lastResult: null,
      debugInfo: null
    };
  },
  beforeDestroy() {
    this.stopScanner();
  },
  methods: {
    async startScanner() {
      this.loading = true;
      this.lastResult = null;
      this.debugInfo = this.debug ? {} : null;

      try {
        // 1. Load the scanner library
        const Html5Qrcode = await this.loadScannerLibrary();
        
        // 2. Initialize scanner
        this.html5QrCode = new Html5Qrcode("qr-scanner");
        
        // 3. Configure scanner
        const config = {
          fps: 10,
          qrbox: 250,
          aspectRatio: 1.0,
          disableFlip: false
        };

        // 4. Start scanning
        await this.html5QrCode.start(
          { facingMode: "environment" },
          config,
          this.onScanSuccess,
          this.onScanError
        );

        this.isScanning = true;
        if (this.debug) {
          this.debugInfo.status = "Scanner started successfully";
        }
      } catch (err) {
        console.error("Scanner error:", err);
        if (this.debug) {
          this.debugInfo.error = err;
          this.debugInfo.errorDetails = {
            name: err.name,
            message: err.message,
            stack: err.stack
          };
        }
        alert(`Scanner error: ${err.message}`);
      } finally {
        this.loading = false;
      }
    },

    async loadScannerLibrary() {
      try {
        // Method 1: Try plugin first
        if (this.$getHtml5Qrcode) {
          const lib = await this.$getHtml5Qrcode();
          if (lib) return lib;
        }
        
        // Method 2: Direct import fallback
        const { Html5Qrcode } = await import("html5-qrcode");
        return Html5Qrcode;
      } catch (err) {
        throw new Error(`Failed to load scanner library: ${err.message}`);
      }
    },

    onScanSuccess(decodedText) {
      console.log("Scan result:", decodedText);
      this.lastResult = decodedText;
      this.$emit("scanned", decodedText);
      
      // Optional: Stop after successful scan
      // this.stopScanner();
    },

    onScanError(errorMessage) {
      console.warn("Scan error:", errorMessage);
      if (this.debug) {
        this.debugInfo.lastError = errorMessage;
      }
    },

    async stopScanner() {
      if (this.html5QrCode && this.isScanning) {
        try {
          await this.html5QrCode.stop();
          this.html5QrCode.clear();
        } catch (err) {
          console.warn("Error stopping scanner:", err);
        }
      }
      this.isScanning = false;
      this.html5QrCode = null;
    }
  }
};
</script>

<style scoped>
.scanner-wrapper {
  max-width: 600px;
  margin: 0 auto;
}

.scanner-view {
  width: 100%;
  height: 400px;
  border: 2px solid #1976d2;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  background-color: #f5f5f5;
}

.scanner-view::before {
  content: "QR Scanner Active";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #999;
  font-size: 1.2rem;
  z-index: 0;
}

.scanner-view video {
  position: relative;
  z-index: 1;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>