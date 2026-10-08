<script setup>
import { computed, onMounted, ref } from 'vue'
import EquipmentItem from './components/EquipmentItem.vue'
import { defaultEquipment, statusOptions } from './data/equipment'

const equipment = ref([])
const selectedStatus = ref('All')
const editingId = ref(null)

const form = ref({
  name: '',
  category: '',
  room: '',
  status: 'Available',
  condition: 'Good',
  quantity: 1,
})

const totals = computed(() => ({
  total: equipment.value.length,
  available: equipment.value.filter((item) => item.status === 'Available').length,
  inUse: equipment.value.filter((item) => item.status === 'In Use').length,
  maintenance: equipment.value.filter((item) => item.status === 'Maintenance').length,
}))

const filteredEquipment = computed(() => {
  if (selectedStatus.value === 'All') {
    return equipment.value
  }

  return equipment.value.filter((item) => item.status === selectedStatus.value)
})

function resetForm() {
  editingId.value = null
  form.value = {
    name: '',
    category: '',
    room: '',
    status: 'Available',
    condition: 'Good',
    quantity: 1,
  }
}

function saveEquipment() {
  localStorage.setItem('class-equipment-system', JSON.stringify(equipment.value))
}

function submitEquipment() {
  const { name, category, room, quantity } = form.value

  if (!name.trim() || !category.trim() || !room.trim()) {
    return
  }

  const payload = {
    ...form.value,
    name: name.trim(),
    category: category.trim(),
    room: room.trim(),
    quantity: Number(quantity) || 1,
  }

  if (editingId.value) {
    equipment.value = equipment.value.map((item) =>
      item.id === editingId.value ? { ...item, ...payload } : item,
    )
  } else {
    equipment.value.unshift({
      id: Date.now(),
      ...payload,
    })
  }

  saveEquipment()
  resetForm()
}

function editEquipment(item) {
  editingId.value = item.id
  form.value = { ...item }
}

function updateEquipmentStatus(id, status) {
  equipment.value = equipment.value.map((item) =>
    item.id === id ? { ...item, status } : item,
  )

  saveEquipment()
}

function deleteEquipment(id) {
  equipment.value = equipment.value.filter((item) => item.id !== id)

  if (editingId.value === id) {
    resetForm()
  }

  saveEquipment()
}

onMounted(() => {
  const savedItemData = localStorage.getItem('class-equipment-system')

  if (savedItemData) {
    try {
      const parsed = JSON.parse(savedItemData)

      if (Array.isArray(parsed) && parsed.length > 0) {
        equipment.value = parsed
        return
      }
    } catch (error) {
      console.error('Failed to load equipment data.', error)
    }
  }

  equipment.value = defaultEquipment
  saveEquipment()
})
</script>

<template>
  <div class="page-shell">
    <header class="topbar">
      <div>
        <p class="eyebrow">Academic Resource Center</p>
        <h1>Class Equipment Management System</h1>
      </div>
      <span class="status-pill">CRUD Dashboard</span>
    </header>

    <section class="summary-grid">
      <article class="summary-card accent">
        <span class="label">Total items</span>
        <strong>{{ totals.total }}</strong>
      </article>
      <article class="summary-card success">
        <span class="label">Available</span>
        <strong>{{ totals.available }}</strong>
      </article>
      <article class="summary-card warning">
        <span class="label">In use</span>
        <strong>{{ totals.inUse }}</strong>
      </article>
      <article class="summary-card danger">
        <span class="label">Maintenance</span>
        <strong>{{ totals.maintenance }}</strong>
      </article>
    </section>

    <section class="panel">
      <div class="panel-header">
        <h2>{{ editingId ? 'Update Equipment' : 'Add New Equipment' }}</h2>

        <label class="filter">
          <span>Filter</span>
          <select v-model="selectedStatus">
            <option value="All">All</option>
            <option v-for="status in statusOptions" :key="status" :value="status">
              {{ status }}
            </option>
          </select>
        </label>
      </div>

      <form class="add-form" @submit.prevent="submitEquipment">
        <input v-model="form.name" type="text" placeholder="Equipment name" />
        <input v-model="form.category" type="text" placeholder="Category" />
        <input v-model="form.room" type="text" placeholder="Room / Lab" />
        <input v-model.number="form.quantity" type="number" min="1" placeholder="Quantity" />

        <select v-model="form.status">
          <option v-for="status in statusOptions" :key="status" :value="status">
            {{ status }}
          </option>
        </select>

        <select v-model="form.condition">
          <option value="Good">Good</option>
          <option value="Fair">Fair</option>
          <option value="Needs Check">Needs Check</option>
        </select>

        <button class="primary-btn" type="submit">
          {{ editingId ? 'Update' : 'Add Equipment' }}
        </button>

        <button v-if="editingId" class="secondary-btn" type="button" @click="resetForm">
          Cancel
        </button>
      </form>
    </section>

    <section class="panel list-panel">
      <h2>Equipment List</h2>

      <div v-if="filteredEquipment.length" class="equipment-list">
        <EquipmentItem
          v-for="item in filteredEquipment"
          :key="item.id"
          :item="item"
          @edit="editEquipment"
          @update-status="updateEquipmentStatus"
          @delete="deleteEquipment"
        />
      </div>

      <div v-else class="empty-state">No equipment matches the selected filter.</div>
    </section>
  </div>
