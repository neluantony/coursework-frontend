<script setup>
defineProps({
  lesson: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['addToCart'])

function handleAddToCart() {
  emit('addToCart')
}
</script>

<template>
  <div class="lesson-card">
    <div class="card-header">
      <div class="icon-wrapper">
        <i :class="lesson.icon"></i>
      </div>
      <span class="price-tag">£{{ lesson.price }}</span>
    </div>

    <div class="card-body">
      <h3>{{ lesson.subject }}</h3>
      <div class="info-row">
        <i class="fas fa-map-marker-alt"></i>
        <span>{{ lesson.location }}</span>
      </div>

      <div class="info-row availability" :class="{ 'low-stock': lesson.spaces < 3 }">
        <i class="fas fa-users"></i>
        <span>
          <strong>{{ lesson.spaces }}</strong> spaces left
        </span>
      </div>
    </div>

    <div class="card-footer">
      <button @click="handleAddToCart" :disabled="lesson.spaces === 0">
        <span v-if="lesson.spaces > 0">Add to Cart</span>
        <span v-else>Sold Out</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
.lesson-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  transition:
    transform 0.3s,
    box-shadow 0.3s;
  display: flex;
  flex-direction: column;
  height: 100%;
  border: 1px solid #eee;
}

.lesson-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

.card-header {
  background-color: #f8f9fa;
  padding: 1.5rem;
  text-align: center;
  position: relative;
  border-bottom: 1px solid #eee;
}

.icon-wrapper {
  font-size: 3.5rem;
  color: #3498db;
  margin-bottom: 0.5rem;
}

.price-tag {
  position: absolute;
  top: 15px;
  right: 15px;
  background-color: #2ecc71;
  color: white;
  font-weight: bold;
  padding: 0.3rem 0.8rem;
  border-radius: 20px;
  font-size: 0.9rem;
  box-shadow: 0 2px 5px rgba(46, 204, 113, 0.3);
}

.card-body {
  padding: 1.2rem;
  flex-grow: 1;
}

h3 {
  margin: 0 0 1rem 0;
  font-size: 1.25rem;
  color: #2c3e50;
}

.info-row {
  display: flex;
  align-items: center;
  margin-bottom: 0.5rem;
  color: #666;
  font-size: 0.95rem;
}

.info-row i {
  width: 24px;
  color: #95a5a6;
}

.availability.low-stock {
  color: #e74c3c;
}

.availability.low-stock i {
  color: #e74c3c;
}

.card-footer {
  padding: 1.2rem;
  background-color: #fff;
  border-top: 1px solid #f5f5f5;
}

button {
  width: 100%;
  background-color: #3498db;
  color: white;
  border: none;
  padding: 0.8rem;
  border-radius: 6px;
  font-weight: bold;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.2s;
}

button:hover:not(:disabled) {
  background-color: #2980b9;
}

button:disabled {
  background-color: #bdc3c7;
  cursor: not-allowed;
}
</style>
