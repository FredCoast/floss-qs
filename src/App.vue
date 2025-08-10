<template>
  <div id="app">
    <header class="banner">
      <div class="banner-left">
        <button class="burger-menu" @click="openHistorySidebar">
          <img src="./assets/menu.svg" alt="Menu" style="width:20px;height:20px;" />
        </button>
      </div>
      <span class="banner-title">
        <img src="./assets/floss-logo.png" alt="FLOSS Logo" class="floss-logo" />
      </span>
      <div class="banner-right">
        <button @click="showUploader = true" class="upload-btn">
          <img src="./assets/upload.svg" alt="Upload" style="width:16px;height:16px;margin-right:6px;filter:brightness(0) invert(1);" />
          Upload
        </button>
      </div>
    </header>
    <HistorySidebar
      :showHistory="showHistory"
      :safeFileHistory="safeFileHistory"
      @close="closeHistorySidebar"
      @new="openUploaderFromSidebar"
      @select-history="selectHistoryFile"
    />
    <div v-if="showUploader" class="modal-overlay" @click.self="showUploader = false">
      <div class="modal">
        <button class="close-btn" @click="showUploader = false">&times;</button>
        <JsonUploader @json-uploaded="handleJsonUploaded" />
      </div>
    </div>
    
    <div v-if="showAddTagModal" class="modal-overlay" @click.self="showAddTagModal = false">
      <div class="modal add-tag-modal">
        <button class="close-btn" @click="showAddTagModal = false">&times;</button>
        <h3>Add Tag to Selected Items</h3>
        <p>Adding tag to {{ selectedItems.length }} selected item(s)</p>
        
        <form @submit.prevent="handleAddTag">
          <div class="form-group">
            <label>Tag Name:</label>
            <input 
              type="text" 
              v-model="newTagName" 
              placeholder="Enter tag name" 
              required 
              class="form-input"
            />
          </div>
          
          <div class="form-group">
            <label>Category:</label>
            <select v-model="newTagCategory" class="form-input">
              <option value="">Select existing category</option>
              <option v-for="categoryName in Object.keys(categories)" :key="categoryName" :value="categoryName">
                {{ categoryName }}
              </option>
            </select>
            <input 
              type="text" 
              v-model="newCategoryName" 
              placeholder="Or enter new category name" 
              class="form-input"
            />
          </div>
          
          <div class="form-buttons">
            <button type="button" @click="showAddTagModal = false" class="cancel-btn">Cancel</button>
            <button type="submit" class="submit-btn">Add Tag</button>
          </div>
        </form>
      </div>
    </div>
    
    <div v-if="showEditModal" class="modal-overlay" @click.self="showEditModal = false">
      <div class="modal edit-string-modal">
        <button class="close-btn" @click="showEditModal = false">&times;</button>
        <h3>Edit Tags for String</h3>
        <p class="edit-string-preview">{{ editingItem?.string }}</p>
        
        <form @submit.prevent="saveEditedString">
          <div class="form-group">
            <label>Current Tags:</label>
            <div class="current-tags">
              <span 
                v-for="(tag, index) in editingTags" 
                :key="index" 
                class="edit-tag-pill"
              >
                {{ tag }}
                <button 
                  type="button" 
                  class="remove-tag-btn" 
                  @click="removeEditTag(index)"
                  title="Remove tag"
                >
                  ×
                </button>
              </span>
            </div>
          </div>
          
          <div class="form-group">
            <label>Add New Tag:</label>
            <input 
              type="text" 
              v-model="newEditTag" 
              placeholder="Enter tag name" 
              class="form-input"
              @keypress.enter.prevent="addEditTag"
            />
            
            <label style="margin-top: 12px;">Category:</label>
            <select v-model="editTagCategory" class="form-input">
              <option value="">Select existing category</option>
              <option v-for="categoryName in Object.keys(categories)" :key="categoryName" :value="categoryName">
                {{ categoryName }}
              </option>
            </select>
            <input 
              type="text" 
              v-model="editCategoryName" 
              placeholder="Or enter new category name" 
              class="form-input"
            />
            
            <button 
              type="button" 
              @click="addEditTag" 
              class="add-tag-button"
              :disabled="!newEditTag.trim()"
            >
              Add Tag
            </button>
          </div>
          
          <div class="form-buttons">
            <button type="button" @click="showEditModal = false" class="cancel-btn">Cancel</button>
            <button type="submit" class="submit-btn">Save Changes</button>
          </div>
        </form>
      </div>
    </div>
    
    <div v-if="showNotificationFlag" class="notification">
      <img src="./assets/check-circle-solid.svg" alt="Success" class="notification-icon" />
      <span>{{ notificationMessage }}</span>
    </div>
    
    <div v-if="uploadedStrings.length" :class="['strings-list', { 'disabled-bg': showUploader }]">
      <h2 class="file-name-header" :title="uploadedFileName">{{ uploadedFileName }}</h2>
      <FilterBar
        :showUploader="showUploader"
        :searchQuery="searchQuery"
        :tagFilters="tagFilters"
        :hiddenTags="hiddenTags"
        :hiddenCategories="hiddenCategories"
        :categories="categories"
        :showTagDropdown="showTagDropdown"
        :showLocationDropdown="showLocationDropdown"
        :allTags="allTags"
        :allLocations="allLocations"
        :selectedLocation="locationFilter"
        :selectedItems="selectedItems"
        :showRange="showRange"
        :showEditColumn="showEditColumn"
        @update:searchQuery="searchQuery = $event"
        @update:tagFilters="tagFilters = $event"
        @update:hiddenTags="hiddenTags = $event"
        @update:hiddenCategories="toggleHiddenCategory"
        @toggle-tag-dropdown="showTagDropdown = !showTagDropdown"
        @toggle-location-dropdown="showLocationDropdown = !showLocationDropdown"
        @set-location="(loc) => { locationFilter = loc; showLocationDropdown = false; }"
        @close-dropdowns="() => { showTagDropdown = false; showLocationDropdown = false; }"
        @show-add-tag-modal="showAddTagModal = true"
        @export-data="exportData"
        @toggle-range="showRange = !showRange"
        @toggle-edit-column="showEditColumn = !showEditColumn"
      />
      <StringList
        :filteredStrings="sortedStrings"
        :openDropdown="openDropdown"
        :categories="categories"
        :hiddenTags="hiddenTags"
        :selectedItems="selectedItems"
        :sortField="sortField"
        :sortDirection="sortDirection"
        :showRange="showRange"
        :showEditColumn="showEditColumn"
        @toggle-dropdown="toggleDropdown"
        @filter-tag="filterForTag"
        @toggle-selection="toggleItemSelection"
        @sort="handleSort"
        @edit-string="handleEditString"
      />
    </div>
    <div v-else-if="!showUploader" class="center-upload-box">
      <div
        class="upload-card dashed-upload"
        @dragover.prevent="dragActive = true"
        @dragleave.prevent="dragActive = false"
        @drop.prevent="handleDrop"
        :class="{ 'drag-active': dragActive }"
      >
        <h2>No file loaded</h2>
        <input
          ref="fileInput"
          type="file"
          accept="application/json,.json"
          style="display:none"
          @change="handleFileInput"
        />
        <button class="upload-btn" @click="triggerFileInput">
          <img src="./assets/upload.svg" alt="Upload" style="width:16px;height:16px;margin-right:6px;filter:brightness(0) invert(1);" />
          Upload JSON File
        </button>
        <p style="color:#888;margin-top:8px;">Click or drag a file here to get started.</p>
      </div>
    </div>
  </div>
