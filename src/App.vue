<script setup>
// Import necessary Vue functions for reactivity and lifecycle management
import { ref, computed, onMounted, watch } from 'vue'
// Import the LessonCard component for displaying individual lesson details
import LessonCard from './components/LessonCard.vue'

// --- STATE VARIABLES ---
// 'lessons' will hold the array of lesson objects fetched from the server
const lessons = ref([])
// 'cart' will store the IDs of lessons added by the user
const cart = ref([])
// 'showLessonsPage' toggles between the main lesson view (true) and checkout view (false)
const showLessonsPage = ref(true)

// --- FILTER & SORT STATE ---
// Stores the attribute to sort by (subject, location, price, spaces)
const sortAttribute = ref('subject')
// Stores the sort direction (ascending or descending)
const sortOrder = ref('ascending')
// Stores the user's search input
const searchTerm = ref('')

// --- CHECKOUT FORM STATE ---
const orderName = ref('')
const orderPhone = ref('')

// --- CONFIGURATION ---
// The base URL for our backend API on Render.
// This is used for all fetch requests to retrieve lessons and submit orders.
const BASE_URL = 'https://coursework-backend-qzv7.onrender.com'

// --- LIFECYCLE HOOKS ---
// onMounted runs automatically when the app starts.
// We use it to fetch the initial list of all lessons from the backend.
onMounted(() => {
  fetch(`${BASE_URL}/lessons`)
    .then((response) => response.json())
    .then((data) => {
      lessons.value = data // Store the fetched data in our reactive variable
    })
    .catch((error) => console.error('Error fetching lessons:', error))
})

// --- WATCHERS ---
// Watch the 'searchTerm' variable. Whenever the user types something, this runs.
// This implements the "Search as you type" requirement.
watch(searchTerm, (newTerm) => {
  if (newTerm.trim().length > 0) {
    // If there is text, send a request to the backend search endpoint
    fetch(`${BASE_URL}/search?q=${newTerm}`)
      .then((response) => response.json())
      .then((data) => {
        lessons.value = data // Update the list with the search results
      })
      .catch((error) => console.error('Error searching lessons:', error))
  } else {
    // If the search box is cleared, fetch the full list of lessons again
    fetch(`${BASE_URL}/lessons`)
      .then((response) => response.json())
      .then((data) => {
        lessons.value = data
      })
      .catch((error) => console.error('Error resetting lessons:', error))
  }
})

// --- METHODS ---

// Adds a lesson ID to the cart and decreases the local availability counter
function addToCart(lesson) {
  cart.value.push(lesson.id)
  lesson.spaces -= 1
}

// Toggles the view between the product list and the shopping cart
function toggleCartPage() {
  showLessonsPage.value = !showLessonsPage.value
}

// Removes an item from the cart and adds the space back to the lesson
function removeFromCart(lesson) {
  // Find the index of the first occurrence of this lesson ID in the cart
  const index = cart.value.indexOf(lesson.id)
  if (index > -1) {
    cart.value.splice(index, 1) // Remove the ID from the cart array
    lesson.spaces += 1 // Increase the available spaces visually
  }
}

