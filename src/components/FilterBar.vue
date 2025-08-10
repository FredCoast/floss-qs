<template>
  <div :class="['filter-bar', { 'disabled-bg': showUploader }]">
      <input
        type="text"
        :value="searchQuery"
        @input="$emit('update:searchQuery', $event.target.value)"
        placeholder="Search strings..."
        class="search-input"
      />
      <div class="range-checkbox">
        <label>
          <input 
            type="checkbox" 
            :checked="showRange"
            @change="$emit('toggle-range')"
          />
          Show Range
        </label>
      </div>
      <div class="edit-toggle-checkbox">
        <label>
          <input 
            type="checkbox" 
            :checked="showEditColumn"
            @change="$emit('toggle-edit-column')"
          />
          Show Edit
        </label>
      </div>
    <div class="filter-controls">
      <button 
        v-if="selectedItems.length > 0" 
        class="add-tag-btn" 
        @click="$emit('show-add-tag-modal')"
      >
        Add Tag ({{ selectedItems.length }})
      </button>
      <button 
        class="export-btn" 
        @click="$emit('export-data')"
        title="Export current data as JSON"
      >
        Export
      </button>
      <div class="tag-filter-dropdown" @click.stop>
        <button class="dropdown-btn location-btn" @click="$emit('toggle-tag-dropdown')">
          Filter Tags
          <span class="arrow">&#9660;</span>
        </button>
        <div v-if="showTagDropdown" class="tag-filter-bar">
          <button class="filter-all-btn" @click="clearTagFilters">All Tags</button>
          
          <div v-for="(categoryTags, categoryName) in categorizedTags" :key="categoryName" class="category-section">
            <div class="category-header">
              <label class="category-label">
                <input 
                  type="checkbox" 
                  :checked="isCategorySelected(categoryTags)" 
                  @change="toggleCategory(categoryTags, $event)"
                />
                <strong>{{ categoryName }}</strong>
              </label>
              <button class="tag-visibility-btn" @click="toggleCategoryVisibility(categoryName)" :aria-label="isCategoryHidden(categoryName) ? 'Show category' : 'Hide category'">
                <img :src="isCategoryHidden(categoryName) ? require('../assets/eye-closed.svg') : require('../assets/eye.svg')" alt="Toggle visibility" style="width:18px;height:18px;" />
              </button>
            </div>
            <div class="category-tags">
              <span v-for="tag in categoryTags.slice(0, 5)" :key="tag" class="tag-filter">
                <label>
                  <input type="checkbox" :value="tag" :checked="tagFilters.includes(tag)" @change="onTagChange(tag, $event)" />
                  {{ tag }}
                </label>
                <span style="flex:1"></span>
                <button class="tag-visibility-btn" @click="toggleTagVisibility(tag)" :aria-label="isTagHidden(tag) ? 'Show tag' : 'Hide tag'">
                  <img :src="isTagHidden(tag) ? require('../assets/eye-closed.svg') : require('../assets/eye.svg')" alt="Toggle visibility" style="width:16px;height:16px;" />
                </button>
              </span>
            </div>
          </div>
          
          <div v-if="uncategorizedTags.length" class="category-section">
            <div class="category-header">
              <strong>Other Tags</strong>
            </div>
            <div class="category-tags">
              <span v-for="tag in uncategorizedTags.slice(0, 5)" :key="tag" class="tag-filter">
                <label>
                  <input type="checkbox" :value="tag" :checked="tagFilters.includes(tag)" @change="onTagChange(tag, $event)" />
                  {{ tag }}
                </label>
                <span style="flex:1"></span>
                <button class="tag-visibility-btn" @click="toggleTagVisibility(tag)" :aria-label="isTagHidden(tag) ? 'Show tag' : 'Hide tag'">
                  <img :src="isTagHidden(tag) ? require('../assets/eye-closed.svg') : require('../assets/eye.svg')" alt="Toggle visibility" style="width:16px;height:16px;" />
                </button>
              </span>
            </div>
          </div>
        </div>
      </div>
      <div class="location-select-wrapper">
        <button class="dropdown-btn location-btn" @click="$emit('toggle-location-dropdown')">
          Locations
          <span class="arrow">&#9660;</span>
        </button>
        <div v-if="showLocationDropdown" class="location-dropdown-bar">
          <button class="filter-all-btn" @click="$emit('set-location', '')">All Locations</button>
          <label v-for="loc in allLocations" :key="loc" class="location-option">
            <input type="checkbox" :value="loc" :checked="selectedLocation === loc" @change="$emit('set-location', loc)" />
            {{ loc }}
          </label>
        </div>
      </div>
    </div>
    <div v-if="showTagDropdown || showLocationDropdown" class="dropdown-overlay" @click="$emit('close-dropdowns')"></div>
  </div>