</template>

<style scoped>
.page-shell {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 24px 80px;
}

.topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  margin-bottom: 32px;
}

.eyebrow {
  margin: 0 0 10px;
  font-size: 0.76rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #4b82b8;
  font-weight: 700;
}

h1 {
  margin: 0;
  font-size: clamp(2rem, 5vw, 3rem);
  line-height: 1.1;
  color: #102a43;
}

.status-pill {
  background: rgba(75, 130, 184, 0.12);
  color: #1c4e80;
  padding: 10px 16px;
  border-radius: 999px;
  font-weight: 700;
  white-space: nowrap;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(170px, 1fr));
  gap: 18px;
  margin-bottom: 28px;
}

.summary-card {
  padding: 22px 20px;
  border-radius: 18px;
  background: #ffffff;
  border: 1px solid rgba(15, 42, 67, 0.08);
  box-shadow: 0 16px 32px rgba(16, 42, 67, 0.06);
}

.summary-card strong {
  display: block;
  margin-top: 10px;
  font-size: clamp(1.8rem, 3vw, 2.4rem);
  color: #102a43;
}

.label {
  font-size: 0.76rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #5f7a90;
  font-weight: 700;
}

.summary-card.accent {
  background: linear-gradient(135deg, #edf6ff, #dfeeff);
}

.summary-card.success {
  background: linear-gradient(135deg, #ebfff3, #dff6e8);
}

.summary-card.warning {
  background: linear-gradient(135deg, #fff9eb, #fef0c8);
}

.summary-card.danger {
  background: linear-gradient(135deg, #fff1f2, #fde6e9);
}

.panel {
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(15, 42, 67, 0.08);
  border-radius: 22px;
  box-shadow: 0 20px 40px rgba(15, 42, 67, 0.07);
  padding: 22px;
  margin-bottom: 24px;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  margin-bottom: 20px;
}

.panel h2 {
  margin: 0;
  color: #102a43;
  font-size: 1.5rem;
}

.filter {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  color: #335270;
  font-weight: 600;
}

.filter select,
.add-form input,
.add-form select {
  border: 1px solid #dfeaf3;
  background: #f9fbfd;
  border-radius: 10px;
  padding: 10px 12px;
  color: #102a43;
  outline: none;
}

.add-form {
  display: grid;
  grid-template-columns: repeat(7, minmax(120px, 1fr));
  gap: 12px;
}

.add-form input,
.add-form select {
  width: 100%;
}

.primary-btn,
.secondary-btn {
  border: none;
  border-radius: 10px;
  padding: 11px 14px;
  font-weight: 700;
  cursor: pointer;
}

.primary-btn {
  background: linear-gradient(135deg, #1d4f91, #3c7bc7);
  color: #fff;
}

.secondary-btn {
  background: #edf4fb;
  color: #234c7a;
}

.equipment-list {
  display: grid;
  gap: 16px;
}

.empty-state {
  background: #f7fafc;
  border: 1px dashed #c7d6e5;
  border-radius: 14px;
  padding: 28px 16px;
  text-align: center;
  color: #56708d;
  font-weight: 600;
}

@media (max-width: 900px) {
  .summary-grid,
  .add-form {
    grid-template-columns: repeat(2, minmax(140px, 1fr));
  }
}

@media (max-width: 640px) {
  .topbar,
  .panel-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .summary-grid,
  .add-form {
    grid-template-columns: 1fr;
  }

  .page-shell {
    padding: 24px 16px 48px;
  }
}
</style>
