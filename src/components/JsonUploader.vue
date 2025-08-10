<template>
  <div class="json-uploader">
    <h2>Upload a JSON File</h2>
    <input type="file" accept="application/json" @change="handleFileUpload" />
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script>
export default {
  name: 'JsonUploader',
  data() {
    return {
      error: ''
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (e) => {
        try {
          const parsed = JSON.parse(e.target.result);
          this.error = '';
          this.$emit('json-uploaded', parsed);
        } catch (err) {
          this.error = 'Invalid JSON file.';
        }
      };
      reader.readAsText(file);
    }
  }
};
</script>

<style scoped>
.json-uploader {
  margin: 20px 0;
  text-align: center;
}

.json-uploader h2 {
  margin-bottom: 20px;
  color: #2c3e50;
  font-size: 1.4em;
  font-weight: 600;
}

.json-uploader input[type="file"] {
  padding: 12px;
  border: 2px dashed #3660af;
  border-radius: 8px;
  background: #f4f7fb;
  cursor: pointer;
  font-size: 1em;
  margin-bottom: 16px;
  width: 100%;
  box-sizing: border-box;
  transition: border-color 0.2s, background 0.2s;
}

.json-uploader input[type="file"]:hover {
  border-color: 3660af;
  background: #f0f6ff;
}

.error {
  color: #e74c3c;
  margin-top: 12px;
  padding: 8px 12px;
  background: #fdf2f2;
  border: 1px solid #f5c6cb;
  border-radius: 4px;
  font-size: 0.95em;
}

pre {
  background: #f4f4f4;
  padding: 10px;
  border-radius: 4px;
}
</style>
