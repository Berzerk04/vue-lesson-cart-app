  <template>
    <div class="app">
      <!-- Button to Toggle Between Lesson List and Shopping Cart View -->
      <!-- @click="toggleCart" is a click event that triggers the toggleCart method to switch views -->
      <!-- :disabled is a dynamic attribute that disables the button if certain conditions are met:
          - The button is disabled only when the cart is empty (cart.length === 0) and the user is on the lesson page (!showCart).
          - If the user is on the cart page (showCart is true), the button remains active, allowing a return to lessons even if the cart is empty. -->
      <button
        @click="toggleCart"
        :disabled="!showCart && cart.length === 0"
        class="cart-toggle-btn"
      >
        <!-- Button Text Changes Based on Current View -->
        <!-- If showCart is true (user is viewing the cart), the button text shows "Back to Lessons".
            Otherwise, if showCart is false (user is viewing lessons), the text displays "Go to Shopping Cart". -->
        {{ showCart ? "Back to Lessons" : "Go to Shopping Cart" }}
      </button>

  <!-- Lesson Page Section -->
  <!-- Only displays this section if showCart is false, meaning the user is not viewing the cart page -->
  <div v-if="!showCart" class="lesson-list">
    
    <!-- Main Heading for Available Lessons -->
    <h1>Available Lessons</h1>

    <!-- Sorting Section -->
    <!-- This section allows the user to sort the lessons by various attributes -->
    <div class="sort-options">

      <!-- Label for Sort Attribute Dropdown -->
      <label for="sort-by">Sort by: </label>

      <!-- Dropdown to Select the Attribute to Sort By -->
      <!-- v-model="sortAttribute" binds the selected value to the sortAttribute data property, allowing it to be tracked in the component's state -->
      <!-- @change="sortLessons" triggers the sortLessons method whenever a new attribute is selected, reordering the list accordingly -->
      <select v-model="sortAttribute" @change="sortLessons">
        <!-- Option to Sort by Subject -->
        <option value="subject">Subject</option>
        <!-- Option to Sort by Location -->
        <option value="location">Location</option>
        <!-- Option to Sort by Price -->
        <option value="price">Price</option>
        <!-- Option to Sort by Available Spaces -->
        <option value="spaces">Spaces</option>
      </select>

      <!-- Label for Sort Order Dropdown -->
      <label for="sort-order">Order: </label>

      <!-- Dropdown to Select the Sorting Order (Ascending or Descending) -->
      <!-- v-model="sortOrder" binds the selected value to the sortOrder data property to track the selected order -->
      <!-- @change="sortLessons" triggers the sortLessons method whenever the order changes, applying the selected order to the sorting process -->
      <select v-model="sortOrder" @change="sortLessons">
        <!-- Option to Sort in Ascending Order -->
        <option value="asc">Ascending</option>
        <!-- Option to Sort in Descending Order -->
        <option value="desc">Descending</option>
      </select>

    </div> <!-- End of Sorting Section -->
  </div> <!-- End of Lesson Page Section -->

  <ul>
    <!-- Loop through each lesson in the lessons array to display its details -->
    <li v-for="lesson in lessons" :key="lesson.id" class="lesson-item">
  
      <!-- Display the subject of the lesson as a heading -->
      <h2>{{ lesson.subject }}</h2>
  
      <!-- Display the location of the lesson -->
      <p><strong>Location:</strong> {{ lesson.location }}</p>
  
      <!-- Display the price of the lesson -->
      <p><strong>Price:</strong> ${{ lesson.price }}</p>
  
      <!-- Display the number of spaces left for the lesson -->
      <p><strong>Spaces Left:</strong> {{ lesson.spaces }}</p>
  
      <!-- Button to add the lesson to the cart -->
      <!-- @click="addToCart(lesson)" triggers the addToCart method with the selected lesson as a parameter -->
      <!-- :disabled="lesson.spaces === 0" disables the button if no spaces are left for the lesson, preventing users from adding it to the cart -->
      <button
        @click="addToCart(lesson)"
        :disabled="lesson.spaces === 0"
        class="cart-btn"
      >
        Add to Cart
      </button>
      
    </li>
  </ul>  
