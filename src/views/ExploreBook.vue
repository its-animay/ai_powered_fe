<template>
    <div>
        <NavBar/>
    
    <div class="books-page">
      <h1>Books</h1>
  
      <!-- Filters Section -->
      <div class="filters">
        <input v-model="titleQuery" placeholder="Title" @input="fetchBooks"/>
        
        <select v-model="authorQuery" @change="fetchBooks">
          <option value="">Select Author</option>
          <option v-for="author in authors" :key="author.id" :value="author.name">{{ author.name }}</option>
        </select>
  
        <select v-model="publisherQuery" @change="fetchBooks">
          <option value="">Select Publisher</option>
          <option v-for="publisher in publishers" :key="publisher.id" :value="publisher.name">{{ publisher.name }}</option>
        </select>
  
        <select v-model="categoryQuery" @change="fetchBooks">
          <option value="">Select Category</option>
          <option v-for="category in categories" :key="category.id" :value="category.name">{{ category.name }}</option>
        </select>
      </div>
  
      <!-- Books Display Section -->
      <div class="books-list">
        <div v-for="book in books" :key="book.id" class="book-item">
          <h3>{{ book.title }}</h3>
          <p>Author: {{ book.author_name }}</p>
          <p>Publisher: {{ book.publisher_name }}</p>
          <p>Category: {{ book.category_name }}</p>
          <p>Year: {{ book.year }}</p>
          <p>Rating</p>
          <img :src="book.cover_path" alt="Book Cover"/>
          <button class="borrow-button" @click="openModal(book.id)">Borrow Book</button>

        </div>
      </div>
      <div v-if="showModal" class="modal">
  <div class="modal-content input-group">

    <span class="close" @click="closeModal">&times;</span>
    <h2>Borrow Book</h2>
    <select place-holder='please select one' v-model="borrowDays" >
        <option disabled value="">Please select one</option>
      <option value="2">2 Days</option>
      <option value="7">7 Days</option>
      <option value="15">15 Days</option>
      <option value="21">21 Days</option>
      <option value="30">30 Days</option>
    </select>
    <button class="borrow-button" @click="issueBook">Confirm</button>
  </div>
</div>
<div  v-if="message" :class="['message-widget', messageType]">
  {{ message }}
</div>
</div>

  
      <!-- Pagination Controls -->
      <div class="pagination">
        <button @click="changePage(currentPage - 1)" :disabled="currentPage <= 1">Previous</button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button  @click="changePage(currentPage + 1)" :disabled="currentPage >= totalPages">Next</button>
      </div>
    </div>

  </template>
  
  <script>
  import { defineComponent } from 'vue';
import NavBar from '../components/NavBar.vue';
  
  export default defineComponent({
    components: {
    NavBar,
  },
    data() {
      return {
        books: [],
        authors: [],
        publishers: [],
        categories: [],
        titleQuery: '',
        authorQuery: '',
        publisherQuery: '',
        categoryQuery: '',
        currentPage: 1,
        totalPages: 0,
        showModal: false,
    selectedBookId: null,
    borrowDays: 14,
    message: '', // For success or error messages
        messageType: ''
      };
    },
    mounted() {
      this.fetchBooks();
      this.fetchAuthors();
      this.fetchPublishers();
      this.fetchCategories();
    },
    methods: {
        showNotification(message, type) {
    this.notificationMessage = message;
    this.notificationType = type;
    this.showNotification = true;
    setTimeout(() => this.showNotification = false, 3000);
  },
  fetchBooks() {
      fetch(`http://localhost:5000/api/books?title=${this.titleQuery}&author=${this.authorQuery}&publisher=${this.publisherQuery}&category=${this.categoryQuery}&page=${this.currentPage}`)
        .then(response => response.json())
        .then(data => {
          this.books = data.books;
          this.totalPages = data.pages;
          this.currentPage = data.current_page;
        })
        .catch(error => {
          console.error("Error fetching books:", error);
        });
    },
    fetchAuthors() {
      fetch('http://localhost:5000/api/authors')
        .then(response => response.json())
        .then(data => {
          this.authors = data;
        })
        .catch(error => {
          console.error("Error fetching authors:", error);
        });
    },
    fetchPublishers() {
      fetch('http://localhost:5000/api/publishers')
        .then(response => response.json())
        .then(data => {
          this.publishers = data;
        })
        .catch(error => {
          console.error("Error fetching publishers:", error);
        });
    },
    fetchCategories() {
      fetch('http://localhost:5000/api/categories')
        .then(response => response.json())
        .then(data => {
          this.categories = data;
        })
        .catch(error => {
          console.error("Error fetching categories:", error);
        });
    },
      changePage(page) {
        this.currentPage = page;
        this.fetchBooks();
      },
      openModal(bookId) {
    this.selectedBookId = bookId;
    this.showModal = true;
    this.$nextTick(() => {
      document.querySelector('.modal').classList.add('show');
    });
  },
  closeModal() {
    const modal = document.querySelector('.modal');
    modal.classList.remove('show');
    setTimeout(() => this.showModal = false, 300); // match the CSS transition time
  },
  
  issueBook() {
    const user = JSON.parse(localStorage.getItem('user'));
  const userId = user ? user.userid : null;  // Safely handle cases where user might not be set

  if (!userId) {
    console.error("User ID is not available");
    this.setMessage('Please log in to issue a book', 'error');
    return;  // Exit the function if no user ID is found
  }
      fetch('http://localhost:5000/api/issue_book', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          book_id: this.selectedBookId,
          user_id: userId,
          days: this.borrowDays
        })
      })
      .then(response => response.json())
      .then(data => {
        console.log(data.message);
        this.setMessage('Book requested successfully', 'success');
        this.closeModal();
      })
      .catch(error => {
        console.error("Error issuing book:", error);
        this.setMessage('Something went wrong', 'error');
      });
    },
  setMessage(msg, type) {
    this.message = msg;
    this.messageType = type;
    this.$nextTick(() => {
      // Ensure the DOM has updated before adding the 'show' class
      document.querySelector('.message-widget').classList.add('show');
    });
    setTimeout(() => {
      this.clearMessages();
    }, 5000);
  },

  clearMessages() {
    const widget = document.querySelector('.message-widget');
    if (widget) {
      widget.classList.remove('show');
    }
    this.message = '';
    this.messageType = '';
  },
  

    },
    
  });
  </script>
  
  <style scoped>
  /* Add your CSS styles here */
  /* General Page Styles *//* General Page Styles */
