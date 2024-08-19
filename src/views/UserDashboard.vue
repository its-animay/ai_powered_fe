<template>
  <div>
      <NavBar/>
  <div class="container py-5">
    <div class="user-details mb-4">
      <h2>User Dashboard</h2>
      <div class="card">
      <div class="card-body">
        <h5 class="card-title">{{ userDetails.username }}</h5> <!-- Changed from name to username -->
        <p class="card-text"><strong>Email:</strong> {{ userDetails.email }}</p>
        <p class="card-text"><strong>Role:</strong> {{ userDetails.role }}</p>
        <p class="card-text"><strong>Membership Type:</strong> {{ userDetails.membership_type }}</p>
        <p class="card-text"><strong>Membership Start:</strong> {{ userDetails.membership_start }}</p>
        <p class="card-text" v-if="userDetails.membership_end"><strong>Membership End:</strong> {{ userDetails.membership_end }}</p>
        <p class="card-text" v-else><strong>Membership End:</strong> N/A</p>
      </div>
    </div>
  </div>
    
    <div class="issued-books">
      <h3>Issued Books</h3>
      <div class="row">
        <div class="col-md-4" v-for="book in issuedBooks" :key="book.book_id">
          <div class="card mb-4 box-shadow">
            <img class="card-img-top" :src="book.cover_path" alt="Book cover">
            <div class="card-body">
              <h5 class="card-title">{{ book.title }}</h5>
              <p class="card-text">{{ book.author }}</p>
              <div class="d-flex justify-content-between align-items-center">
                <small class="text-muted">Due: {{ formatDate(book.due_date) }}</small>
                <button @click="openPdfViewer(book.source)" class="btn btn-sm btn-outline-secondary">Read</button>
                <button @click="showRatingForm(book.book_id)" class="btn btn-primary btn-sm">Rate</button>
          </div>
          <div v-show="book.showRatingForm">
              <input type="number" v-model="rating" placeholder="Rating (1-5)" />
              <input type="text" v-model="comment" placeholder="Comment" />
              <button @click="submitReview(book.book_id)" class="btn btn-primary btn-sm">Submit</button>
              <button @click="showRatingForm(book.book_id)" class="btn btn-secondary btn-sm">Close</button>
          </div>
              </div>
            </div>
          </div>
          
        </div>
      </div>
      <div v-if="showPdfViewer" class="pdf-viewer-modal">
    <div class="pdf-viewer-container">
      <iframe :src="pdfViewerUrl" frameborder="0" style="width: 100%; height: 100%;"></iframe>
      <button @click="closePdfViewer" class="btn btn-secondary">Close</button>
    </div>
  </div>
    </div>
  </div>

</template>

<script>
import { onMounted, ref } from 'vue';
import { defineComponent } from 'vue';
import NavBar from '../components/NavBar.vue';



