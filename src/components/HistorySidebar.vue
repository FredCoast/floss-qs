<template>
  <div v-if="showHistory" class="sidebar-overlay">
    <aside class="sidebar">
      <div class="sidebar-header">
        File History
        <button class="sidebar-upload-btn" @click="$emit('new')">
          <img src="../assets/upload.svg" alt="Upload" style="width:14px;height:14px;margin-right:4px;filter:brightness(0) invert(1);" />
          Upload
        </button>
        <button class="close-btn" @click="$emit('close')">&times;</button>
      </div>
      <div class="sidebar-list">
        <template v-if="safeFileHistory.length">
          <button
            v-for="(file, idx) in safeFileHistory"
            :key="file.name"
            class="sidebar-item"
            @click="$emit('select-history', idx)"
          >
            <span class="sidebar-filename" :title="file.name">{{ file.name }}</span>
          </button>
        </template>
        <div v-else class="sidebar-empty">No previous files found.</div>
      </div>
    </aside>
    <div class="fade-overlay" @click="$emit('close')"></div>
  </div>
</template>

<script>
export default {
  name: 'HistorySidebar',
  props: {
    showHistory: {
      type: Boolean,
      required: true
    },
    safeFileHistory: {
      type: Array,
      required: true
    }
  }
};
</script>

<style scoped>
.sidebar-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 350;
  display: flex;
}
.fade-overlay {
  position: fixed;
  top: 0;
  left: 320px;
  width: calc(100vw - 320px);
  height: 100vh;
  background: rgba(0,0,0,0.25);
  z-index: 340;
  cursor: pointer;
}
.sidebar {
  width: 320px;
  height: 100vh;
  background: #fff;
  color: #2c3e50;
  box-shadow: 2px 0 16px rgba(0,0,0,0.10);
  display: flex;
  flex-direction: column;
  padding-top: 38px;
  position: relative;
}
.sidebar-header {
  font-weight: 600;
  font-size: 1.1em;
  padding: 18px 18px 10px 18px;
  border-bottom: 1px solid #eee;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.sidebar-upload-btn {
  background-color: #3660af;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 6px 12px;
  font-size: 0.9em;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.sidebar-upload-btn:hover {
  background-color: #274b8c;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.3em;
  cursor: pointer;
  color: #888;
  padding: 4px;
  transition: color 0.2s;
  flex-shrink: 0;
}

.close-btn:hover {
  color: #555;
}
.sidebar-list {
  display: flex;
  flex-direction: column;
  padding: 10px 0;
}
.sidebar-item {
  background: none;
  border: none;
  text-align: left;
  padding: 10px 18px;
  font-size: 1em;
  color: #2c3e50;
  cursor: pointer;
  border-radius: 0;
  transition: background 0.2s;
}
.sidebar-item:hover {
  background: #f3f3f3;
}
.sidebar-filename {
  display: inline-block;
  max-width: 220px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  vertical-align: middle;
}
.sidebar-empty {
  padding: 10px 18px;
  color: #888;
  font-size: 0.95em;
}
</style>
