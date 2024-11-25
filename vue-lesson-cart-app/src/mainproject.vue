<template>
  <div class="app">
    <!-- Button to toggle between showing the cart and the lessons view -->
    <button
      @click="toggleCart" 
      :disabled="!showCart && cart.length === 0"
      class="cart-toggle-btn"
    >
      {{ showCart ? "Back to Lessons" : "Go to Shopping Cart" }}
    </button>

    <!-- Lesson List -->
    <div v-if="!showCart" class="lesson-list">
      <h1>Available Lessons</h1>

      <!-- Search Section -->
      <div class="search-section">
        <label for="search">Search Lessons:</label>
        <input
          type="text"
          v-model="searchQuery"
          placeholder="Search by subject"
          @input="filterLessons"
        />
      </div>

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
        <li v-for="lesson in filteredLessons" :key="lesson._id" class="lesson-item"> 
          <h2>{{ lesson.subject }}</h2>
          <p><strong>Subject:</strong> {{ lesson.topic }}</p> 
          <p><strong>Location:</strong> {{ lesson.location }}</p>
          <p><strong>Price:</strong> £{{ lesson.price }}</p>
          <p><strong>Spaces Left:</strong> {{ lesson.space }}</p>
          <button
            @click="addToCart(lesson)"
            :disabled="lesson.space === 0"
            class="cart-btn"
          >
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
        <li v-for="(item, index) in cart" :key="item._id" class="cart-item">
          <h2>{{ item.subject }}</h2>
          <p><strong>Subject:</strong> {{ item.topic }}</p>
          <p><strong>Price:</strong> £{{ item.price }}</p>
          <button @click="removeFromCart(item, index)" class="remove-btn">
            Remove from Cart
          </button>
        </li>
      </ul>
      <p v-if="cart.length > 0"><strong>Total:</strong> £{{ totalPrice }}</p>

      <!-- Checkout Form -->
      <div class="checkout-form">
        <h2>Checkout</h2>
        <form @submit.prevent="submitOrder">
          <label for="firstName">First Name:</label>
          <input type="text" v-model="firstName" @input="validateForm" />

          <label for="lastName">Last Name:</label>
          <input type="text" v-model="lastName" />

          <label for="email">Email:</label>
          <input type="email" v-model="email" />

          <label for="address">Address:</label>
          <input type="text" v-model="address" />

          <label for="city">City:</label>
          <input type="text" v-model="city" />

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
      showCart: false, // Tracks whether the cart modal is visible
      lessons: [], // Lessons fetched from the backend
      cart: [], // Stores lessons added to the cart
      sortAttribute: "subject", // Attribute to sort the lessons
      sortOrder: "asc", // Order for sorting (ascending)
      firstName: "", // User's first name input
      lastName: "", // User's last name input
      email: "", // User's email input
      address: "", // User's address input
      city: "", // User's city input
      phone: "", // User's phone number input
      isCheckoutEnabled: false, // This determines if the checkout button is enabled
      searchQuery: "", // User's input for filtering lessons
      filteredLessons: [], // Filtered list of lessons based on the search
    };
  },
  computed: {
    totalPrice() { // Calculates the total price of lessons in the cart
      return this.cart.reduce((total, lesson) => total + lesson.price, 0);
    },
  },
  methods: {
    async fetchLessons() { // Fetches lessons from the backend API
      try {
        const response = await fetch("http://localhost:3000/lessons"); // Sends a GET request to the lessons endpoint
        if (!response.ok) {
          throw new Error("Failed to fetch lessons"); // Handles failed requests
        }
        const lessons = await response.json(); //Parse the JSON responses
        this.lessons = lessons; // Stores fetched lessons in the component state
        this.filteredLessons = lessons; // Initialize filteredLessons
      } catch (error) {
        console.error("Error fetching lessons:", error); //Logs errors to debug
        alert("Failed to load lessons. Please try again later."); // Informs the user of the Error
      }
    },
    toggleCart() { // Toggles the visibility of the cart 
      this.showCart = !this.showCart;
    },
    addToCart(lesson) { // Adds a lesson to the Cart
      if (lesson.space > 0) { // This line ensures that the lesson has available space
        lesson.space--; // Decrements the available space
        this.cart.push(lesson); // Add the lesson to the cart
      }
    },
    removeFromCart(lesson, index) { // Removes a lesson from the cart
      lesson.space++; // Increment the available space
      this.cart.splice(index, 1); // Remove the lesson from the cart at the specified index
    },
    sortLessons() { // Sorts lessons based on the selected attribute and order
      const attribute = this.sortAttribute; // attribute to sort by
      const order = this.sortOrder; // order to sort (ascending or descending)

      this.filteredLessons.sort((a, b) => {
        let comparison = 0; // This initializes the comparison variable to 0
        if (a[attribute] < b[attribute]) {
          comparison = -1;
        } else if (a[attribute] > b[attribute]) {
          comparison = 1;
        }

        return order === "asc" ? comparison : -comparison;
      });
    },
    validateForm() { // Validates the checkout form inputs as required
  const nameRegex = /^[A-Za-z\s]+$/; // Regex to validate names (letters and spaces only)
  const phoneRegex = /^[0-9]+$/; // Regex to validate phone numbers (digits only)

  // Enable checkout only if First Name and Phone are valid
  this.isCheckoutEnabled =
    nameRegex.test(this.firstName) && phoneRegex.test(this.phone);
},
    async submitOrder() { // Submits the user's order to the baclend
  const orderDetails = {
    firstName: this.firstName, 
    lastName: this.lastName, 
    phone: this.phone,
    cart: this.cart.map((item) => ({
      id: item._id,
      subject: item.subject,
      price: item.price,
    })),
    totalPrice: this.totalPrice,
  };

  try {
    const response = await fetch("http://localhost:3000/orders", {
      method: "POST",
      headers: {
        "Content-Type": "application/json", // Content type for JSON
      },
      body: JSON.stringify(orderDetails), // Send order details as JSON
    });

    if (!response.ok) {
      throw new Error("Failed to submit order"); // Handles failed responses
    }

    const data = await response.json(); // Parse the JSON response
    alert(`Order submitted successfully! Order ID: ${data.orderId}`); // Notify the user of successful order which is displayed
    this.cart = []; // Clears the cart
    this.firstName = ""; // Resets the form fields
    this.lastName = "";
    this.email = "";
    this.address = "";
    this.city = "";
    this.phone = "";
    this.isCheckoutEnabled = true; // Re-enable checkout
  } catch (error) {
    console.error("Error submitting order:", error); // Log errors for debugging
    alert("An error occurred while submitting your order. Please try again."); // Notifies the user of the error
  }
},
    filterLessons() { // Filters the lessons based on the search query
      const query = this.searchQuery.toLowerCase(); // Converts the query to lowercase for case-insensitive matching
      this.filteredLessons = this.lessons.filter((lesson) =>
        lesson.subject.toLowerCase().includes(query) // Check if the subject includes the query
      );
    },
  },
  mounted() {
    this.fetchLessons(); // Fetch lessons when the component is mounted
  },
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

.search-section {
  margin: 10px 0;
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