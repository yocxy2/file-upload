<template>
  <div>
    <input type="file" @change="handleFileChange" />
    <button @click="uploadFile">Upload</button>
    <button @click="downloadFile">Download</button>
  </div>
</template>

<script>
import CryptoJS from "crypto-js";

export default {
  data() {
    return {
      container: {
        file: null,
        hash: ''
      },
      status: ''
    };
  },
  methods: {
    handleFileChange(e) {
      this.container.file = e.target.files[0];
      // Compute file hash here (e.g., using a library like spark-md5)
    },
    async uploadFile() {
      const reader = new FileReader();
      reader.readAsArrayBuffer(this.container.file);
      reader.onload = async () => {
        const wordArray = CryptoJS.lib.WordArray.create(reader.result);
        const encrypted = CryptoJS.AES.encrypt(wordArray, 'your-encryption-key').toString();

        // Replace original file data with encrypted data
        const encryptedFile = new Blob([encrypted], { type: "application/octet-stream" });

        const formData = new FormData();
        formData.append('file', encryptedFile, this.container.file.name);

        await this.request({
          url: "http://localhost:3000/upload",
          method: "POST",
          headers: {
            "content-type": "multipart/form-data"
          },
          data: formData
        });

        this.$message.success("Upload success, check /target directory");
        this.status = 'wait';
      };
    },
    async downloadFile(fileHash) {
      const { data: encryptedData } = await this.request({
        url: "http://localhost:3000/download",
        method: "POST",
        headers: {
          "content-type": "application/json"
        },
        data: JSON.stringify({ fileHash })
      });

      const decrypted = CryptoJS.AES.decrypt(encryptedData, 'your-encryption-key');
      const decryptedWordArray = CryptoJS.lib.WordArray.create(decrypted.toString(CryptoJS.enc.Utf8));

      const blob = new Blob([decryptedWordArray], { type: "application/octet-stream" });
      const link = document.createElement('a');
      link.href = window.URL.createObjectURL(blob);
      link.download = 'filename.apk';
      link.click();
    }
  }
};
</script>