export default defineComponent({
  components: {
  NavBar,
},
  name: 'UserDashboard',
  setup() {
    const issuedBooks = ref([]);
    const userDetails = ref({}); 
    const rating = ref(null);
  const comment = ref('');
  const showPdfViewer = ref(false);
    const pdfViewerUrl = ref('');
  
  const showRatingForm = (bookId) => {
  const book = issuedBooks.value.find(b => b.book_id === bookId);
  if (book) book.showRatingForm = !book.showRatingForm;
};
    const fetchIssuedBooks = async (userId) => {
      try {
        const response = await fetch(`http://localhost:5000/api/user/${userId}/issued_books`, {
          method: 'GET',
        });
        if (!response.ok) throw new Error('Failed to fetch issued books');
        const data = await response.json();
        issuedBooks.value = data.issued_books;
      } catch (error) {
        console.error('Error fetching issued books:', error);
      }
    };
    
    const fetchUserDetails = async (userId) => {
      try {
        const response = await fetch(`http://localhost:5000/api/users/${userId}`, {
          method: 'GET',
        });
        if (!response.ok) throw new Error('Failed to fetch user details');
        const data = await response.json();
        userDetails.value = {
          ...data,
          membership_start: formatDate(data.membership_start),
          membership_end: data.membership_end ? formatDate(data.membership_end) : 'N/A'
        };
      } catch (error) {
        console.error('Error fetching user details:', error);
      }
    };
    const getUserIdFromLocalStorage = () => {
    const user = JSON.parse(localStorage.getItem('user'));
    if (user && user.userid) {
        return user.userid;
    }
    // Return null if the user ID does not exist or is invalid
    return null;
};

    const formatDate = (dateString) => {
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return new Date(dateString).toLocaleDateString(undefined, options);
    };
    const submitReview = async (bookId) => {
      const user = JSON.parse(localStorage.getItem('user'));
  const userId = user ? user.userid : null;  
  // Placeholder user ID, replace with actual logic to retrieve user ID
try {
  const response = await fetch('http://localhost:5000/api/reviews', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      book_id: bookId,
      user_id: getUserIdFromLocalStorage(),
      rating: rating.value,
      comment: comment.value,
    }),
  });
  if (!response.ok) throw new Error('Failed to submit review');
  // Clear form fields after successful submission
  rating.value = null;
  comment.value = '';
  // Refresh issued books list after review submission
  fetchIssuedBooks(userId);
} catch (error) {
  console.error('Error submitting review:', error);
}
const book = issuedBooks.value.find(b => b.book_id === bookId);
  if (book) book.showRatingForm = false;
};
const openPdfViewer = (pdfUrl) => {
    // Debugging: Log the pdfUrl argument to check its value
    console.log('Opening PDF Viewer with URL:', pdfUrl);
    
    // Check if pdfUrl is valid
    if (pdfUrl && typeof pdfUrl === 'string') {
        // Set the PDF viewer URL with parameters to remove download and print options
        pdfViewerUrl.value = `${pdfUrl}#toolbar=0&navpanes=0`;
        showPdfViewer.value = true;
    } else {
        console.error('Invalid PDF URL:', pdfUrl);
        showPdfViewer.value = false;
    }
};

    const closePdfViewer = () => {
      showPdfViewer.value = false;
    };




    onMounted(() => {
      
  const userId = getUserIdFromLocalStorage()  
      fetchIssuedBooks(userId);
      issuedBooks.value.forEach(book => book.showRatingForm = false);
      fetchUserDetails(userId); 
    });

    return {
      issuedBooks,
      userDetails,
      formatDate,
      rating,
    comment,
    showRatingForm,
    submitReview,
    showPdfViewer,
      pdfViewerUrl,
      openPdfViewer,
      closePdfViewer,
    };
  },
});
</script>
  
  
  <style scoped>
  /* Styles for User Dashboard */

/* Container styles */
.container {
  max-width: 960px;
  margin: 0 auto;
}

/* User details section */
.user-details h2 {
  margin-bottom: 20px;
  color: #333;
}

.card {
  border: none;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.card-body {
  padding: 20px;
}

.card-title {
  font-size: 24px;
  margin-bottom: 10px;
}

.card-text {
  color: #666;
}

/* Issued books section */
.issued-books h3 {
  margin-top: 40px;
  margin-bottom: 20px;
  color: #333;
}

.card-img-top {
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
  max-height: 200px;
  object-fit: cover;
}

.card-body {
  padding: 20px;
}

.card-title {
  font-size: 20px;
  margin-bottom: 10px;
}

.card-text {
  color: #666;
}

.btn-outline-secondary {
  color: #333;
  border-color: #333;
}

.btn-outline-secondary:hover {
  background-color: #333;
  color: #fff;
}

/* Animation on hover */
.card:hover {
  transform: translateY(-5px);
  transition: all 0.3s ease;
}

.btn-outline-secondary:hover {
  transform: translateY(-2px);
  transition: all 0.3s ease;
}
.pdf-viewer-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.pdf-viewer-container {
  background-color: #fff;
  width: 90%;
  height: 90%;
  position: relative;
}

  </style>
  