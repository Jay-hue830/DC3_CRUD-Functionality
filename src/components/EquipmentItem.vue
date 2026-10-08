<script setup>
import { statusOptions } from '../data/equipment'

const props = defineProps({
  item: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['delete', 'edit', 'update-status'])

function updateStatus(event) {
  emit('update-status', props.item.id, event.target.value)
}
</script>

<template>
  <article class="equipment-card">
    <div class="equipment-top">
      <div>
        <p class="equipment-category">{{ item.category }}</p>
        <h3>{{ item.name }}</h3>
      </div>
      <span class="status-badge" :class="item.status.toLowerCase().replace(/\s+/g, '-')">
        {{ item.status }}
      </span>
    </div>

    <div class="metadata">
      <div>
        <span>Room</span>
        <strong>{{ item.room }}</strong>
      </div>
      <div>
        <span>Quantity</span>
        <strong>{{ item.quantity }}</strong>
      </div>
      <div>
        <span>Condition</span>
        <strong>{{ item.condition }}</strong>
      </div>
    </div>

    <div class="card-actions">
      <label class="inline-select">
        <span>Status</span>
        <select :value="item.status" @change="updateStatus">
          <option v-for="status in statusOptions" :key="status" :value="status">
            {{ status }}
          </option>
        </select>
      </label>

      <button class="edit-btn" type="button" @click="emit('edit', item)">Edit</button>
      <button class="delete-btn" type="button" @click="emit('delete', item.id)">Delete</button>
    </div>
  </article>
</template>

<style scoped>
.equipment-card {
  background: #fff;
  border: 1px solid rgba(15, 42, 67, 0.08);
  border-radius: 18px;
  padding: 20px;
  box-shadow: 0 14px 32px rgba(15, 42, 67, 0.05);
}

.equipment-top {
  display: flex;
  justify-content: space-between;
  align-items: start;
  gap: 16px;
  margin-bottom: 18px;
}

.equipment-category {
  margin: 0 0 6px;
  font-size: 0.72rem;
  color: #5e7b97;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  font-weight: 700;
}

h3 {
  margin: 0;
  font-size: 1.4rem;
  color: #102a43;
}

.status-badge {
  border-radius: 999px;
  padding: 8px 12px;
  font-size: 0.8rem;
  font-weight: 700;
  white-space: nowrap;
}

.status-badge.available {
  background: #e3f8eb;
  color: #126b3d;
}

.status-badge.in-use {
  background: #fff4d8;
  color: #8a6000;
}

.status-badge.maintenance {
  background: #ffe4e6;
  color: #a4262c;
}

.metadata {
  display: grid;
  grid-template-columns: repeat(3, minmax(100px, 1fr));
  gap: 14px;
  margin-bottom: 18px;
}

.metadata span,
.inline-select span {
  display: block;
  margin-bottom: 4px;
  font-size: 0.72rem;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #6f8195;
  font-weight: 700;
}

.metadata strong {
  color: #102a43;
  font-size: 1rem;
}

.card-actions {
  display: flex;
  flex-wrap: wrap;
  align-items: end;
  gap: 10px;
}

.inline-select {
  flex: 1 1 200px;
}

.inline-select select,
.edit-btn,
.delete-btn {
  border-radius: 10px;
  border: 1px solid #dfeaf3;
  padding: 10px 12px;
}

.inline-select select {
  width: 100%;
  background: #f7fafc;
  color: #102a43;
}

.edit-btn,
.delete-btn {
  cursor: pointer;
  font-weight: 700;
}

.edit-btn {
  background: #edf4ff;
  color: #1f5ca8;
  border-color: #d8e8ff;
}

.delete-btn {
  background: #fff1f3;
  color: #b42318;
  border-color: #fdd9de;
}
</style>
