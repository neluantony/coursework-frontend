<script setup>
// This tells the component what data to expect from its parent (App.vue)
defineProps({
  lesson: {
    type: Object,
    required: true,
  },
})

// This tells the parent what events this component can send up
const emit = defineEmits(['addToCart'])

// When the button is clicked, this function sends the 'addToCart' event to App.vue
function handleAddToCart() {
  emit('addToCart')
}
</script>

<template>
  <div class="lesson-card">
    <div class="lesson-icon">
      <i :class="lesson.icon" class="fa-3x"></i>
    </div>
    <div class="lesson-details">
      <p><strong>Subject:</strong> {{ lesson.subject }}</p>
      <p><strong>Location:</strong> {{ lesson.location }}</p>
      <p><strong>Price:</strong> £{{ lesson.price }}</p>
      <p><strong>Spaces:</strong> {{ lesson.spaces }}</p>
    </div>
    <button @click="handleAddToCart" :disabled="lesson.spaces === 0">Add to Cart</button>
  </div>
</template>

<style scoped>
/* Styles here ONLY apply to this component, preventing style conflicts */
.lesson-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  margin: 0.5rem;
  width: 250px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.lesson-icon {
  text-align: center;
  margin-bottom: 1rem;
  color: #3498db;
}
</style>