</template>

<script>


import JsonUploader from './components/JsonUploader.vue';
import FilterBar from './components/FilterBar.vue';
import StringList from './components/StringList.vue';
import HistorySidebar from './components/HistorySidebar.vue';

function setFavicon(src) {
  let link = document.querySelector("link[rel~='icon']");
  if (!link) {
    link = document.createElement('link');
    link.rel = 'icon';
    document.head.appendChild(link);
  }
  link.href = src;
}

export default {
  name: 'App',
  components: {
    JsonUploader,
    FilterBar,
    StringList,
    HistorySidebar
  },
  data() {
    return {
      showUploader: false,
      uploadedStrings: [],
      openDropdown: null,
      tagFilters: [],
      hiddenTags: [],
      hiddenCategories: [],
      showTagDropdown: false,
      showLocationDropdown: false,
      searchQuery: '',
      locationFilter: '',
      uploadedFileName: '',
      fileHistory: [],
      showHistory: false,
      dragActive: false,
      categories: {},
      selectedItems: [],
      showAddTagModal: false,
      newTagName: '',
      newTagCategory: '',
      newCategoryName: '',
      showNotificationFlag: false,
      notificationMessage: '',
      sortField: null,
      sortDirection: 'asc',
      showRange: false,
      showEditColumn: true,
      showEditModal: false,
      editingItem: null,
      editingTags: [],
      newEditTag: '',
      editTagCategory: '',
      editCategoryName: ''
    };
  },
  mounted() {
    const saved = localStorage.getItem('fileHistory');
    if (saved) {
      try {
        const parsed = JSON.parse(saved);
        if (Array.isArray(parsed)) {
          this.fileHistory.splice(0, this.fileHistory.length, ...parsed);
        } else {
          this.fileHistory.splice(0, this.fileHistory.length);
        }
      } catch (e) {
        this.fileHistory.splice(0, this.fileHistory.length);
      }
    } else {
      this.fileHistory.splice(0, this.fileHistory.length);
    }
    setFavicon(require('./assets/logo.png'));
    document.title = this.uploadedFileName || 'FLOSS';
  },
  computed: {
    safeFileHistory() {
      return Array.isArray(this.fileHistory) ? this.fileHistory : [];
    },
    allTags() {
      const tags = new Set();
      this.uploadedStrings.forEach(item => {
        (item.groupedTags || []).forEach(tag => tags.add(tag));
      });
      return Array.from(tags);
    },
    allLocations() {
      const locs = new Set();
      this.uploadedStrings.forEach(item => {
        if (item.location) locs.add(item.location);
      });
      return Array.from(locs);
    },
    filteredStrings() {
      let filtered = this.uploadedStrings;
      if (this.tagFilters.length) {
        filtered = filtered.filter(item =>
          item.groupedTags && item.groupedTags.some(tag => this.tagFilters.includes(tag))
        );
      }
      if (this.locationFilter) {
        filtered = filtered.filter(item => item.location === this.locationFilter);
      }
      if (this.searchQuery.trim()) {
        const q = this.searchQuery.trim().toLowerCase();
        filtered = filtered.filter(item =>
          item.string && item.string.toLowerCase().includes(q)
        );
      }
      return filtered;
    },
    sortedStrings() {
      if (!this.sortField) {
        return this.filteredStrings;
      }
      
      const sorted = [...this.filteredStrings].sort((a, b) => {
        let aValue, bValue;
        
        switch (this.sortField) {
          case 'string':
            aValue = (a.string || '').toLowerCase();
            bValue = (b.string || '').toLowerCase();
            break;
          case 'tags':
            aValue = (a.groupedTags || []).join(',').toLowerCase();
            bValue = (b.groupedTags || []).join(',').toLowerCase();
            break;
          case 'offset':
            aValue = a.offset || 0;
            bValue = b.offset || 0;
            break;
          case 'structure':
            aValue = (a.structure || '').toLowerCase();
            bValue = (b.structure || '').toLowerCase();
            break;
          case 'location':
            aValue = (a.location || '').toLowerCase();
            bValue = (b.location || '').toLowerCase();
            break;
          default:
            return 0;
        }
        
        if (this.sortField === 'offset') {
          return this.sortDirection === 'asc' ? aValue - bValue : bValue - aValue;
        }
        
        if (aValue < bValue) {
          return this.sortDirection === 'asc' ? -1 : 1;
        }
        if (aValue > bValue) {
          return this.sortDirection === 'asc' ? 1 : -1;
        }
        return 0;
      });
      
      return sorted;
    }
  },
  methods: {
    filterForTag(tag) {
      this.tagFilters = [tag];
    },
    openHistorySidebar() {
      this.showHistory = true;
    },
    closeHistorySidebar() {
      this.showHistory = false;
    },
    openUploaderFromSidebar() {
      this.showUploader = true;
      this.showHistory = false;
    },
    toggleHiddenCategory(categoryName) {
      const hidden = this.hiddenCategories || [];
      let newHidden;
      if (hidden.includes(categoryName)) {
        newHidden = hidden.filter(c => c !== categoryName);
      } else {
        newHidden = [...hidden, categoryName];
      }
      this.hiddenCategories = newHidden;
    },
    toggleItemSelection(itemString) {
      const index = this.selectedItems.indexOf(itemString);
      if (index > -1) {
        this.selectedItems.splice(index, 1);
      } else {
        this.selectedItems.push(itemString);
      }
    },
    handleSort(field) {
      if (this.sortField === field) {
        this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
      } else {
        this.sortField = field;
        this.sortDirection = 'asc';
      }
    },
    handleEditString(item) {
      this.editingItem = item;
      this.editingTags = [...(item.groupedTags || [])];
      this.newEditTag = '';
      this.editTagCategory = '';
      this.editCategoryName = '';
      this.showEditModal = true;
    },
    addEditTag() {
      if (!this.newEditTag.trim()) return;
      
      let tagName = this.newEditTag.trim();
      if (!tagName.startsWith('#')) {
        tagName = '#' + tagName;
      }
      
      if (!this.editingTags.includes(tagName)) {
        this.editingTags.push(tagName);
        
        const categoryName = this.editCategoryName.trim() || this.editTagCategory || 'Other';
        
        if (this.editCategoryName.trim() && !this.categories[categoryName]) {
          this.categories = {
            ...this.categories,
            [categoryName]: [tagName]
          };
        } else if (this.categories[categoryName] && !this.categories[categoryName].includes(tagName)) {
          this.categories[categoryName].push(tagName);
        } else if (!this.categories[categoryName]) {
          this.categories = {
            ...this.categories,
            [categoryName]: [tagName]
          };
        }
      }
      
      this.newEditTag = '';
      this.editTagCategory = '';
      this.editCategoryName = '';
    },
    removeEditTag(index) {
      this.editingTags.splice(index, 1);
    },
    saveEditedString() {
      if (!this.editingItem) return;
      
      const itemIndex = this.uploadedStrings.findIndex(str => str.string === this.editingItem.string);
      if (itemIndex > -1) {
        this.uploadedStrings[itemIndex].groupedTags = [...this.editingTags];
      }
      
      this.updateFileHistoryInStorage();
      
      this.showEditModal = false;
      this.editingItem = null;
      this.editingTags = [];
      this.newEditTag = '';
      this.editTagCategory = '';
      this.editCategoryName = '';
      
      this.showNotification('Tags updated successfully');
    },
    handleAddTag() {
      if (!this.newTagName.trim()) {
        alert('Please enter a tag name');
        return;
      }
      
      let tagName = this.newTagName.trim();
      if (!tagName.startsWith('#')) {
        tagName = '#' + tagName;
      }
      
      const categoryName = this.newCategoryName.trim() || this.newTagCategory || 'Other';
      const tagData = {
        name: tagName,
        category: categoryName,
        appliedTo: [...this.selectedItems]
      };
      
      // Add tag to each selected item
      this.selectedItems.forEach(itemString => {
        const item = this.uploadedStrings.find(str => str.string === itemString);
        if (item) {
          if (!item.groupedTags) {
            item.groupedTags = [];
          }
          if (!item.groupedTags.includes(tagName)) {
            item.groupedTags.push(tagName);
          }
        }
      });

      if (this.newCategoryName.trim() && !this.categories[categoryName]) {
        this.categories = {
          ...this.categories,
          [categoryName]: [tagName]
        };
      } else if (this.categories[categoryName] && !this.categories[categoryName].includes(tagName)) {
        this.categories[categoryName].push(tagName);
      }
      
      this.resetAddTagForm();
      this.showAddTagModal = false;
      this.selectedItems = [];
      
      this.updateFileHistoryInStorage();

      alert(`Tag "${tagName}" has been added to ${tagData.appliedTo.length} items`);
    },
    resetAddTagForm() {
      this.newTagName = '';
      this.newTagCategory = '';
      this.newCategoryName = '';
    },
    updateFileHistoryInStorage() {
      const currentFileName = this.uploadedFileName;
      if (currentFileName) {
        const fileIndex = this.fileHistory.findIndex(f => f.name === currentFileName);
        if (fileIndex > -1) {
          const originalData = this.fileHistory[fileIndex].data[currentFileName] || {};
          const updatedData = {
            [currentFileName]: {
              static_strings: this.uploadedStrings.map(item => ({
                string: item.string,
                offset: item.offset,
                tags: item.groupedTags || []
              })),
              ...Object.fromEntries(
                Object.entries(originalData).filter(([key]) => key !== 'static_strings' && key !== 'categories')
              ),
              categories: this.categories
            }
          };
          this.fileHistory[fileIndex].data = updatedData;
          localStorage.setItem('fileHistory', JSON.stringify(this.fileHistory));
        }
      }
    },
    exportData() {
      if (!this.uploadedFileName || !this.uploadedStrings.length) {
        alert('No data to export');
        return;
      }
      const originalData = this.fileHistory.find(f => f.name === this.uploadedFileName)?.data[this.uploadedFileName] || {};
      const exportData = {
        [this.uploadedFileName]: {
          static_strings: this.uploadedStrings.map(item => ({
            string: item.string,
            offset: item.offset,
            tags: item.groupedTags || []
          })),
          ...Object.fromEntries(
            Object.entries(originalData).filter(([key]) => key !== 'static_strings' && key !== 'categories')
          ),
          categories: this.categories
        }
      };
      const jsonString = JSON.stringify(exportData, null, 2);
      const blob = new Blob([jsonString], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${this.uploadedFileName}_exported.json`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
      this.showNotification('Export complete');
    },
    showNotification(message) {
      this.notificationMessage = message;
      this.showNotificationFlag = true;
      setTimeout(() => {
        this.showNotificationFlag = false;
      }, 3000);
    },
    handleJsonUploaded(json) {
      this.showUploader = false;
      this.uploadedStrings = this.extractStrings(json);
      this.openDropdown = null;
      this.searchQuery = '';
      this.tagFilters = [];
      this.locationFilter = '';
      this.showTagDropdown = false;
      this.showLocationDropdown = false;
      let fileName = '';
      let categories = {};
      if (json && typeof json === 'object' && !Array.isArray(json)) {
        fileName = Object.keys(json)[0] || '';
        if (json[fileName] && json[fileName].categories) {
          categories = json[fileName].categories;
        }
      }
      this.uploadedFileName = fileName;
      this.categories = categories;
      this.hiddenCategories = [];
      document.title = fileName || 'FLOSS';
      if (fileName) {
        const exists = this.fileHistory.find(f => f.name === fileName);
        if (!exists) {
          this.fileHistory.unshift({ name: fileName, data: json });
          if (this.fileHistory.length > 10) this.fileHistory.pop();
        }
        localStorage.setItem('fileHistory', JSON.stringify(this.fileHistory));
      }
    },
    selectHistoryFile(idx) {
      const fileObj = this.fileHistory[idx];
      if (fileObj) {
        this.uploadedFileName = fileObj.name;
        this.uploadedStrings = this.extractStrings(fileObj.data);
        let categories = {};
        if (fileObj.data && typeof fileObj.data === 'object' && !Array.isArray(fileObj.data)) {
          const fileName = Object.keys(fileObj.data)[0] || '';
          if (fileObj.data[fileName] && fileObj.data[fileName].categories) {
            categories = fileObj.data[fileName].categories;
          }
        }
        this.categories = categories;
        this.hiddenCategories = [];
      }
      this.showHistory = false;
    },
    extractStrings(json) {
      if (!json) return [];
      let obj = json;
      if (typeof obj === 'object' && !Array.isArray(obj)) {
        const keys = Object.keys(obj);
        if (keys.length === 1 && obj[keys[0]].static_strings) {
          obj = obj[keys[0]];
        }
      }
      if (!obj.static_strings || !Array.isArray(obj.static_strings)) return [];
      const header = (obj.context && obj.context.pe && Array.isArray(obj.context.pe.header)) ? obj.context.pe.header : [];
      const rdata = (obj.context && obj.context.pe && Array.isArray(obj.context.pe[".rdata"])) ? obj.context.pe[".rdata"] : [];
      const allSections = [...header, ...rdata];
      function offsetInRanges(offset, ranges) {
        for (const r of ranges) {
          if (Array.isArray(r)) {
            if (offset >= r[0] && offset <= r[1]) return true;
          }
        }
        return false;
      }
      return obj.static_strings.map(strObj => {
        let structure = null;
        let sectionTags = [];
        let offset = strObj.offset;
        let location = null;
        let offsetRange = null;
        for (let i = 0; i < allSections.length; i++) {
          const section = allSections[i];
          if (section.offsets && offsetInRanges(offset, section.offsets)) {
            if (section.structure) structure = section.structure;
            if (section.tags && section.tags.length) sectionTags = sectionTags.concat(section.tags);
            for (const [contextKey, contextArr] of Object.entries(obj.context?.pe || {})) {
              if (Array.isArray(contextArr) && contextArr.indexOf(section) !== -1) {
                location = `pe.${contextKey}`;
                break;
              }
            }
            for (const r of section.offsets) {
              if (Array.isArray(r) && offset >= r[0] && offset <= r[1]) {
                offsetRange = r;
                break;
              }
            }
          }
        }
        const allTags = [...(strObj.tags || []), ...sectionTags];
        return {
          string: strObj.string,
          offset: strObj.offset,
          offsetRange,
          structure,
          location,
          groupedTags: [...new Set(allTags)],
          fileName: obj.fileName
        };
      });
    },
    toggleDropdown(idx) {
      this.openDropdown = this.openDropdown === idx ? null : idx;
    },
    handleDrop(e) {
      this.dragActive = false;
      const files = e.dataTransfer.files;
      if (files && files.length) {
        this.processFile(files[0]);
      }
    },
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileInput(e) {
      const file = e.target.files[0];
      if (file) {
        this.processFile(file);
      }
      e.target.value = '';
    },
    processFile(file) {
      if (file.type === 'application/json' || file.name.endsWith('.json')) {
        const reader = new FileReader();
        reader.onload = (evt) => {
          try {
            const json = JSON.parse(evt.target.result);
            this.handleJsonUploaded(json);
          } catch (err) {
            alert('Invalid JSON file.');
          }
        };
        reader.readAsText(file);
      } else {
        alert('Please upload a valid JSON file.');
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 38px;
}
.banner {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  z-index: 300;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0;
  background-color: #ddd;
  color: #2c3e50;
  height: 64px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.07);
}

.banner-left {
  display: flex;
  align-items: center;
  height: 100%;
  padding-left: calc(50vw - 600px);
  min-width: 60px;
  justify-content: flex-start;
}


.banner-title {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  height: 100%;
  display: flex;
  align-items: center;
  gap: 16px;
  pointer-events: none;
  z-index: 1;
}

.floss-logo {
  height: 56px;
  width: 56px;
  object-fit: contain;
  margin-right: 8px;
}

.floss-heading {
  font-size: 2.3em;
  font-weight: 700;
  color: #888;
  letter-spacing: 2px;
  margin: 0;
  line-height: 1;
}

.banner-right {
  display: flex;
  align-items: center;
  height: 100%;
  padding-right: calc(50vw - 600px);
  min-width: 60px;
  justify-content: flex-end;
}

.burger-menu {
  background: none;
  border: none;
  color: #fff;
  font-size: 1.5em;
  cursor: pointer;
  padding: 0;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 40px;
}

.burger-icon {
  font-size: 1.5em;
}

.upload-btn {
  background-color: #3660af;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 20px;
  cursor: pointer;
  font-weight: 500;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.upload-btn:hover {
  background-color: #274b8c;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 200;
}

.modal {
  background: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 500px;
  width: 100%;
  position: relative;
  z-index: 100;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  color: #666;
  transition: color 0.2s;
}

.close-btn:hover {
  color: #333;
}

.add-tag-modal {
  max-width: 600px;
}

.edit-string-modal {
  max-width: 700px;
}

.edit-string-preview {
  background: #f8f9fa;
  padding: 12px;
  border-radius: 6px;
  font-family: monospace;
  font-size: 0.95em;
  color: #2c3e50;
  margin-bottom: 20px;
  word-break: break-all;
  border: 1px solid #e0e0e0;
}

.current-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  min-height: 40px;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 6px;
  background: #f9f9f9;
}

.edit-tag-pill {
  background: #f0f6ff;
  color: #3660af;
  border: 1.5px solid #3660af;
  border-radius: 16px;
  padding: 4px 12px 4px 14px;
  font-size: 0.9em;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 6px;
  transition: background 0.2s;
}

.remove-tag-btn {
  background: none;
  border: none;
  color: #3660af;
  cursor: pointer;
  font-size: 1.2em;
  line-height: 1;
  padding: 0;
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background 0.2s;
}

.remove-tag-btn:hover {
  background: rgba(54, 96, 175, 0.2);
}

.add-tag-button {
  background: #28a745;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 8px 16px;
  cursor: pointer;
  font-size: 0.9em;
  margin-top: 8px;
  transition: background 0.2s;
}

.add-tag-button:hover:not(:disabled) {
  background: #218838;
}

.add-tag-button:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.add-tag-modal h3 {
  margin-top: 0;
  margin-bottom: 8px;
  color: #2c3e50;
}

.add-tag-modal p {
  margin-bottom: 20px;
  color: #666;
  font-size: 0.95em;
}

.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: 600;
  color: #2c3e50;
}

.form-input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 1em;
  transition: border-color 0.2s;
  box-sizing: border-box;
}

.form-input:focus {
  outline: none;
  border-color: #3660af;
}

.form-input + .form-input {
  margin-top: 8px;
}

.form-buttons {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  margin-top: 24px;
}

.cancel-btn {
  background: #6c757d;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 20px;
  cursor: pointer;
  transition: background 0.2s;
}

.cancel-btn:hover {
  background: #5a6268;
}

.submit-btn {
  background: #28a745;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 20px;
  cursor: pointer;
  transition: background 0.2s;
}

.submit-btn:hover {
  background: #218838;
}

/* Content Area Styles */
.strings-list {
  margin-top: 10px;
  text-align: left;
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
  padding: 0 20px;
}

.center-upload-box {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 300px;
  height: calc(100vh - 64px);
}
.upload-card {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 16px rgba(0,0,0,0.08);
  padding: 40px 32px 32px 32px;
  text-align: center;
  max-width: 340px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.dashed-upload {
  border: 2.5px dashed #3660af;
  background: #f9f9f9;
  transition: border-color 0.2s, background 0.2s;
}
.dashed-upload.drag-active {
  border-color: #3660af;
  background: #e6f0fa;
}

.file-name-header {
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  display: block;
  margin-bottom: 8px;
  margin-top: 8px;
  font-size: 1.3em;
  font-weight: 600;
  color: #2c3e50;
}

.disabled-bg {
  pointer-events: none;
  user-select: none;
  opacity: 0.6;
}

/* History Dropdown */
.history-dropdown {
  position: absolute;
  top: 38px;
  left: 0;
  background: #fff;
  color: #2c3e50;
  border-radius: 0 0 8px 8px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.10);
  min-width: 180px;
  z-index: 400;
}

.history-list {
  display: flex;
  flex-direction: column;
}

.history-item {
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

.history-item:hover {
  background: #f3f3f3;
}

.history-empty {
  padding: 10px 18px;
  color: #888;
  font-size: 0.95em;
}

h1 {
  display: none;
}

@media (max-width: 1240px) {
  .banner-left {
    padding-left: 20px;
  }
  
  .banner-right {
    padding-right: 20px;
  }
}

.notification {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: #3660af;
  color: white;
  padding: 12px 20px;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.15);
  display: flex;
  align-items: center;
  gap: 10px;
  z-index: 400;
  font-weight: 500;
  animation: slideInBottom 0.3s ease-out;
}

.notification-icon {
  width: 20px;
  height: 20px;
  filter: brightness(0) invert(1);
}

@keyframes slideInBottom {
  from {
    transform: translateY(100%);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
</style>
