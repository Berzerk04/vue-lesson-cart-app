<template>
  <div class="app">
    <!-- Shopping Cart Button -->
    <button
      @click="toggleCart"
      :disabled="cart.length === 0"
      class="cart-toggle-btn"
    >
      {{ showCart ? "Back to Lessons" : "Go to Shopping Cart" }}
    </button>

    <!-- Lesson Page -->
    <div v-if="!showCart" class="lesson-list">
      <h1>Available Lessons</h1>
      
      <!-- Sorting Section -->
      <div class="sort-options">
        <label for="sort-by">Sort by: </label>
        <select v-model="sortAttribute" @change="sortLessons">
          <option value="subject">Subject</option>
          <option value="location">Location</option>
          <option value="price">Price</option>
          <option value="spaces">Spaces</option>
        </select>

        <label for="sort-order">Order: </label>
        <select v-model="sortOrder" @change="sortLessons">
          <option value="asc">Ascending</option>
          <option value="desc">Descending</option>
        </select>
      </div>

      <ul>
        <li v-for="lesson in lessons" :key="lesson.id" class="lesson-item">
          <h2>{{ lesson.subject }}</h2>
          <p><strong>Location:</strong> {{ lesson.location }}</p>
          <p><strong>Price:</strong> ${{ lesson.price }}</p>
          <p><strong>Spaces Left:</strong> {{ lesson.spaces }}</p>
          <!-- Add to Cart Button -->
          <button 
            @click="addToCart(lesson)" 
            :disabled="lesson.spaces === 0"
            class="cart-btn">
            Add to Cart
          </button>
        </li>
      </ul>
    </div>

    <!-- Cart Page -->
    <div v-if="showCart" class="cart-page">
      <h1>Your Shopping Cart</h1>
      <p v-if="cart.length === 0">Your cart is empty.</p>
      <ul>
        <li v-for="(item, index) in cart" :key="item.id" class="cart-item">
          <h2>{{ item.subject }}</h2>
          <p><strong>Price:</strong> ${{ item.price }}</p>
          <!-- Remove from Cart Button -->
          <button @click="removeFromCart(item, index)" class="remove-btn">
            Remove from Cart
          </button>
        </li>
      </ul>
      <p v-if="cart.length > 0"><strong>Total:</strong> ${{ totalPrice }}</p>

      <!-- Checkout Form -->
      <div class="checkout-form">
        <h2>Checkout</h2>
        <form @submit.prevent="submitOrder">
          <label for="name">Name:</label>
          <input type="text" v-model="name" @input="validateForm" />

          <label for="phone">Phone:</label>
          <input type="text" v-model="phone" @input="validateForm" />

          <button type="submit" :disabled="!isCheckoutEnabled" class="checkout-btn">
            Checkout
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showCart: false, // Toggle between lesson page and cart page
      lessons: [
        { id: 1, subject: 'Math', location: 'Room 101', price: 50, spaces: 5 },
        { id: 2, subject: 'English', location: 'Room 102', price: 45, spaces: 5 },
        { id: 3, subject: 'Science', location: 'Room 103', price: 60, spaces: 5 },
        { id: 4, subject: 'History', location: 'Room 104', price: 55, spaces: 5 },
        { id: 5, subject: 'Geography', location: 'Room 105', price: 50, spaces: 5 },
        { id: 6, subject: 'Physics', location: 'Room 106', price: 65, spaces: 5 },
        { id: 7, subject: 'Chemistry', location: 'Room 107', price: 70, spaces: 5 },
        { id: 8, subject: 'Biology', location: 'Room 108', price: 60, spaces: 5 },
        { id: 9, subject: 'Art', location: 'Room 109', price: 40, spaces: 5 },
        { id: 10, subject: 'Music', location: 'Room 110', price: 55, spaces: 5 }
      ],
      cart: [], // Lessons added to the cart
      sortAttribute: 'subject', // Default sort by subject
      sortOrder: 'asc', // Default order ascending
      name: '', // User name for checkout
      phone: '', // User phone for checkout
      isCheckoutEnabled: false, // Controls whether checkout is enabled
    };
  },
  computed: {
    // Calculate total price of items in the cart
    totalPrice() {
      return this.cart.reduce((total, lesson) => total + lesson.price, 0);
    }
  },
  methods: {
    // Toggle between the lesson list and shopping cart view
    toggleCart() {
      this.showCart = !this.showCart;
    },
    // Add lesson to the cart and reduce available spaces
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        lesson.spaces--; // Reduce space count
        this.cart.push(lesson); // Add lesson to the cart
      }
    },
    // Remove lesson from the cart and return it to the lesson list
    removeFromCart(lesson, index) {
      lesson.spaces++; // Increase space count
      this.cart.splice(index, 1); // Remove lesson from cart
    },
    // Sort lessons by the selected attribute and order
    sortLessons() {
      const attribute = this.sortAttribute;
      const order = this.sortOrder;

      this.lessons.sort((a, b) => {
        let comparison = 0;
        if (a[attribute] < b[attribute]) {
          comparison = -1;
        } else if (a[attribute] > b[attribute]) {
          comparison = 1;
        }

        return order === 'asc' ? comparison : -comparison;
      });
    },
    // Validate the checkout form for name and phone
    validateForm() {
      const nameRegex = /^[A-Za-z\s]+$/;
      const phoneRegex = /^[0-9]+$/;

      this.isCheckoutEnabled =
        nameRegex.test(this.name) && phoneRegex.test(this.phone);
    },
    // Submit the order
    submitOrder() {
      alert(`Order submitted for ${this.name} with phone ${this.phone}.`);
      this.cart = []; // Clear cart after submission
      this.name = '';
      this.phone = '';
      this.isCheckoutEnabled = false; // Reset checkout form
    }
  }
};
</script>

<style scoped>
.app {
  font-family: Arial, sans-serif;
  padding: 20px;
}

.lesson-list, .cart-page {
  margin-top: 20px;
}

.lesson-item, .cart-item {
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  margin: 10px 0;
  padding: 15px;
  list-style: none;
}

.lesson-item h2, .cart-item h2 {
  font-size: 1.5rem;
  margin-bottom: 10px;
}

.lesson-item p, .cart-item p {
  margin: 5px 0;
}

.cart-btn, .remove-btn, .checkout-btn {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
}

.cart-btn:disabled, .checkout-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.cart-toggle-btn {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
}

.cart-toggle-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.remove-btn {
  background-color: #dc3545;
}

.checkout-form {
  margin-top: 20px;
}

.checkout-form label {
  display: block;
  margin-top: 10px;
}

.checkout-form input {
  padding: 8px;
  margin-top: 5px;
  margin-bottom: 10px;
  width: 100%;
}
</style>
