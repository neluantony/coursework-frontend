<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import LessonCard from './components/LessonCard.vue'

const lessons = ref([])
const cart = ref([])
const showLessonsPage = ref(true)
const sortAttribute = ref('subject')
const sortOrder = ref('ascending')
const searchTerm = ref('')
const orderName = ref('')
const orderPhone = ref('')

// Define your backend URL here to make it easier to manage
const BASE_URL = 'https://coursework-backend-qzv7.onrender.com'

// 1. Load all lessons when the app starts
onMounted(() => {
  fetch(`${BASE_URL}/lessons`)
    .then((response) => response.json())
    .then((data) => {
      lessons.value = data
    })
    .catch((error) => console.error('Error fetching lessons:', error))
})

// 2. Watch the searchTerm variable. Whenever it changes, query the backend.
watch(searchTerm, (newTerm) => {
  if (newTerm.trim().length > 0) {
    // If there is text, search on the backend using the /search route
    fetch(`${BASE_URL}/search?q=${newTerm}`)
      .then((response) => response.json())
      .then((data) => {
        lessons.value = data
      })
      .catch((error) => console.error('Error searching lessons:', error))
  } else {
    // If the search box is empty, fetch all lessons again
    fetch(`${BASE_URL}/lessons`)
      .then((response) => response.json())
      .then((data) => {
        lessons.value = data
      })
      .catch((error) => console.error('Error resetting lessons:', error))
  }
})

function addToCart(lesson) {
  cart.value.push(lesson.id)
  lesson.spaces -= 1
}

function toggleCartPage() {
  showLessonsPage.value = !showLessonsPage.value
}

function removeFromCart(lesson) {
  const index = cart.value.indexOf(lesson.id)
  if (index > -1) {
    cart.value.splice(index, 1)
    lesson.spaces += 1
  }
}

async function submitOrder() {
  const order = {
    name: orderName.value,
    phone: orderPhone.value,
    lessonIds: cart.value,
  }

  const cartItemDetails = cart.value.map((id) => lessons.value.find((l) => l.id === id))

  try {
    // 1. Save the order
    await fetch(`${BASE_URL}/orders`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(order),
    })

    // 2. Update spaces for each lesson in the cart
    const updatePromises = cartItemDetails.map((item) => {
      return fetch(`${BASE_URL}/lessons/${item.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ spaces: item.spaces }),
      })
    })
    await Promise.all(updatePromises)

    // 3. Handle success
    alert('Order submitted successfully!')
    cart.value = []
    orderName.value = ''
    orderPhone.value = ''
    toggleCartPage()
  } catch (error) {
    console.error('Failed to submit order:', error)
    alert('There was an error submitting your order. Please try again.')
  }
}

// 3. Updated Computed Property: Removed local filtering, kept sorting
const sortedLessons = computed(() => {
  // The 'lessons' array now contains exactly what we want (either all lessons or search results)
  // so we only need to sort it here.
  const lessonsArray = [...lessons.value]

  lessonsArray.sort((a, b) => {
    let comparison = 0
    if (a[sortAttribute.value] > b[sortAttribute.value]) comparison = 1
    else if (a[sortAttribute.value] < b[sortAttribute.value]) comparison = -1
    return sortOrder.value === 'descending' ? -comparison : comparison
  })

  return lessonsArray
})

const cartItems = computed(() => {
  return cart.value.map((itemId) => lessons.value.find((lesson) => lesson.id === itemId))
})

const isCheckoutFormValid = computed(() => {
  const nameRegex = /^[A-Za-z\s]+$/
  const phoneRegex = /^[0-9]+$/
  return nameRegex.test(orderName.value) && phoneRegex.test(orderPhone.value)
})
</script>

<template>
  <header>
    <h1>After School Activities</h1>
    <div class="header-controls">
      <button @click="toggleCartPage" :disabled="cart.length === 0">
        <i class="fas fa-shopping-cart"></i>
        <span v-if="showLessonsPage">Shopping Cart ({{ cart.length }})</span>
        <span v-else>Back to Lessons</span>
      </button>
    </div>
  </header>

  <main>
    <div id="lessons-page" v-if="showLessonsPage">
      <aside class="controls">
        <h2>Filter & Sort</h2>
        <div class="control-group">
          <label for="search">Search:</label>
          <input type="text" id="search" placeholder="Search for lessons..." v-model="searchTerm" />
        </div>
        <div class="control-group">
          <label for="sort-attribute">Sort by:</label>
          <select id="sort-attribute" v-model="sortAttribute">
            <option value="subject">Subject</option>
            <option value="location">Location</option>
            <option value="price">Price</option>
            <option value="spaces">Availability</option>
          </select>
        </div>
        <div class="control-group">
          <label>Order:</label>
          <label
            ><input type="radio" name="sort-order" value="ascending" v-model="sortOrder" />
            Ascending</label
          >
          <label
            ><input type="radio" name="sort-order" value="descending" v-model="sortOrder" />
            Descending</label
          >
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
      <h2>Shopping Cart</h2>
      <div id="cart-items-container">
        <div v-if="cart.length === 0">
          <p>Your cart is empty.</p>
        </div>
        <div v-else v-for="item in cartItems" :key="item.id" class="cart-item">
          <p>{{ item.subject }} - £{{ item.price }}</p>
          <button @click="removeFromCart(item)">Remove</button>
        </div>
      </div>
      <hr v-if="cart.length > 0" />
      <h2>Checkout</h2>
      <div class="checkout-form">
        <p>
          <label for="name"><strong>Name:</strong></label>
          <input type="text" id="name" placeholder="Letters and spaces only" v-model="orderName" />
        </p>
        <p>
          <label for="phone"><strong>Phone:</strong></label>
          <input type="text" id="phone" placeholder="Numbers only" v-model="orderPhone" />
        </p>
        <button :disabled="!isCheckoutFormValid" @click="submitOrder">Checkout</button>
      </div>
    </div>
  </main>
</template>

<style>
/* Global styles for the application */
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  max-width: 1200px;
  margin: 0 auto;
}
main {
  display: flex;
  padding: 1rem;
}
aside.controls {
  margin-right: 2rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  height: fit-content;
}
.control-group {
  margin-bottom: 1rem;
}
#lessons-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}
#checkout-page {
  width: 100%;
}
.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid #eee;
  padding: 0.5rem 1rem;
  margin-bottom: 0.5rem;
  border-radius: 4px;
}
.checkout-form p {
  margin-bottom: 1rem;
}
.checkout-form input {
  width: 250px;
  padding: 0.5rem;
  margin-left: 1rem;
}
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
