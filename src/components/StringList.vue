<template>
  <div class="table-header desktop-only">
    <div class="header-left">
      <span class="header-title sortable" @click="$emit('sort', 'string')">
        String
        <img v-if="sortField === 'string' && sortDirection === 'asc'" src="@/assets/sort-up.svg" class="sort-icon" alt="Sort ascending" />
        <img v-else-if="sortField === 'string' && sortDirection === 'desc'" src="@/assets/sort-down.svg" class="sort-icon" alt="Sort descending" />
      </span>
    </div>
    <div class="header-right">
      <div class="header-cell tags-cell sortable" @click="$emit('sort', 'tags')">
        Tags
        <img v-if="sortField === 'tags' && sortDirection === 'asc'" src="@/assets/sort-up.svg" class="sort-icon" alt="Sort ascending" />
        <img v-else-if="sortField === 'tags' && sortDirection === 'desc'" src="@/assets/sort-down.svg" class="sort-icon" alt="Sort descending" />
      </div>
      <div class="header-cell offset-cell sortable" @click="$emit('sort', 'offset')">
        Offset
        <img v-if="sortField === 'offset' && sortDirection === 'asc'" src="@/assets/sort-up.svg" class="sort-icon" alt="Sort ascending" />
        <img v-else-if="sortField === 'offset' && sortDirection === 'desc'" src="@/assets/sort-down.svg" class="sort-icon" alt="Sort descending" />
      </div>
      <div class="header-cell structure-cell sortable" @click="$emit('sort', 'structure')">
        Structure
        <img v-if="sortField === 'structure' && sortDirection === 'asc'" src="@/assets/sort-up.svg" class="sort-icon" alt="Sort ascending" />
        <img v-else-if="sortField === 'structure' && sortDirection === 'desc'" src="@/assets/sort-down.svg" class="sort-icon" alt="Sort descending" />
      </div>
      <div class="header-cell location-cell sortable" @click="$emit('sort', 'location')">
        Location
        <img v-if="sortField === 'location' && sortDirection === 'asc'" src="@/assets/sort-up.svg" class="sort-icon" alt="Sort ascending" />
        <img v-else-if="sortField === 'location' && sortDirection === 'desc'" src="@/assets/sort-down.svg" class="sort-icon" alt="Sort descending" />
      </div>
      <div v-if="showEditColumn" class="header-cell actions-cell">
        Edit
      </div>
    </div>
  </div>
  
  <div v-for="(item, idx) in filteredStrings" :key="idx" class="string-block" @click="$emit('toggle-dropdown', idx)" style="cursor:pointer;">
    <div class="string-row">
      <div class="string-left">
        <strong>{{ item.string }}</strong>
      </div>
      <div class="string-right">
        <!-- Desktop view: show all info inline as table -->
        <div class="desktop-info">
          <div class="table-cell tags-cell">
            <div v-if="item.groupedTags && item.groupedTags.length" class="tags-row">
              <span
                v-for="tag in item.groupedTags.filter(tag => !hiddenTags.includes(tag))"
                :key="tag"
                class="tag-pill"
                @click.stop="$emit('filter-tag', tag)"
                style="cursor:pointer;"
              >{{ tag }}</span>
            </div>
          </div>
          <div class="table-cell offset-cell">
            <span v-if="item.offsetRange && showRange">
              <span v-html="formatHex(item.offsetRange[0])"></span><span class="hex-separator">-</span><span v-html="formatHex(item.offsetRange[1])"></span>
            </span>
            <span v-else>
              <span v-html="formatHex(item.offsetRange ? item.offsetRange[0] : item.offset)"></span>
            </span>
          </div>
          <div class="table-cell structure-cell">
            <span>{{ item.structure || '-' }}</span>
          </div>
          <div class="table-cell location-cell">
            <span>{{ item.location || '-' }}</span>
          </div>
          <div v-if="showEditColumn" class="table-cell actions-cell">
            <input 
              type="checkbox" 
              :checked="selectedItems.includes(item.string)"
              @click.stop="$emit('toggle-selection', item.string)"
              class="string-checkbox"
            />
            <button 
              class="edit-btn" 
              @click.stop="$emit('edit-string', item)"
              title="Edit tags"
            >
              <img src="@/assets/edit-pencil.svg" alt="Edit" class="edit-icon" />
            </button>
          </div>
        </div>
        
        <!-- Mobile view: show tags and dropdown button -->
        <div class="mobile-info">
          <div v-if="item.groupedTags && item.groupedTags.length" class="tags-row">
            <span
              v-for="tag in item.groupedTags.filter(tag => !hiddenTags.includes(tag))"
              :key="tag"
              class="tag-pill"
              @click.stop="$emit('filter-tag', tag)"
              style="cursor:pointer;"
            >{{ tag }}</span>
          </div>
          <button class="dropdown-btn" @click.stop="$emit('toggle-dropdown', idx)">
            <span v-if="openDropdown === idx">&#9650;</span>
            <span v-else>&#9660;</span>
          </button>
        </div>
      </div>
    </div>
    <div v-if="openDropdown === idx" class="dropdown-content mobile-only">
      <div v-if="showEditColumn" class="dropdown-actions">
        <div class="dropdown-selection">
          <label>
            <input 
              type="checkbox" 
              :checked="selectedItems.includes(item.string)"
              @change="$emit('toggle-selection', item.string)"
              class="string-checkbox"
            />
            Select
          </label>
        </div>
        <button 
          class="edit-btn-mobile" 
          @click.stop="$emit('edit-string', item)"
          title="Edit tags"
        >
          <img src="@/assets/edit-pencil.svg" alt="Edit" class="edit-icon" />
          Edit Tags
        </button>
      </div>
      <div>
        <strong>Offset:</strong>
        <span v-if="item.offsetRange && showRange">
          <span v-html="formatHex(item.offsetRange[0])"></span><span class="hex-separator">-</span><span v-html="formatHex(item.offsetRange[1])"></span>
        </span>
        <span v-else>
          <span v-html="formatHex(item.offsetRange ? item.offsetRange[0] : item.offset)"></span>
        </span>
      </div>
      <div v-if="item.structure"><strong>Structure:</strong> {{ item.structure }}</div>
      <div v-if="item.location"><strong>Location:</strong> {{ item.location }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StringList',
  props: {
    filteredStrings: Array,
    openDropdown: [Number, null],
    hiddenTags: {
      type: Array,
      default: () => []
    },
    selectedItems: {
      type: Array,
      default: () => []
    },
    sortField: {
      type: String,
      default: null
    },
    sortDirection: {
      type: String,
      default: 'asc'
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
  emits: ['toggle-dropdown', 'filter-tag', 'toggle-selection', 'sort', 'edit-string'],
  methods: {
    formatHex(value) {
      const hex = value.toString(16).toUpperCase().padStart(8, '0');
      const firstNonZeroIndex = hex.search(/[^0]/);
      
      if (firstNonZeroIndex === -1) {
        return `<span class="hex-leading">${hex.slice(0, 7)}</span><span class="hex-relevant">${hex.slice(7)}</span>`;
      } else if (firstNonZeroIndex === 0) {
        return `<span class="hex-relevant">${hex}</span>`;
      } else {
        return `<span class="hex-leading">${hex.slice(0, firstNonZeroIndex)}</span><span class="hex-relevant">${hex.slice(firstNonZeroIndex)}</span>`;
      }
    }
  }
};
</script>
<style scoped>
.table-header {
  display: none;
  padding: 8px 12px;
  background-color: #f5f5f5;
  border-bottom: 2px solid #ddd;
  margin-bottom: 0;
  font-weight: 600;
  color: #2c3e50;
  border-radius: 8px 8px 0 0;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
  min-width: 0;
}

.header-title {
  font-weight: 600;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-shrink: 0;
}

.header-cell {
  font-weight: 600;
  color: #2c3e50;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  position: relative;
}

.header-cell:not(:last-child)::after {
  content: '';
  position: absolute;
  right: -8px;
  top: 20%;
  bottom: 20%;
  width: 1px;
  background-color: #ccc;
}

.sortable {
  cursor: pointer;
  user-select: none;
  transition: color 0.2s;
}

.sortable:hover {
  color: #3660af;
}

.sort-icon {
  width: 12px;
  height: 12px;
  opacity: 0.7;
}

.string-block {
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin-bottom: 8px;
  background-color: #f9f9f9;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
}

.string-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
}

.string-left {
  font-weight: bold;
  flex: 1;
  min-width: 0;
  display: flex;
  align-items: center;
  gap: 8px;
}

.string-actions {
  display: flex;
  align-items: center;
  gap: 6px;
}

.edit-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0.6;
}

