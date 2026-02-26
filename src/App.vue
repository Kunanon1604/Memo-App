<script setup>
import { ref, onMounted, watch, computed } from 'vue';

const notes = ref([]);
const searchQuery = ref('');
const activeTab = ref('all'); // 'all', 'pinned', 'categories', 'add'
const categories = ['General', 'Work', 'Personal', 'Idea', 'Urgent'];

const toasts = ref([]);
const showConfirmModal = ref(false);
const noteToDelete = ref(null);

const showToast = (message, type = 'success') => {
  const id = Date.now();
  toasts.value.push({ id, message, type });
  setTimeout(() => {
    toasts.value = toasts.value.filter(t => t.id !== id);
  }, 3000);
};

const confirmDelete = (id) => {
  noteToDelete.value = id;
  showConfirmModal.value = true;
};

const deleteNote = () => {
  if (noteToDelete.value) {
    notes.value = notes.value.filter(n => n.id !== noteToDelete.value);
    showToast('Note deleted', 'danger');
    noteToDelete.value = null;
    showConfirmModal.value = false;
  }
};

const currentNote = ref({
  title: '',
  content: '',
  category: 'General',
  color: '#6366f1',
  isPinned: false,
  attachments: [],
  id: null
});

// Handle File Upload
const handleFileUpload = (event) => {
  const files = event.target.files;
  if (!files.length) return;

  for (let file of files) {
    if (file.size > 1024 * 1024) { // 1MB Limit
      alert(`File ${file.name} is too large. Please choose an image under 1MB.`);
      continue;
    }

    const reader = new FileReader();
    reader.onload = (e) => {
      currentNote.value.attachments.push({
        name: file.name,
        type: file.type,
        data: e.target.result
      });
    };
    reader.readAsDataURL(file);
  }
};

const removeAttachment = (index) => {
  currentNote.value.attachments.splice(index, 1);
};

// Load notes from localStorage
onMounted(() => {
  const savedNotes = localStorage.getItem('notes');
  if (savedNotes) {
    notes.value = JSON.parse(savedNotes);
  }
});

// Watch notes and save to localStorage
watch(notes, (newNotes) => {
  localStorage.setItem('notes', JSON.stringify(newNotes));
}, { deep: true });

// Filtered and Sorted Notes
const filteredNotes = computed(() => {
  let filtered = notes.value;
  
  // Filter by active tab
  if (activeTab.value === 'pinned') {
    filtered = filtered.filter(n => n.isPinned);
  }

  // Filter by search query
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    filtered = filtered.filter(n => 
      n.title.toLowerCase().includes(query) || 
      n.content.toLowerCase().includes(query) ||
      n.category.toLowerCase().includes(query)
    );
  }

  // Sort: Pinned first, then by date (newest first)
  return [...filtered].sort((a, b) => {
    if (a.isPinned === b.isPinned) {
      return new Date(b.date) - new Date(a.date);
    }
    return a.isPinned ? -1 : 1;
  });
});

const saveNote = () => {
  if (!currentNote.value.title && !currentNote.value.content) return;

  if (currentNote.value.id) {
    const index = notes.value.findIndex(n => n.id === currentNote.value.id);
    if (index !== -1) {
      notes.value[index] = { 
        ...currentNote.value, 
        date: new Date().toLocaleString() 
      };
      // Force reactivity update
      notes.value = [...notes.value];
      showToast('Note updated successfully!');
    }
  } else {
    notes.value.unshift({
      ...currentNote.value,
      id: Date.now(),
      date: new Date().toLocaleString()
    });
    showToast('Note saved successfully!');
  }
  
  resetForm();
  activeTab.value = 'all';
};

const togglePin = (note) => {
  const index = notes.value.findIndex(n => n.id === note.id);
  if (index !== -1) {
    notes.value[index] = { 
      ...notes.value[index], 
      isPinned: !notes.value[index].isPinned 
    };
    // Force reactivity update
    notes.value = [...notes.value];
    showToast(notes.value[index].isPinned ? 'Note pinned' : 'Note unpinned', 'info');
  }
};

