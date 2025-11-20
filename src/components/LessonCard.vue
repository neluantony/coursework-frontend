<script setup>
// Define the properties this component expects from the parent
defineProps({
  lesson: {
    type: Object, // Expect a lesson object containing subject, price, etc.
    required: true, // The component won't work without this data
  },
})

// Define custom events this component can emit to the parent
const emit = defineEmits(['addToCart'])

// Local handler for the button click
function handleAddToCart() {
  // Emit the 'addToCart' event so the parent component can update the cart state
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
/* Card Container styling for structure and hover effects */
.lesson-card {
  background: white;
  border-radius: 12px;
  overflow: hidden; /* Keeps content inside rounded corners */
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  transition:
    transform 0.3s,
    box-shadow 0.3s;
  display: flex;
  flex-direction: column;
  height: 100%;
  border: 1px solid #eee;
}

/* Lift the card slightly when hovered */
.lesson-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
}

/* Header area with light background */
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

/* Floating Green Price Tag */
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

/* Content Area */
.card-body {
  padding: 1.2rem;
  flex-grow: 1; /* Ensures the footer stays at the bottom */
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
  text-align: center;
  margin-right: 8px;
}

/* Style for low availability (Red warning color) */
.availability.low-stock {
  color: #e74c3c;
}

.availability.low-stock i {
  color: #e74c3c;
}

/* Footer area for the button */
.card-footer {
  padding: 1.2rem;
  background-color: #fff;
  border-top: 1px solid #f5f5f5;
}

/* Action Button Styling */
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