.books-page {
  font-family: 'Segoe UI', 'Roboto', sans-serif;
  color: #2d2d2d;
  padding: 30px;
  background-color: #ffffff;
}

.books-page h1 {
  text-align: center;
  color: #2f54eb;
  font-size: 2.5em;
  margin-bottom: 30px;
}

/* Filters Section Styles */
.filters {
  display: flex;
  justify-content: space-around;
  margin-bottom: 40px;
  align-items: center;
}

.filters input,
.filters select {
  padding: 10px 15px;
  border: none;
  border-bottom: 2px solid #9e9e9e;
  background-color: transparent;
  font-size: 1em;
  color: #424242;
  transition: all 0.3s ease-in-out;
}

.filters input:focus,
.filters select:focus {
  border-bottom: 2px solid #2f54eb;
  outline: none;
}

/* Books List Styles */
.books-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 30px;
}

.book-item {
  background-color: #ffffff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
  transition: transform 0.3s ease, box-shadow 0.4s ease;
  overflow: hidden;
}

.book-item:hover {
  transform: translateY(-10px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.2);
}

.book-item h3 {
  color: #2f54eb;
  margin-top: 0;
  font-size: 1.4em;
}

.book-item p {
  margin: 10px 0;
  font-size: 0.9em;
  color: #666;
}

.book-item img {
  width: 100%;
  height: auto;
  border-radius: 8px;
  transition: transform 0.3s ease-in-out;
}

.book-item img:hover {
  transform: scale(1.05);
}

/* Pagination Styles */
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 40px;
  font-size: 1em;
}

.pagination button {
  padding: 10px 20px;
  margin: 0 15px;
  background-color: #2f54eb;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.pagination button:hover {
  background-color: #1d40c4;
}

.pagination button:disabled {
  background-color: #b0b0b0;
  cursor: not-allowed;
}

.pagination span {
  color: #424242;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  width: 300px;
  border-radius: 5px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  text-align: center;
  animation: modalEnter 0.5s ease forwards;
}

@keyframes modalEnter {
  from {
    transform: translateY(-50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.close {
  position: absolute;
  top: 10px;
  right: 15px;
  font-size: 20px;
  cursor: pointer;
}

.modal.show {
  opacity: 1;
  visibility: visible;
}
.borrow-button {
    background-color: #4CAF50;
  color: white;
  padding: 15px 25px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: transform 0.3s ease, background-color 0.3s ease;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  font-size: 16px;
  font-weight: 500;
  letter-spacing: 0.5px;
  }

.borrow-button:hover {
    background-color: #45a049;
  transform: translateY(-3px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.25);
  
}

.borrow-button:active {
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
.input-group {
  margin-bottom: 15px;
  position: relative;
  transition: transform 0.3s ease;
}

.input-group input:focus, .input-group select:focus {
  outline: none;
  border-color: #007bff;
}

.input-group input,
.input-group select {
width: 100%;
padding: 5px;
    margin-bottom: 15px;
    margin-top: 15px;
border: 1px solid #ddd;
border-radius: 4px;
box-sizing: border-box;
transition: border-color 0.3s ease;
}

.input-group input:focus, .input-group select:focus {
border-color: #007bff;
box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
}

.input-group label {
display: block;
margin-bottom: 5px;
color: #666;
transition: color 0.3s ease;
}

.input-group:hover label {
color: #007bff;
}

.message-widget {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%) translateY(50px); /* Center horizontally and start below the view */
  padding: 15px 25px;
  border-radius: 10px;
  color: white;
  text-align: center;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  transition: transform 0.5s ease, opacity 0.5s ease;
  opacity: 0;
  z-index: 1000; /* Ensure it's above other elements */
  background-color: #333; /* Default background color */
  width: auto;
  max-width: 80%; /* Max width */
  min-width: 300px; /* Minimum width */
  text-align: center;
  font-size: 16px;
}

.message-widget.show {
  opacity: 1;
  transform: translateX(-50%) translateY(0); /* Slide up into view */
}

.message-widget.success {
  background-color: #28a745; /* Green for success messages */
}

.message-widget.error {
  background-color: #dc3545; /* Red for error messages */
}

/* Optional: Add keyframe animation for a bounce effect */
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {transform: translateX(-50%) translateY(0);}
  40% {transform: translateX(-50%) translateY(-20px);}
  60% {transform: translateX(-50%) translateY(-10px);}
}

.message-widget.show {
  animation: bounce 1s ease;
}

  </style>
  