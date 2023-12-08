<template>
  <html lang="en">
  <body>
    <div class="container mt-5">
      <div class="card mx-2 mt-5">
        <div class="card-body m-x5">
          <div class="row mx-1">
            <div v-if="errorAlert" class="alert alert-danger alert-dismissible fade show mt-2" role="alert">
              <strong>Error!</strong> {{ errorMessage }}
              <button type="button" class="btn-close" data-bs-dismiss="alert" @click="resetAlert"
                aria-label="Close"></button>
            </div>
            <div v-if="successAlert" class="alert alert-success alert-dismissible fade show mt-2" role="alert">
              <strong>Sucess!</strong> {{ successMessage }}
              <button type="button" class="btn-close" data-bs-dismiss="alert" @click="resetAlert"
                aria-label="Close"></button>
            </div>
          </div>
          <div class="row text-center">
            <div class="col-md-4"></div>
            <div class="col-md-4">
              <div class="input-group">
                <input class="form-control" type="file" id="formFile" @change="previewImage">
                <label class="input-group-text" for="formFile">
                  <i class="bi bi-cloud-upload"></i>
                </label>
              </div>
            </div>
            <div class="col-md-4"></div>
          </div>
          <div class="row text-center mt-2">
            <div class="col-md-4"></div>
            <div class="col-md-4">
              <button class="btn btn-secondary btn-block custom-btn" type="button" @click="uploadImage">
                Kirim
              </button>
            </div>
            <div class="col-md-4"></div>
          </div>
          <div class="row mt-3 text-center">
            <div class="col-md-4"></div>
            <div class="col-md-4 mt-2">
              <span class="mt-2">Pertinjau Gambar :</span>
              <img v-if="imageUrl" :src="imageUrl" alt="Preview" class="mt-3" style="max-width: 100%; height: auto;" />
            </div>
            <div class="col-md-4"></div>
          </div>
          <hr>
          <div v-if="success && uploadedUrls.length > 0" class="mt-5">
            <div class="row">
              <div v-for="(url, index) in uploadedUrls" :key="index" class="col-lg-4 col-md-12 mb-4 mb-lg-0">
                <img :src="url" class="w-100 shadow-1-strong rounded mb-4" :alt="'Image ' + (index + 1)" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </body>

  </html>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      selectedFile: null,
      imageUrl: null,
      errorAlert: false,
      successAlert: false,
      errorMessage: "",
      success: false,
      uploadedUrls: [],
    };
  },
  mounted() {
    this.uploadedUrls = JSON.parse(localStorage.getItem('uploadedUrls')) || [];
    this.success = true
  },
  methods: {
    resetAlert() {
      this.errorAlert = false;
      this.successAlert = false;
    },
    resetInput() {
      this.selectedFile = null;
      this.imageUrl = null;

      const inputElement = document.getElementById("formFile");
      if (inputElement) {
        inputElement.value = "";
      }
    },
    previewImage(event) {
      this.selectedFile = event.target.files[0];

      const allowedTypes = ["image/jpeg", "image/jpg", "image/bmp", "image/png", "image/gif"];
      const maxFileSizeMB = 2;

      if (!allowedTypes.includes(this.selectedFile.type)) {
        this.errorMessage = "Hanya boleh file gambar.";
        this.errorAlert = true;

        this.resetInput();
        return;
      }

      if (this.selectedFile.size > maxFileSizeMB * 1024 * 1024) {
        this.errorMessage = "File gambar terlalu besar.";
        this.errorAlert = true;

        this.resetInput();
        return;
      }

      const reader = new FileReader();
      reader.onload = (e) => {
        this.imageUrl = e.target.result;
        this.errorAlert = false;
      };
      reader.readAsDataURL(this.selectedFile);
    },
    async uploadImage() {
      if (this.errorAlert || !this.selectedFile) {
        this.errorMessage = "Hanya boleh file gambar.";
        this.errorAlert = true;

        this.resetInput();
        return;
      }

      try {
        const formData = new FormData();
        formData.append('image', this.selectedFile);

        const response = await axios.post('https://api.imgbb.com/1/upload', formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
            'Accept': 'application/json'
          },
          params: {
            key: 'beac88d43bcc6ad118d3ef4c7c29b4a3',
          },
        });

        this.saveToLocalStorage(response.data.data.display_url);
        this.uploadedUrls = JSON.parse(localStorage.getItem('uploadedUrls')) || [];
        this.successMessage = "Gambar berhasil diupload.";
        this.success = true;
        this.successAlert = true;

      } catch (error) {
        console.error('error:', error);
        this.errorMessage = "Upload error.";
        this.errorAlert = true;
        this.resetInput();
      }
    },
    saveToLocalStorage(url) {
      const existingUrls = JSON.parse(localStorage.getItem('uploadedUrls')) || [];
      existingUrls.push(url);
      localStorage.setItem('uploadedUrls', JSON.stringify(existingUrls));
      this.resetInput();
    },
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
}
</style>