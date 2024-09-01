<template>
  <div class="container">
    <h1>Posts</h1>

    <!-- Search Bar -->
    <div class="search-bar">
      <input v-model="searchQuery" type="text" placeholder="Search..." />
      <button @click="searchPosts">Search</button>
    </div>

    <div v-if="loading" class="loading">Loading...</div>
    <div v-else>
      <table>
        <thead>
          <tr>
            <th @click="sortPosts('id')">ID</th>
            <th>Title</th>
            <th>Body</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="post in paginatedPosts" :key="post.id">
            <td>{{ post.id }}</td>
            <td>{{ post.title }}</td>
            <td>{{ post.body }}</td>
            <td class="actions">
              <button class="edit-btn" @click="editPost(post)">Edit</button>
              <button class="delete-btn" @click="deletePost(post.id)">
                Delete
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      <div class="btn">
        <button @click="prevPage" :disabled="currentPage === 1">
          Previous
        </button>
        <button @click="nextPage" :disabled="currentPage === totalPages">
          Next
        </button>
      </div>
      <button class="add-post-btn" @click="openModal">Add New Post</button>
    </div>

    <!-- Modal Window -->
    <div v-if="isModalOpen" class="modal">
      <div class="modal-content">
        <h2>{{ isEditMode ? "Edit Post" : "Create New Post" }}</h2>
        <form @submit.prevent="isEditMode ? updatePost() : createPost()">
          <div class="form-group">
            <label for="title">Title:</label>
            <input
              v-model="currentPost.title"
              type="text"
              id="title"
              required
            />
          </div>
          <div class="form-group">
            <label for="body">Body:</label>
            <textarea v-model="currentPost.body" id="body" required></textarea>
          </div>
          <div class="modal-buttons">
            <button type="submit" class="save-btn">
              {{ isEditMode ? "Update" : "Save" }}
            </button>
            <button type="button" class="cancel-btn" @click="closeModal">
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";

const posts = ref([]);
const loading = ref(false);
const currentPage = ref(1);
const postsPerPage = 10;
const sortOrder = ref("asc");
const isModalOpen = ref(false);
const isEditMode = ref(false);
const searchQuery = ref(""); // متغير البحث
const currentPost = ref({
  id: null,
  title: "",
  body: "",
});

const fetchPosts = async () => {
  loading.value = true;
  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/posts");
    const data = await response.json();
    posts.value = data;
  } catch (error) {
    console.error("Error fetching posts:", error);
  } finally {
    loading.value = false;
  }
};

onMounted(fetchPosts);

const filteredPosts = computed(() => {
  return posts.value.filter(
    (post) =>
      post.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      post.body.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * postsPerPage;
  const end = start + postsPerPage;
  return filteredPosts.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.ceil(filteredPosts.value.length / postsPerPage);
});

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
};

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};

const sortPosts = (key) => {
  posts.value.sort((a, b) => {
    if (sortOrder.value === "asc") {
      return a[key] > b[key] ? 1 : -1;
    } else {
      return a[key] < b[key] ? 1 : -1;
    }
  });
  sortOrder.value = sortOrder.value === "asc" ? "desc" : "asc";
};

const openModal = () => {
  isEditMode.value = false;
  currentPost.value = { id: null, title: "", body: "" };
  isModalOpen.value = true;
};

const closeModal = () => {
  isModalOpen.value = false;
  currentPost.value = { id: null, title: "", body: "" };
};

const createPost = async () => {
  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        title: currentPost.value.title,
        body: currentPost.value.body,
        userId: 1,
      }),
    });
    const createdPost = await response.json();

    createdPost.id = posts.value.length + 1;
    posts.value.push(createdPost);
    closeModal();
  } catch (error) {
    console.error("Error creating post:", error);
  }
};

const deletePost = async (id) => {
  if (!confirm("Are you sure you want to delete this post?")) {
    return;
  }
  try {
    await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
      method: "DELETE",
    });
    posts.value = posts.value.filter((post) => post.id !== id);
  } catch (error) {
    console.error("Error deleting post:", error);
  }
};

const editPost = (post) => {
  isEditMode.value = true;
  currentPost.value = { ...post };
  isModalOpen.value = true;
};

const updatePost = async () => {
  try {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/posts/${currentPost.value.id}`,
      {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(currentPost.value),
      }
    );
    const updatedPost = await response.json();
    const index = posts.value.findIndex((post) => post.id === updatedPost.id);
    if (index !== -1) {
      posts.value[index] = updatedPost;
    }
    closeModal();
  } catch (error) {
    console.error("Error updating post:", error);
  }
};

const searchPosts = () => {
  currentPage.value = 1;
};
</script>

<style scoped>
/* Container Style */
.container {
  max-width: 900px;
  margin: 50px auto;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  font-family: "Arial", sans-serif;
}

/* Heading Style */
h1 {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 20px;
  color: #333;
  font-weight: bold;
}

/* Search Bar Style */
.search-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.search-bar input {
  padding: 10px;
  font-size: 1rem;
  border-radius: 5px 0 0 5px;
  border: 1px solid #ccc;
  width: 250px;
}

.search-bar input:focus {
  border-color: #007bff;
}

.search-bar button {
  padding: 10px 20px;
  font-size: 1rem;
  border: none;
  border-radius: 0 5px 5px 0;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

.search-bar button:hover {
  background-color: #0056b3;
}

/* Loading Style */
.loading {
  text-align: center;
  font-size: 1.5rem;
  color: #777;
}

/* Table Style */
table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

th,
td {
  border: 1px solid #ddd;
  padding: 15px;
  text-align: left;
  font-size: 1rem;
}

th {
  background-color: #007bff;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

th:hover {
  background-color: #0056b3;
}

td {
  background-color: #f9f9f9;
}

/* Actions Button Style */
.edit-btn,
.delete-btn {
  padding: 6px 12px;
  border: none;
  border-radius: 5px;
  color: white;
  font-size: 1rem;
  cursor: pointer;
  margin-right: 10px;
}

.edit-btn {
  background-color: #ffc107;
}

.edit-btn:hover {
  background-color: #e0a800;
}

.delete-btn {
  background-color: #dc3545;
}

.delete-btn:hover {
  background-color: #c82333;
}

/* Pagination Button Style */
.btn {
  text-align: center;
  margin-bottom: 20px;
}

.btn button {
  padding: 10px 20px;
  margin: 0 5px;
  border: none;
  border-radius: 5px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

.btn button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.add-post-btn {
  padding: 10px 20px;
  font-size: 1rem;
  border: none;
  border-radius: 5px;
  background-color: #28a745;
  color: white;
  cursor: pointer;
  display: block;
  margin: 0 auto;
}

.add-post-btn:hover {
  background-color: #218838;
}

/* Modal Style */
.modal {
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

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 100%;
  max-width: 500px;
}

.modal-content h2 {
  margin-top: 0;
  font-size: 1.5rem;
  text-align: center;
  color: #333;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-size: 1rem;
  color: #555;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  font-size: 1rem;
}

.modal-buttons {
  display: flex;
  justify-content: space-between;
}

.save-btn {
  padding: 10px 20px;
  font-size: 1rem;
  border: none;
  border-radius: 5px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

.save-btn:hover {
  background-color: #0056b3;
}

.cancel-btn {
  padding: 10px 20px;
  font-size: 1rem;
  border: none;
  border-radius: 5px;
  background-color: #dc3545;
  color: white;
  cursor: pointer;
}

.cancel-btn:hover {
  background-color: #c82333;
}
.actions {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 71px;
}
</style>
