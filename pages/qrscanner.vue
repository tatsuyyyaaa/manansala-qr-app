<template>
  <div>
    <div id="qr-reader" style="width: 300px;"></div>
    <p v-if="scannedText">Scanned Code: {{ scannedText }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      scannedText: ''
    };
  },
  mounted() {
    const qrCodeSuccessCallback = (decodedText, decodedResult) => {
      this.scannedText = decodedText;
      this.qrScanner.stop(); // Optional: stop scanning after first result
    };

    this.qrScanner = new this.$Html5Qrcode("qr-reader");

    this.qrScanner.start(
      { facingMode: "environment" }, // or use device ID
      {
        fps: 10,
        qrbox: { width: 250, height: 250 }
      },
      qrCodeSuccessCallback,
      (errorMessage) => {
        // handle scan errors or ignore
        console.warn(errorMessage);
      }
    );
  },
  beforeDestroy() {
    if (this.qrScanner) {
      this.qrScanner.stop().then(() => {
        this.qrScanner.clear();
      });
    }
  }
};
</script>