</template>

<script>
export default {
  name: 'FilterBar',
  props: {
    showUploader: Boolean,
    searchQuery: String,
    tagFilters: Array,
    showTagDropdown: Boolean,
    showLocationDropdown: Boolean,
    allTags: Array,
    allLocations: Array,
    hiddenTags: Array,
    hiddenCategories: Array,
    selectedLocation: String,
    categories: {
      type: Object,
      default: () => ({})
    },
    selectedItems: {
      type: Array,
      default: () => []
    },
    showRange: {
      type: Boolean,
      default: false
    },
    showEditColumn: {
      type: Boolean,
      default: true
    }
  },
  emits: [
    'toggle-tag-dropdown',
    'toggle-location-dropdown',
    'set-location',
    'close-dropdowns',
    'update:searchQuery',
    'update:tagFilters',
    'update:hiddenTags',
    'update:hiddenCategories',
    'show-add-tag-modal',
    'export-data',
    'toggle-range',
    'toggle-edit-column'
  ],
  methods: {
    onTagChange(tag, event) {
      const checked = event.target.checked;
      const tags = this.tagFilters || [];
      let newTags;
      if (checked) {
        newTags = tags.includes(tag) ? tags : [...tags, tag];
      } else {
        newTags = tags.filter(t => t !== tag);
      }
      this.$emit('update:tagFilters', newTags);
    },
    clearTagFilters() {
      this.$emit('update:tagFilters', []);
    },
    toggleTagVisibility(tag) {
      const hidden = this.hiddenTags || [];
      let newHidden;
      if (hidden.includes(tag)) {
        newHidden = hidden.filter(t => t !== tag);
      } else {
        newHidden = [...hidden, tag];
      }
      this.$emit('update:hiddenTags', newHidden);
    },
    isTagHidden(tag) {
      return (this.hiddenTags || []).includes(tag);
    },
    toggleCategory(categoryTags, event) {
      const checked = event.target.checked;
      const currentTags = this.tagFilters || [];
      let newTags;
      
      if (checked) {
        const tagsToAdd = categoryTags.filter(tag => !currentTags.includes(tag));
        newTags = [...currentTags, ...tagsToAdd];
      } else {
        newTags = currentTags.filter(tag => !categoryTags.includes(tag));
      }
      this.$emit('update:tagFilters', newTags);
    },
    isCategorySelected(categoryTags) {
      const currentTags = this.tagFilters || [];
      return categoryTags.some(tag => currentTags.includes(tag));
    },
    toggleCategoryVisibility(categoryName) {
      const categoryTags = this.categorizedTags[categoryName] || [];
      const currentHidden = this.hiddenTags || [];
      
      const hasVisibleTags = categoryTags.some(tag => !currentHidden.includes(tag));
      
      if (hasVisibleTags) {
        const newHidden = [...currentHidden];
        categoryTags.forEach(tag => {
          if (!newHidden.includes(tag)) {
            newHidden.push(tag);
          }
        });
        this.$emit('update:hiddenTags', newHidden);
      } else {
        const newHidden = currentHidden.filter(tag => !categoryTags.includes(tag));
        this.$emit('update:hiddenTags', newHidden);
      }
    },
    isCategoryHidden(categoryName) {
      const categoryTags = this.categorizedTags[categoryName] || [];
      const currentHidden = this.hiddenTags || [];
      return categoryTags.length > 0 && categoryTags.every(tag => currentHidden.includes(tag));
    }
  },
  computed: {
    categorizedTags() {
      const result = {};
      for (const [categoryName, tags] of Object.entries(this.categories)) {
        if (Array.isArray(tags)) {
          result[categoryName] = tags.filter(tag => this.allTags.includes(tag));
        }
      }
      return result;
    },
    uncategorizedTags() {
      const categorizedTagsList = Object.values(this.categorizedTags).flat();
      return this.allTags.filter(tag => !categorizedTagsList.includes(tag));
    }
  }
};
</script>
<style scoped>
.filter-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 18px;
  position: sticky;
  top: 64px;
  z-index: 100;
  background: #f3f3f3;
  border-radius: 10px;
  padding: 14px 22px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
  gap: 16px;
}