// Handles the checkout process: saves the order and updates lesson spaces
async function submitOrder() {
  // 1. Create the order object to send to the server
  const order = {
    name: orderName.value,
    phone: orderPhone.value,
    lessonIds: cart.value,
  }

  // Identify which lessons need their space count updated in the database
  const cartItemDetails = cart.value.map((id) => lessons.value.find((l) => l.id === id))

  try {
    // 2. Send the order data to the POST /orders endpoint
    await fetch(`${BASE_URL}/orders`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(order),
    })

    // 3. Update the remaining spaces for each purchased lesson
    // We create an array of fetch promises (PUT requests) to run in parallel
    const updatePromises = cartItemDetails.map((item) => {
      return fetch(`${BASE_URL}/lessons/${item.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ spaces: item.spaces }), // Send the new space count
      })
    })

    // Wait for all space updates to complete
    await Promise.all(updatePromises)

    // 4. Reset the app state after a successful order
    alert('Order submitted successfully!')
    cart.value = []
    orderName.value = ''
    orderPhone.value = ''
    toggleCartPage() // Return to the main page
  } catch (error) {
    console.error('Failed to submit order:', error)
    alert('There was an error submitting your order. Please try again.')
  }
}

// --- COMPUTED PROPERTIES ---

// Returns a sorted version of the 'lessons' array.
// Note: Filtering is handled by the backend (via the watcher), so we only sort here.
const sortedLessons = computed(() => {
  const lessonsArray = [...lessons.value] // Create a copy to avoid mutating state directly

  lessonsArray.sort((a, b) => {
    let comparison = 0
    // Compare values based on the selected attribute
    if (a[sortAttribute.value] > b[sortAttribute.value]) comparison = 1
    else if (a[sortAttribute.value] < b[sortAttribute.value]) comparison = -1

    // Reverse the comparison if sorting in descending order
    return sortOrder.value === 'descending' ? -comparison : comparison
  })

  return lessonsArray
})

// Maps the IDs in the cart array to the full lesson objects for display
const cartItems = computed(() => {
  return cart.value.map((itemId) => lessons.value.find((lesson) => lesson.id === itemId))
})

// Validates the checkout form using Regular Expressions
// Returns true only if Name is letters/spaces and Phone is numbers
const isCheckoutFormValid = computed(() => {
  const nameRegex = /^[A-Za-z\s]+$/
  const phoneRegex = /^[0-9]+$/
  return nameRegex.test(orderName.value) && phoneRegex.test(orderPhone.value)
})
</script>

<template>
  <div class="app-container">
    <header>
      <div class="header-content">
        <h1><i class="fas fa-graduation-cap"></i> After School Activities</h1>
        <button class="cart-btn" @click="toggleCartPage" :disabled="cart.length === 0">
          <i class="fas fa-shopping-cart"></i>
          <span v-if="showLessonsPage">Cart ({{ cart.length }})</span>
          <span v-else>Back to Lessons</span>
        </button>
      </div>
    </header>

    <main>
      <div id="lessons-page" v-if="showLessonsPage">
        <aside class="controls">
          <div class="filter-card">
            <h2><i class="fas fa-filter"></i> Filter & Sort</h2>

            <div class="control-group">
              <label for="search">Search</label>
              <div class="input-icon-wrapper">
                <i class="fas fa-search input-icon"></i>
                <input type="text" id="search" placeholder="Find lessons..." v-model="searchTerm" />
              </div>
            </div>

            <div class="control-group">
              <label for="sort-attribute">Sort by</label>
              <div class="select-wrapper">
                <select id="sort-attribute" v-model="sortAttribute">
                  <option value="subject">Subject</option>
                  <option value="location">Location</option>
                  <option value="price">Price</option>
                  <option value="spaces">Availability</option>
                </select>
              </div>
            </div>

            <div class="control-group radio-group">
              <label class="radio-label">
                <input type="radio" name="sort-order" value="ascending" v-model="sortOrder" />
                <span>Ascending</span>
              </label>
              <label class="radio-label">
                <input type="radio" name="sort-order" value="descending" v-model="sortOrder" />
                <span>Descending</span>
              </label>
            </div>
          </div>
        </aside>

        <div id="lessons-container">
          <LessonCard
            v-for="lesson in sortedLessons"
            :key="lesson.id"
            :lesson="lesson"
            @add-to-cart="addToCart(lesson)"
          />
        </div>
      </div>

      <div id="checkout-page" v-else>
        <div class="checkout-container">
          <div class="cart-section">
            <h2>Your Shopping Cart</h2>
            <div v-if="cart.length === 0" class="empty-cart">
              <i class="fas fa-shopping-basket fa-3x"></i>
              <p>Your cart is empty.</p>
              <button class="secondary-btn" @click="toggleCartPage">Go Back</button>
            </div>
            <div v-else class="cart-items-list">
              <div v-for="item in cartItems" :key="item.id" class="cart-item">
                <div class="item-info">
                  <i :class="item.icon" class="item-icon"></i>
                  <div>
                    <h3>{{ item.subject }}</h3>
                    <p class="item-location">{{ item.location }}</p>
                  </div>
                </div>
                <div class="item-actions">
                  <span class="item-price">£{{ item.price }}</span>
                  <button class="remove-btn" @click="removeFromCart(item)">
                    <i class="fas fa-trash"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>

          <div class="checkout-section" v-if="cart.length > 0">
            <h2>Checkout Details</h2>
            <div class="checkout-form">
              <div class="form-group">
                <label for="name">Full Name</label>
                <input type="text" id="name" placeholder="e.g. John Doe" v-model="orderName" />
                <small>Letters and spaces only</small>
              </div>
              <div class="form-group">
                <label for="phone">Phone Number</label>
                <input type="text" id="phone" placeholder="e.g. 07123456789" v-model="orderPhone" />
                <small>Numbers only</small>
              </div>
              <button class="checkout-btn" :disabled="!isCheckoutFormValid" @click="submitOrder">
                Complete Order <i class="fas fa-check"></i>
              </button>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
/* Global Reset & Base Styles */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;

  /* --- BACKGROUND IMAGE UPDATE --- */
  /* This loads the image directly from your Render Backend */
  background-image: url('https://coursework-backend-qzv7.onrender.com/images/background.png');
  background-size: cover; /* Ensures image covers the whole screen */
  background-position: center; /* Centers the image */
  background-attachment: fixed; /* Keeps background still when scrolling */
  background-repeat: no-repeat; /* Prevents tiling */
  background-color: #f5f7fa; /* Fallback color */

  color: #2c3e50;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px 40px;
}

/* Header Styles */
header {
  /* Added transparency to header so background shows through slightly */
  background: linear-gradient(135deg, rgba(52, 152, 219, 0.95), rgba(44, 62, 80, 0.95));
  color: white;
  padding: 1rem 2rem;
  border-radius: 0 0 15px 15px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
  backdrop-filter: blur(5px); /* Adds a modern blur effect */
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

header h1 {
  margin: 0;
  font-size: 1.8rem;
  font-weight: 600;
}

.cart-btn {
  background-color: white;
  color: #3498db;
  border: none;
  padding: 0.6rem 1.2rem;
  border-radius: 25px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.cart-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.cart-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

/* Main Layout */
main {
  min-height: 60vh;
}

#lessons-page {
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 2rem;
}

/* Sidebar Controls */
.filter-card {
  background: rgba(255, 255, 255, 0.95); /* Slight transparency */
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
  position: sticky;
  top: 20px;
  backdrop-filter: blur(5px);
}

.filter-card h2 {
  font-size: 1.2rem;
  margin-top: 0;
  margin-bottom: 1.5rem;
  color: #2c3e50;
  border-bottom: 2px solid #f0f2f5;
  padding-bottom: 0.5rem;
}

.control-group {
  margin-bottom: 1.2rem;
}

.control-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  font-size: 0.9rem;
  color: #666;
}

/* Input Styling */
.input-icon-wrapper {
  position: relative;
}

.input-icon {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: #aaa;
}

input[type='text'],
select {
  width: 100%;
  padding: 0.6rem 0.6rem 0.6rem 2.2rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 0.95rem;
  transition: border-color 0.3s;
}

select {
  padding-left: 0.8rem;
}

input:focus,
select:focus {
  outline: none;
  border-color: #3498db;
}

/* Radio Buttons */
.radio-group {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.radio-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  font-weight: normal !important;
}

.radio-label input {
  margin-right: 0.4rem;
}

/* Lessons Grid */
#lessons-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 1.5rem;
}

/* Checkout Page Styles */
.checkout-container {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 2rem;
}

.cart-section,
.checkout-section {
  background: rgba(255, 255, 255, 0.95); /* Slight transparency */
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
  backdrop-filter: blur(5px);
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border-bottom: 1px solid #eee;
  transition: background 0.2s;
}

.cart-item:last-child {
  border-bottom: none;
}

.item-info {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.item-icon {
  color: #3498db;
  font-size: 1.2rem;
  width: 30px;
  text-align: center;
}

.item-info h3 {
  margin: 0;
  font-size: 1rem;
  color: #2c3e50;
}

.item-location {
  margin: 0;
  font-size: 0.85rem;
  color: #888;
}

.item-price {
  font-weight: bold;
  color: #2c3e50;
  margin-right: 1rem;
}

.remove-btn {
  background: #ff7675;
  color: white;
  border: none;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  cursor: pointer;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.remove-btn:hover {
  background: #d63031;
}

/* Checkout Form */
.form-group {
  margin-bottom: 1.5rem;
}

.form-group input {
  padding: 0.8rem;
}

.form-group small {
  display: block;
  margin-top: 0.3rem;
  color: #999;
  font-size: 0.8rem;
}

.checkout-btn {
  width: 100%;
  padding: 1rem;
  background-color: #27ae60;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.3s;
  margin-top: 1rem;
}

.checkout-btn:hover:not(:disabled) {
  background-color: #219150;
}

.checkout-btn:disabled {
  background-color: #95a5a6;
  cursor: not-allowed;
}

/* Empty State */
.empty-cart {
  text-align: center;
  padding: 3rem 0;
  color: #95a5a6;
}

.secondary-btn {
  margin-top: 1rem;
  padding: 0.6rem 1.2rem;
  background: transparent;
  border: 2px solid #3498db;
  color: #3498db;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
}

/* Responsive Adjustments */
@media (max-width: 768px) {
  #lessons-page,
  .checkout-container {
    grid-template-columns: 1fr;
  }

  .header-content {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
}
</style>
