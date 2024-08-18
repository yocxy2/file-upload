import os
import zipfile

# Create the directory structure and files for the Vue project
project_dir = '/mnt/data/vue-upload-encrypt'
src_dir = os.path.join(project_dir, 'src')
os.makedirs(src_dir, exist_ok=True)

# Create the main Vue component file with the provided content
app_vue_content = """
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
        this.status = Status.wait;
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

      const blob = new Blob([decryptedWordArray], { type: "application/vnd.android.package-archive" });
      const link = document.createElement('a');
      link.href = window.URL.createObjectURL(blob);
      link.download = 'filename.apk';
      link.click();
    }
  }
};
</script>
"""

with open(os.path.join(src_dir, 'App.vue'), 'w') as file:
    file.write(app_vue_content)

# Create a basic package.json file
package_json_content = """
{
  "name": "vue-upload-encrypt",
  "version": "1.0.0",
  "scripts": {
    "serve": "vue-cli-service serve"
  },
  "dependencies": {
    "crypto-js": "^4.1.1",
    "vue": "^2.6.11"
  },
  "devDependencies": {
    "@vue/cli-service": "^4.5.0"
  }
}
"""

with open(os.path.join(project_dir, 'package.json'), 'w') as file:
    file.write(package_json_content)

# Zip the created directory structure
zip_filename = '/mnt/data/vue-upload-encrypt.zip'
with zipfile.ZipFile(zip_filename, 'w', zipfile.ZIP_DEFLATED) as zipf:
    for root, dirs, files in os.walk(project_dir):
        for file in files:
            zipf.write(os.path.join(root, file), os.path.relpath(os.path.join(root, file), project_dir))

zip_filename