.edit-btn:hover {
  background: #e0e0e0;
  opacity: 1;
}

.edit-icon {
  width: 14px;
  height: 14px;
}

.string-checkbox {
  width: 18px;
  height: 18px;
  cursor: pointer;
  accent-color: #3660af;
}

.string-right {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-shrink: 0;
}

.desktop-info {
  display: none;
  align-items: center;
  gap: 12px;
}

.desktop-checkbox-only {
  display: none;
  align-items: center;
}

.table-cell {
  text-align: center;
  font-size: 0.85em;
  min-height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.table-cell:not(:last-child)::after {
  content: '';
  position: absolute;
  right: -8px;
  top: 20%;
  bottom: 20%;
  width: 1px;
  background-color: #e0e0e0;
}

.tags-cell {
  width: 200px;
  justify-content: flex-end;
}

.offset-cell {
  width: 140px;
}

.structure-cell {
  width: 120px;
}

.location-cell {
  width: 120px;
}

.actions-cell {
  width: 80px;
}

.actions-cell .string-checkbox,
.actions-cell .edit-btn {
  margin: 0 2px;
}

.mobile-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.dropdown-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 4px;
  transition: background 0.2s;
  font-size: 1.2em;
  color: #666;
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 24px;
  height: 24px;
}

.dropdown-btn:hover {
  background: #e0e0e0;
}

