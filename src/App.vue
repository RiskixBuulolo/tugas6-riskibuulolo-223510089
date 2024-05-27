<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <h2>Buat Artikel Baru</h2>
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">Save</button>
    </form>
    <ul class="article-list">
      <h2>List Article </h2>
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div v-if="editId === article.id">
          <h2>Edit Article</h2>
          <input type="text" v-model="editForm.title" class="input" /><br />
          <textarea v-model="editForm.content" class="textarea"></textarea><br />
          <button @click="updateArticle(article.id)" class="button save-button">Update</button>
          <button @click="cancelEdit" class="button cancel-button">Cancel</button>
        </div>
        <div v-else>
          <strong>{{ article.title }}</strong><br />
          <p>{{ article.content }}</p>
          <button @click="edit(article)" class="button edit-button">Edit</button>
          <button @click="deleteArticle(article.id)" class="button delete-button">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load Articles</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });
    const editForm = reactive({
      id: null,
      title: '',
      content: ''
    });
    const articles = ref([]);
    const editId = ref(null);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, { title: form.title, content: form.content });

        if (form.id) {
          const index = articles.value.findIndex(article => article.id === form.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function updateArticle(id) {
      try {
        const response = await axios.put(`http://localhost:3000/articles/${id}`, {
          title: editForm.title,
          content: editForm.content
        });

        const index = articles.value.findIndex(article => article.id === id);
        if (index !== -1) {
          articles.value[index] = response.data;
        }

        cancelEdit();
      } catch (error) {
        console.error('Error updating article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      editId.value = article.id;
      editForm.id = article.id;
      editForm.title = article.title;
      editForm.content = article.content;
    }

    function cancelEdit() {
      editId.value = null;
      editForm.id = null;
      editForm.title = '';
      editForm.content = '';
    }

    onMounted(load);

    return { form, editForm, articles, save, edit, updateArticle, deleteArticle, load, cancelEdit, editId };
  }
}
</script>

<style scoped>
/* General Container Styling */
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}



/* Form Styling */
.form {
  background-color: #f9f9f957;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 30px;
}

.form h2 {
  margin-bottom: 20px;
  color: whitesmoke;
  text-align: center;
  font-size: 24px;
}

.input, .textarea {
  width: 100%;
  padding: 15px;
  margin-bottom: 20px;
  background-color: #f9f9f957;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.input:focus, .textarea:focus {
  border-color: #4caf50;
  outline: none;
}

.textarea {
  height: 120px;
  resize: vertical;
  
}

/* Button Styling */
.button {
  display: inline-block;
  padding: 12px 24px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.save-button {
  background-color: #4caf50;
  color: white;
}

.save-button:hover {
  background-color: #45a049;
}

.edit-button {
  background-color: #2196f3;
  color: white;
  margin-right: 10px;
}

.edit-button:hover {
  background-color: #1e87d3;
}

.cancel-button {
  background-color: #9e9e9e;
  color: white;
}

.cancel-button:hover {
  background-color: #8b8b8b;
}

.delete-button {
  background-color: #f44336;
  color: white;
}

.delete-button:hover {
  background-color: #e4352b;
}

.load-button {
  background-color: #ff9800;
  color: white;
  margin-top: 20px;
}

.load-button:hover {
  background-color: #e68a00;
}

/* Article List Styling */
.article-list {
  list-style-type: none;
  padding: 0;
}

.article-list h2 {
  color: whitesmoke;
  text-align: center;
  font-size: 24px;
}

.article-item {
  background-color: #f9f9f957;
  border: 1px solid #ddd;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.article-item strong {
  font-size: 20px;
  color: whitesmoke;
}

.article-item p {
  margin: 15px 0;
  color: whitesmoke;
}

.article-item .button {
  margin-top: 15px;
}

/* Media Queries for Responsive Design */
@media (max-width: 600px) {
  .form {
    padding: 20px;
  }

  .article-item {
    padding: 15px;
  }
}
</style>
