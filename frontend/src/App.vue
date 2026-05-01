<template>
  <div id="app">
    <div class="container">
      <h1>📝 CRUD приложение на Vue 3</h1>
      
      <!-- Форма для создания/редактирования -->
      <div class="form-container">
        <h2>{{ editingItem ? 'Редактировать' : 'Добавить' }} задачу</h2>
        <form @submit.prevent="submitForm">
          <input 
            v-model="formData.title"
            type="text"
            placeholder="Заголовок"
            required
          />
          <textarea 
            v-model="formData.description"
            placeholder="Описание"
            required
          ></textarea>
          <button type="submit">
            {{ editingItem ? 'Обновить' : 'Создать' }}
          </button>
          <button v-if="editingItem" type="button" @click="cancelEdit" class="cancel">
            Отмена
          </button>
        </form>
      </div>

      <!-- Список задач -->
      <div class="items-list">
        <h2>Список задач</h2>
        <div v-if="loading" class="loading">Загрузка...</div>
        <div v-else-if="items.length === 0" class="empty">
          Нет задач. Создайте первую!
        </div>
        <div v-else class="items-grid">
          <div v-for="item in items" :key="item._id" class="item-card">
            <div class="item-header">
              <h3>{{ item.title }}</h3>
              <div class="status">
                <span :class="{ completed: item.completed }">
                  {{ item.completed ? '✓ Выполнено' : '○ Не выполнено' }}
                </span>
              </div>
            </div>
            <p class="description">{{ item.description }}</p>
            <div class="date">
              {{ formatDate(item.createdAt) }}
            </div>
            <div class="actions">
              <button @click="toggleStatus(item)" class="toggle">
                {{ item.completed ? 'Отменить' : 'Выполнить' }}
              </button>
              <button @click="editItem(item)" class="edit">
                Редактировать
              </button>
              <button @click="deleteItem(item._id)" class="delete">
                Удалить
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const API_URL = 'http://localhost:5000/api/items';

export default {
  name: 'App',
  data() {
    return {
      items: [],
      formData: {
        title: '',
        description: ''
      },
      editingItem: null,
      loading: false
    };
  },
  mounted() {
    this.fetchItems();
  },
  methods: {
    async fetchItems() {
      this.loading = true;
      try {
        const response = await axios.get(API_URL);
        this.items = response.data;
      } catch (error) {
        console.error('Ошибка загрузки:', error);
        alert('Ошибка загрузки задач');
      } finally {
        this.loading = false;
      }
    },

    async submitForm() {
      try {
        if (this.editingItem) {
          await axios.put(`${API_URL}/${this.editingItem._id}`, this.formData);
          alert('Задача обновлена');
        } else {
          await axios.post(API_URL, this.formData);
          alert('Задача создана');
        }
        this.resetForm();
        this.fetchItems();
      } catch (error) {
        console.error('Ошибка:', error);
        alert(error.response?.data?.message || 'Произошла ошибка');
      }
    },

    async toggleStatus(item) {
      try {
        await axios.put(`${API_URL}/${item._id}`, {
          ...item,
          completed: !item.completed
        });
        this.fetchItems();
      } catch (error) {
        console.error('Ошибка:', error);
        alert('Ошибка обновления статуса');
      }
    },

    editItem(item) {
      this.editingItem = item;
      this.formData.title = item.title;
      this.formData.description = item.description;
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },

    async deleteItem(id) {
      if (confirm('Вы уверены, что хотите удалить эту задачу?')) {
        try {
          await axios.delete(`${API_URL}/${id}`);
          alert('Задача удалена');
          this.fetchItems();
        } catch (error) {
          console.error('Ошибка:', error);
          alert('Ошибка удаления задачи');
        }
      }
    },

    cancelEdit() {
      this.resetForm();
    },

    resetForm() {
      this.formData = {
        title: '',
        description: ''
      };
      this.editingItem = null;
    },

    formatDate(date) {
      if (!date) return '';
      return new Date(date).toLocaleString('ru-RU');
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

h1 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 30px;
}

.form-container {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
}

.form-container h2 {
  margin-top: 0;
  color: #34495e;
}

input, textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

textarea {
  min-height: 80px;
  resize: vertical;
}

button {
  background: #3498db;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
  font-size: 14px;
}

button:hover {
  background: #2980b9;
}

button.cancel {
  background: #95a5a6;
}

button.cancel:hover {
  background: #7f8c8d;
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 20px;
}

.item-card {
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.item-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.item-header h3 {
  margin: 0;
  color: #2c3e50;
}

.status span {
  font-size: 12px;
  padding: 3px 8px;
  border-radius: 12px;
  background: #ecf0f1;
}

.status .completed {
  background: #2ecc71;
  color: white;
}

.description {
  color: #7f8c8d;
  margin: 10px 0;
  line-height: 1.4;
}

.date {
  font-size: 12px;
  color: #95a5a6;
  margin-bottom: 15px;
}

.actions {
  display: flex;
  gap: 10px;
}

.actions button {
  flex: 1;
  padding: 8px;
  font-size: 12px;
}

button.toggle {
  background: #2ecc71;
}

button.toggle:hover {
  background: #27ae60;
}

button.edit {
  background: #f39c12;
}

button.edit:hover {
  background: #e67e22;
}

button.delete {
  background: #e74c3c;
}

button.delete:hover {
  background: #c0392b;
}

.loading, .empty {
  text-align: center;
  padding: 40px;
  color: #7f8c8d;
  font-size: 18px;
}
</style>