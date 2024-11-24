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
      showCart: false,
      lessons: [], // Lessons fetched from the backend
      cart: [],
      sortAttribute: "subject",
      sortOrder: "asc",
      firstName: "",
      lastName: "",
      email: "",
      address: "",
      city: "",
      phone: "",
      isCheckoutEnabled: false,
      searchQuery: "",
      filteredLessons: [],
    };
  },
  computed: {
    totalPrice() {
      return this.cart.reduce((total, lesson) => total + lesson.price, 0);
    },
  },
  methods: {
    async fetchLessons() {
      try {
        const response = await fetch("http://localhost:3000/lessons"); // Updated to use proxy
        if (!response.ok) {
          throw new Error("Failed to fetch lessons");
        }
        const lessons = await response.json();
        this.lessons = lessons;
        this.filteredLessons = lessons; // Initialize filteredLessons
      } catch (error) {
        console.error("Error fetching lessons:", error);
        alert("Failed to load lessons. Please try again later.");
      }
    },
    toggleCart() {
      this.showCart = !this.showCart;
    },
    addToCart(lesson) {
      if (lesson.space > 0) {
        lesson.space--;
        this.cart.push(lesson);
      }
    },
    removeFromCart(lesson, index) {
      lesson.space++;
      this.cart.splice(index, 1);
    },
    sortLessons() {
      const attribute = this.sortAttribute;
      const order = this.sortOrder;

      this.filteredLessons.sort((a, b) => {
        let comparison = 0;
        if (a[attribute] < b[attribute]) {
          comparison = -1;
        } else if (a[attribute] > b[attribute]) {
          comparison = 1;
        }

        return order === "asc" ? comparison : -comparison;
      });
    },
    validateForm() {
  const nameRegex = /^[A-Za-z\s]+$/; // Regex to validate names (letters and spaces only)
  const phoneRegex = /^[0-9]+$/; // Regex to validate phone numbers (digits only)

  // Enable checkout only if First Name and Phone are valid
  this.isCheckoutEnabled =
    nameRegex.test(this.firstName) && phoneRegex.test(this.phone);
},
    async submitOrder() {
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
        "Content-Type": "application/json",
      },
      body: JSON.stringify(orderDetails),
    });

    if (!response.ok) {
      throw new Error("Failed to submit order");
    }

    const data = await response.json();
    alert(`Order submitted successfully! Order ID: ${data.orderId}`);
    this.cart = [];
    this.firstName = "";
    this.lastName = "";
    this.email = "";
    this.address = "";
    this.city = "";
    this.phone = "";
    this.isCheckoutEnabled = true;
  } catch (error) {
    console.error("Error submitting order:", error);
    alert("An error occurred while submitting your order. Please try again.");
  }
},
    filterLessons() {
      const query = this.searchQuery.toLowerCase();
      this.filteredLessons = this.lessons.filter((lesson) =>
        lesson.subject.toLowerCase().includes(query)
      );
    },
  },
  mounted() {
    this.fetchLessons();
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