.dropdown-content {
  margin-top: 8px;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 6px;
  background-color: #fff;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
}

.dropdown-actions {
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid #eee;
}

.dropdown-selection {
  margin-bottom: 8px;
}

.dropdown-selection label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.9em;
  color: #2c3e50;
  cursor: pointer;
}

.edit-btn-mobile {
  background: #3660af;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 8px 12px;
  cursor: pointer;
  font-size: 0.9em;
  font-weight: 500;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  gap: 6px;
}

.edit-btn-mobile:hover {
  background: #274b8c;
}

.edit-btn-mobile .edit-icon {
  width: 14px;
  height: 14px;
}

.dropdown-content > div {
  margin-bottom: 8px;
  line-height: 1.4;
}

.dropdown-content > div:last-child {
  margin-bottom: 0;
}

.dropdown-content strong {
  color: #2c3e50;
  margin-right: 8px;
}

.tags-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tag-pill {
  background: #f0f6ff;
  color: #3660af;
  border: 1.5px solid #3660af;
  border-radius: 16px;
  padding: 4px 14px;
  font-size: 0.98em;
  font-weight: 500;
  display: inline-block;
  margin-bottom: 2px;
  transition: background 0.2s, color 0.2s, border-color 0.2s;
}

.tag-pill:hover {
  filter: brightness(0.85);
}

.hex-leading {
  color: #ccc;
  font-weight: normal;
}

.hex-relevant {
  color: #3660af;
  font-weight: 600;
}

.hex-separator {
  color: #999;
  margin: 0 2px;
}

@media (min-width: 1024px) {
  .table-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 16px;
  }
  
  .desktop-info {
    display: flex;
  }
  
  .desktop-checkbox-only {
    display: flex;
  }
  
  .mobile-info {
    display: none;
  }
  
  .mobile-only {
    display: none !important;
  }
  
  .desktop-only {
    display: flex;
  }
  
  .string-block {
    margin-bottom: 0;
    border-radius: 0;
    border-bottom: none;
    background-color: #fff;
    box-shadow: none;
  }
  
  .string-block:first-child {
    border-top-left-radius: 8px;
    border-top-right-radius: 8px;
  }
  
  .string-block:last-child {
    border-bottom-left-radius: 8px;
    border-bottom-right-radius: 8px;
    border-bottom: 1px solid #ddd;
  }
  
  .string-block:hover {
    background-color: #f8f9fa;
  }
  
  .header-right .tags-cell {
    width: 200px;
  }
  
  .header-right .offset-cell {
    width: 140px;
  }
  
  .header-right .structure-cell {
    width: 120px;
  }
  
  .header-right .location-cell {
    width: 120px;
  }
  
  .header-right .actions-cell {
    width: 80px;
  }
}

@media (max-width: 1023px) {
  .desktop-info {
    display: none;
  }
  
  .desktop-checkbox-only {
    display: none;
  }
  
  .mobile-info {
    display: flex;
  }
  
  .desktop-only {
    display: none;
  }
}
</style>