.filter-controls {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: nowrap;
}

.add-tag-btn {
  background: #28a745;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px 16px;
  font-size: 1em;
  cursor: pointer;
  transition: background 0.2s;
  box-shadow: 0 2px 8px rgba(0,0,0,0.07);
  height: 40px;
  box-sizing: border-box;
  white-space: nowrap;
  font-weight: 500;
}

.add-tag-btn:hover {
  background: #218838;
}

.export-btn {
  background: #6f42c1;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px 16px;
  font-size: 1em;
  cursor: pointer;
  transition: background 0.2s;
  box-shadow: 0 2px 8px rgba(0,0,0,0.07);
  height: 40px;
  box-sizing: border-box;
  white-space: nowrap;
  font-weight: 500;
}

.export-btn:hover {
  background: #5a2d91;
}

.search-input {
  flex: 1;
  min-width: 200px;
  padding: 10px 16px;
  border-radius: 8px;
  border: 1px solid #bbb;
  font-size: 1em;
  outline: none;
  transition: border 0.2s;
  background: #fff;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
  height: 40px;
  box-sizing: border-box;
}

.search-input:focus {
  border-color: #3660af;
}

.range-checkbox {
  display: flex;
  align-items: center;
  margin-left: 12px;
}

.range-checkbox label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.9em;
  color: #2c3e50;
  cursor: pointer;
}

.range-checkbox input[type="checkbox"] {
  width: 16px;
  height: 16px;
  cursor: pointer;
  accent-color: #3660af;
}

.edit-toggle-checkbox {
  display: flex;
  align-items: center;
  margin-left: 12px;
}

.edit-toggle-checkbox label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.9em;
  color: #2c3e50;
  cursor: pointer;
}

.edit-toggle-checkbox input[type="checkbox"] {
  width: 16px;
  height: 16px;
  cursor: pointer;
  accent-color: #3660af;
}

.tag-filter-dropdown {
  position: relative;
}

.tag-filter-bar {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  min-width: 200px;
  padding: 12px;
  background: #fff;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.10);
  z-index: 30;
}

.tag-filter {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 4px 0;
}

.filter-all-btn {
  background: #3660af;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 8px 16px;
  font-size: 0.95em;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
  margin-bottom: 4px;
}

.filter-all-btn:hover {
  background: #274b8c;
}

.category-section {
  margin-bottom: 12px;
  border-bottom: 1px solid #eee;
  padding-bottom: 8px;
}

.category-section:last-child {
  border-bottom: none;
  margin-bottom: 0;
}

.category-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 6px;
  padding: 4px 0;
}

.category-label {
  cursor: pointer;
  font-size: 1em;
  color: #2c3e50;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
}

.category-tags {
  margin-left: 16px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.tag-visibility-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0 4px;
  margin-left: 8px;
  display: flex;
  align-items: center;
}

.tag-filter label {
  cursor: pointer;
  font-size: 1em;
  color: #2c3e50;
  display: flex;
  align-items: center;
  gap: 8px;
}

.tag-filter input[type="checkbox"] {
  margin: 0;
}

.dropdown-btn.location-btn {
  background: #3660af;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px 16px;
  font-size: 1em;
  cursor: pointer;
  transition: background 0.2s;
  box-shadow: 0 2px 8px rgba(0,0,0,0.07);
  display: flex;
  align-items: center;
  gap: 6px;
  height: 40px;
  box-sizing: border-box;
  white-space: nowrap;
}

.dropdown-btn.location-btn:hover {
  background: 3660af;
}

.arrow {
  font-size: 0.8em;
  transition: transform 0.2s;
}

.location-select-wrapper {
  position: relative;
}

.location-dropdown-bar {
  position: absolute;
  top: calc(100% + 8px);
  right: 0;
  min-width: 180px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.10);
  z-index: 30;
  display: flex;
  flex-direction: column;
  padding: 8px;
  gap: 4px;
}

.location-option {
  background: none;
  border: none;
  text-align: left;
  padding: 8px 12px;
  font-size: 1em;
  color: #2c3e50;
  cursor: pointer;
  border-radius: 4px;
  transition: background 0.2s;
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 8px;
}

.location-option:hover {
  background: #f3f3f3;
}

.location-option input[type="checkbox"] {
  margin: 0;
}

.dropdown-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 15;
}
</style>