</div>
<!-- Cart Page Section -->
<!-- This section displays the shopping cart if showCart is true, indicating the user is viewing their cart -->
<div v-if="showCart" class="cart-page">
  
  <!-- Main Heading for the Cart Page -->
  <h1>Your Shopping Cart</h1>

  <!-- Message for an Empty Cart -->
  <!-- Displays "Your cart is empty." if there are no items in the cart (cart.length is 0) -->
  <p v-if="cart.length === 0">Your cart is empty.</p>

  <!-- List of Items in the Cart -->
  <ul>
    <!-- Loops through each item in the cart array to display details for each one -->
    <li v-for="(item, index) in cart" :key="item.id" class="cart-item">

      <!-- Display the subject of the lesson added to the cart -->
      <h2>{{ item.subject }}</h2>

      <!-- Display the price of the lesson in the cart -->
      <p><strong>Price:</strong> ${{ item.price }}</p>

      <!-- Button to Remove the Lesson from the Cart -->
      <!-- @click="removeFromCart(item, index)" triggers the removeFromCart method, passing both the item and its index to remove it from the cart array -->
      <button @click="removeFromCart(item, index)" class="remove-btn">
        Remove from Cart
      </button>

    </li>
  </ul>

  <!-- Total Price Display -->
  <!-- Shows the total price of items in the cart if the cart contains at least one item -->
  <p v-if="cart.length > 0"><strong>Total:</strong> ${{ totalPrice }}</p>

</div> <!-- End of Cart Page Section -->

<!-- Checkout Form Section -->
<div class="checkout-form">
  
  <!-- Heading for the Checkout Section -->
  <h2>Checkout</h2>

  <!-- Form for User Checkout Information -->
  <!-- @submit.prevent="submitOrder" prevents the default form submission and triggers the submitOrder method to handle order submission -->
  <form @submit.prevent="submitOrder">
    
    <!-- First Name Field -->
    <label for="firstName">First Name:</label>
    <!-- v-model="firstName" binds the input to the firstName data property, updating its value dynamically -->
    <!-- @input="validateForm" triggers the validateForm method whenever the input changes, allowing for form validation in real-time -->
    <input type="text" v-model="firstName" @input="validateForm" />

    <!-- Last Name Field -->
    <label for="lastName">Last Name:</label>
    <!-- Similar to First Name: binds the input to lastName and triggers validation on input -->
    <input type="text" v-model="lastName" @input="validateForm" />

    <!-- Email Field -->
    <label for="email">Email:</label>
    <!-- v-model binds to email data property and validateForm checks for proper format -->
    <input type="email" v-model="email" @input="validateForm" />

    <!-- Address Field -->
    <label for="address">Address:</label>
    <!-- Collects the user's address, with real-time validation -->
    <input type="text" v-model="address" @input="validateForm" />

    <!-- City Field -->
    <label for="city">City:</label>
    <!-- Collects the city and updates city property in data -->
    <input type="text" v-model="city" @input="validateForm" />

    <!-- Phone Field -->
    <label for="phone">Phone:</label>
    <!-- Bound to phone property and checked for numeric format by validateForm -->
    <input type="text" v-model="phone" @input="validateForm" />

    <!-- Submit Button for Checkout -->
    <!-- :disabled="!isCheckoutEnabled" disables the button unless all required fields are valid as checked by validateForm -->
    <button type="submit" :disabled="!isCheckoutEnabled" class="checkout-btn">
      Checkout
    </button>

  </form>
</div> <!-- End of Checkout Form Section -->
</template>

  <script>
  export default {
    data() {
      return {
        showCart: false,
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
        cart: [],
        sortAttribute: 'subject',
        sortOrder: 'asc',
        firstName: '',
        lastName: '',
        email: '',
        address: '',
        city: '',
        phone: '',
        isCheckoutEnabled: false,
      };
    },
    computed: {
      totalPrice() {
        return this.cart.reduce((total, lesson) => total + lesson.price, 0);
      }
    },
    methods: {
      toggleCart() {
        this.showCart = !this.showCart;
      },
      addToCart(lesson) {
        if (lesson.spaces > 0) {
          lesson.spaces--;
          this.cart.push(lesson);
        }
      },
      removeFromCart(lesson, index) {
        lesson.spaces++;
        this.cart.splice(index, 1);
      },
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
      validateForm() {
        const nameRegex = /^[A-Za-z\s]+$/;
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        const phoneRegex = /^[0-9]+$/;

        this.isCheckoutEnabled =
          nameRegex.test(this.firstName) &&
          nameRegex.test(this.lastName) &&
          emailRegex.test(this.email) &&
          this.address.trim() !== '' &&
          this.city.trim() !== '' &&
          phoneRegex.test(this.phone);
      },
      submitOrder() {
        alert(`Order submitted for ${this.firstName} ${this.lastName}, email: ${this.email}, address: ${this.address}, city: ${this.city}, phone: ${this.phone}.`);
        this.cart = [];
        this.firstName = '';
        this.lastName = '';
        this.email = '';
        this.address = '';
        this.city = '';
        this.phone = '';
        this.isCheckoutEnabled = false;
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