const editNote = (note) => {
  // Deep clone to avoid reference issues
  const clonedNote = JSON.parse(JSON.stringify(note));
  // Ensure attachments array exists for older notes
  if (!clonedNote.attachments) {
    clonedNote.attachments = [];
  }
  currentNote.value = clonedNote;
  activeTab.value = 'add';
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const resetForm = () => {
  currentNote.value = {
    title: '',
    content: '',
    category: 'General',
    color: '#6366f1',
    isPinned: false,
    attachments: [],
    id: null
  };
};

const switchTab = (tab) => {
  if (tab === 'add') {
    resetForm();
  }
  activeTab.value = tab;
  searchQuery.value = ''; // Clear search when switching tabs to avoid confusion
};
</script>

<template>
  <!-- Toast Notifications -->
  <div class="toast-container position-fixed top-0 start-50 translate-middle-x p-3" style="z-index: 2000">
    <div v-for="toast in toasts" :key="toast.id" 
         class="toast show align-items-center text-white border-0 mb-2 shadow-lg" 
         style="border-radius: 16px; backdrop-filter: blur(10px);"
         :class="'bg-' + toast.type" role="alert">
      <div class="d-flex p-1">
        <div class="toast-body fw-medium">
          {{ toast.message }}
        </div>
        <button type="button" class="btn-close btn-close-white me-2 m-auto" @click="toasts = toasts.filter(t => t.id !== toast.id)"></button>
      </div>
    </div>
  </div>

  <!-- Confirmation Modal -->
  <div v-if="showConfirmModal" class="custom-modal-overlay">
    <div class="custom-modal-content rounded-5 p-4 shadow-2xl bg-white border-0">
      <div class="text-center mb-4">
        <div class="bg-danger bg-opacity-10 d-inline-block p-4 rounded-circle mb-3">
          <i class="bi bi-trash3-fill text-danger fs-1"></i>
        </div>
        <h3 class="fw-bold">Delete Note?</h3>
        <p class="text-secondary px-3">This memory will be permanently removed. This action cannot be undone.</p>
      </div>
      <div class="d-flex gap-3">
        <button @click="deleteNote" class="btn btn-danger w-100 rounded-4 py-3 fw-bold shadow-sm">Delete</button>
        <button @click="showConfirmModal = false" class="btn btn-light w-100 rounded-4 py-3 fw-bold">Cancel</button>
      </div>
    </div>
  </div>

  <div class="container py-5 pb-5 mb-5">
    <div class="row justify-content-center">
      <div class="col-md-11 col-lg-9">
        
        <!-- Premium Header -->
        <div class="d-flex justify-content-between align-items-end mb-5">
          <div>
            <h1 class="fw-black display-6 mb-0" style="background: linear-gradient(135deg, #0f172a 0%, #6366f1 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent;">PureNote</h1>
            <p class="text-secondary small fw-medium mb-0 opacity-75">Your mind, perfectly organized.</p>
          </div>
          <div class="text-end">
            <span class="badge bg-white text-primary border shadow-sm rounded-pill px-3 py-2 fw-bold">
              <i class="bi bi-journal-text me-1"></i> {{ notes.length }}
            </span>
          </div>
        </div>
        
        <!-- Search Bar -->
        <div v-if="activeTab !== 'add'" class="mb-5 animate-in">
          <div class="input-group shadow-lg rounded-4 overflow-hidden border-0 bg-white p-1">
            <span class="input-group-text bg-transparent border-0 ps-3">
              <i class="bi bi-search text-primary fs-5"></i>
            </span>
            <input 
              v-model="searchQuery" 
              type="text" 
              class="form-control border-0 px-3 py-3 fs-6" 
              placeholder="Search through your thoughts..."
              style="background: transparent;"
            >
          </div>
        </div>

        <!-- View: Add/Edit -->
        <div v-if="activeTab === 'add'" class="view-add animate-in">
          <div class="card p-4 border-0 shadow-lg mb-4">
            <div class="d-flex justify-content-between align-items-center mb-4">
              <h4 class="fw-bold mb-0">{{ currentNote.id ? 'Edit Note' : 'New Note' }}</h4>
              <button @click="activeTab = 'all'" class="btn-close"></button>
            </div>
            <div class="row g-4">
              <div class="col-12">
                <input v-model="currentNote.title" type="text" class="form-control fs-5 fw-bold py-3 px-4 rounded-4" placeholder="Note Title">
              </div>
              <div class="col-md-6">
                <label class="small text-secondary fw-bold mb-2 ms-2">CATEGORY</label>
                <select v-model="currentNote.category" class="form-select py-3 px-4 rounded-4 fw-medium">
                  <option v-for="cat in categories" :key="cat" :value="cat">{{ cat }}</option>
                </select>
              </div>
              <div class="col-md-6">
                <label class="small text-secondary fw-bold mb-2 ms-2">THEME COLOR</label>
                <div class="d-flex align-items-center gap-3 bg-light p-2 rounded-4 border">
                  <input v-model="currentNote.color" type="color" class="form-control-color border-0 rounded-circle" style="width: 45px; height: 45px;">
                  <span class="small fw-bold text-muted">{{ currentNote.color.toUpperCase() }}</span>
                </div>
              </div>

              <div class="col-12">
                <label class="small text-secondary fw-bold mb-2 ms-2">CONTENT</label>
                <textarea v-model="currentNote.content" class="form-control py-3 px-4 rounded-4" rows="10" placeholder="Type your thoughts here..."></textarea>
              </div>

              <div class="col-12">
                <div class="bg-light p-4 rounded-4 border border-dashed">
                  <div class="d-flex justify-content-between align-items-center mb-3">
                    <h6 class="fw-bold mb-0"><i class="bi bi-images me-2 text-primary"></i>Media Attachments</h6>
                    <label class="btn btn-primary btn-sm rounded-pill px-4">
                      Upload
                      <input type="file" @change="handleFileUpload" accept="image/*" class="d-none" multiple>
                    </label>
                  </div>
                  <div class="d-flex flex-wrap gap-3">
                    <div v-for="(file, index) in (currentNote.attachments || [])" :key="index" class="position-relative">
                      <img :src="file.data" class="rounded-4 shadow-sm border" style="width: 100px; height: 100px; object-fit: cover;">
                      <button @click="removeAttachment(index)" class="btn btn-danger btn-sm rounded-circle position-absolute top-0 start-100 translate-middle shadow">
                        <i class="bi bi-x"></i>
                      </button>
                    </div>
                    <div v-if="!(currentNote.attachments && currentNote.attachments.length)" class="text-center w-100 py-3 text-muted small opacity-50">
                      No images attached yet.
                    </div>
                  </div>
                </div>
              </div>

              <div class="col-12 d-flex justify-content-between align-items-center mt-5">
                <div class="form-check form-switch bg-light py-2 px-5 rounded-pill border">
                  <input v-model="currentNote.isPinned" class="form-check-input ms-0" type="checkbox" id="pinSwitch">
                  <label class="form-check-label fw-bold small ms-2" for="pinSwitch">Pin to Top</label>
                </div>
                <button @click="saveNote" class="btn btn-primary btn-lg px-5 py-3 rounded-4 shadow-lg">
                  {{ currentNote.id ? 'Update Note' : 'Create Note' }}
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- View: Categories -->
        <div v-else-if="activeTab === 'categories'" class="view-categories animate-in">
          <div class="row g-4">
            <div v-for="cat in categories" :key="cat" class="col-6 col-md-4">
              <div 
                class="card border-0 p-4 text-center cursor-pointer h-100 d-flex flex-column justify-content-center"
                @click="searchQuery = cat; activeTab = 'all'"
              >
                <div class="bg-primary bg-opacity-10 d-inline-block mx-auto p-3 rounded-circle mb-3">
                  <i class="bi bi-hash fs-3 text-primary"></i>
                </div>
                <h5 class="fw-bold mb-1">{{ cat }}</h5>
                <span class="badge bg-light text-secondary border rounded-pill">{{ notes.filter(n => n.category === cat).length }} Notes</span>
              </div>
            </div>
          </div>
        </div>

        <!-- View: Notes List -->
        <div v-else class="view-notes animate-in">
          <div v-if="filteredNotes.length > 0" class="row g-4">
            <div v-for="note in filteredNotes" :key="note.id" class="col-md-6">
              <div class="card h-100 border-0 overflow-hidden" :style="{ borderTop: `8px solid ${note.color}` }">
                <div class="card-body p-4">
                  <div class="d-flex justify-content-between align-items-start mb-3">
                    <span class="badge rounded-pill fw-bold" :style="{ backgroundColor: note.color + '15', color: note.color, border: `1px solid ${note.color}30` }">
                      {{ note.category }}
                    </span>
                    <div class="d-flex gap-2">
                      <i v-if="note.isPinned" class="bi bi-pin-fill text-warning fs-5"></i>
                      <div class="dropdown">
                        <button class="btn btn-light rounded-circle p-0" style="width: 32px; height: 32px;" data-bs-toggle="dropdown">
                          <i class="bi bi-three-dots"></i>
                        </button>
                        <ul class="dropdown-menu dropdown-menu-end rounded-4 shadow-xl border-0 p-2">
                          <li><button @click="togglePin(note)" class="dropdown-item rounded-3"><i class="bi bi-pin me-2 text-warning"></i>{{ note.isPinned ? 'Unpin' : 'Pin' }}</button></li>
                          <li><button @click="editNote(note)" class="dropdown-item rounded-3"><i class="bi bi-pencil-square me-2 text-primary"></i>Edit</button></li>
                          <li><hr class="dropdown-divider opacity-50"></li>
                          <li><button @click="confirmDelete(note.id)" class="dropdown-item rounded-3 text-danger"><i class="bi bi-trash3 me-2"></i>Delete</button></li>
                        </ul>
                      </div>
                    </div>
                  </div>
                  
                  <h4 class="fw-black mb-3">{{ note.title || 'Untitled' }}</h4>
                  
                  <!-- Premium Image Preview -->
                  <div v-if="note.attachments && note.attachments.length" class="row g-2 mb-4">
                    <div v-for="(file, idx) in (note.attachments || []).slice(0, 3)" :key="idx" :class="note.attachments.length === 1 ? 'col-12' : 'col-4'">
                      <img :src="file.data" class="rounded-4 w-100 shadow-sm border" style="height: 120px; object-fit: cover;">
                    </div>
                  </div>

                  <p class="card-text text-secondary mb-4 opacity-75" style="line-height: 1.6; display: -webkit-box; -webkit-line-clamp: 4; -webkit-box-orient: vertical; overflow: hidden;">
                    {{ note.content }}
                  </p>
                  
                  <div class="d-flex justify-content-between align-items-center mt-auto pt-3 border-top border-light">
                    <span class="text-secondary small fw-bold opacity-50"><i class="bi bi-calendar4-event me-1"></i> {{ note.date }}</span>
                    <i class="bi bi-chevron-right text-primary opacity-25"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Empty State -->
          <div v-else class="text-center py-5">
            <div class="bg-white d-inline-block p-5 rounded-circle shadow-lg mb-4">
              <i class="bi bi-journal-x display-1 text-primary opacity-25"></i>
            </div>
            <h2 class="fw-black">Your space is quiet</h2>
            <p class="text-secondary mb-4">No notes found. Tap the button below to start your collection.</p>
            <button v-if="searchQuery" @click="searchQuery = ''" class="btn btn-outline-primary rounded-pill px-5">Clear Search</button>
          </div>
        </div>

      </div>
    </div>
  </div>

  <!-- Premium Bottom Navigation -->
  <nav class="bottom-nav fixed-bottom">
    <div class="container-fluid h-100 px-0">
      <div class="d-flex justify-content-around align-items-center h-100 w-100">
        <button @click="switchTab('all')" class="nav-btn" :class="{ 'active': activeTab === 'all' }">
          <i class="bi" :class="activeTab === 'all' ? 'bi-house-heart-fill' : 'bi-house-heart'"></i>
          <span>Home</span>
        </button>
        <button @click="switchTab('pinned')" class="nav-btn" :class="{ 'active': activeTab === 'pinned' }">
          <i class="bi" :class="activeTab === 'pinned' ? 'bi-star-fill' : 'bi-star'"></i>
          <span>Pinned</span>
        </button>
        
        <div class="nav-center-btn">
          <button @click="switchTab('add')" class="btn btn-primary rounded-circle shadow-lg p-0 d-flex align-items-center justify-content-center" style="width: 56px; height: 56px; margin-top: -40px; border: 4px solid #fff;">
            <i class="bi bi-plus-lg fs-3"></i>
          </button>
        </div>

        <button @click="switchTab('categories')" class="nav-btn" :class="{ 'active': activeTab === 'categories' }">
          <i class="bi" :class="activeTab === 'categories' ? 'bi-collection-fill' : 'bi-collection'"></i>
          <span>Tags</span>
        </button>
        <button @click="searchQuery = ''; activeTab = 'all'" class="nav-btn">
          <i class="bi bi-arrow-repeat"></i>
          <span>Refresh</span>
        </button>
      </div>
    </div>
  </nav>
</template>

<style>
/* Premium Component Styles */
.fw-black { font-weight: 900; }

.animate-in {
  animation: slideUp 0.6s cubic-bezier(0.23, 1, 0.32, 1) both;
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.shadow-2xl {
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
}

.shadow-xl {
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
}

.nav-btn {
  background: none;
  border: none;
  color: #94a3b8;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  transition: all 0.3s ease;
  padding: 8px 4px;
  border-radius: 12px;
  flex: 1;
  max-width: 80px;
}

.nav-btn i { font-size: 1.3rem; }
.nav-btn span { font-size: 0.65rem; font-weight: 800; text-transform: uppercase; letter-spacing: 0.05em; }
.nav-btn.active { color: var(--pure-primary); }

.nav-center-btn {
  flex: 1;
  display: flex;
  justify-content: center;
  max-width: 80px;
}

.border-dashed {
  border: 2px dashed #e2e8f0 !important;
}

.form-check-input:checked {
  background-color: var(--pure-primary);
  border-color: var(--pure-primary);
}

.dropdown-item {
  padding: 10px 15px;
  font-weight: 600;
  font-size: 0.9rem;
}

/* Custom Modal Refinement */
.custom-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 3000;
  padding: 20px;
}

.custom-modal-content {
  max-width: 400px;
  width: 100%;
  animation: modalIn 0.3s ease-out;
}

@keyframes modalIn {
  from { transform: scale(0.9); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.pinned-note { background-color: #fffdf5; border-color: #ffeeba !important; }
.rounded-pill { border-radius: 50rem !important; }

.form-control-color {
  width: 45px;
  height: 45px;
  padding: 0;
  border-radius: 50%;
  cursor: pointer;
  overflow: hidden;
}

.form-control-color::-webkit-color-swatch-wrapper {
  padding: 0;
}

.form-control-color::-webkit-color-swatch {
  border: none;
}
</style>
