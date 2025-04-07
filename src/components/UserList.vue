<template>
  <div class="container">
    <h1>User List</h1>

    <div v-if="loading" class="loader">Loading users...</div>
    <div v-else-if="error" class="error">{{ error }}</div>

    <div v-else>
      <table>
        <thead>
          <tr>
            <th @click="sortBy('id')">
              ID
              <span v-if="orderBy === 'id'">
                {{ order === 'ASC' ? '↑' : '↓' }}
              </span>
            </th>
            <th @click="sortBy('firstName')">
              Name
              <span v-if="orderBy === 'firstName'">
                {{ order === 'ASC' ? '↑' : '↓' }}
              </span>
            </th>
            <th @click="sortBy('email')">
              Email
              <span v-if="orderBy === 'email'">
                {{ order === 'ASC' ? '↑' : '↓' }}
              </span>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in users" :key="user.id">
            <td>{{ user.id }}</td>
            <td>{{ user.firstName }} {{ user.lastName }}</td>
            <td>{{ user.email }}</td>
          </tr>
        </tbody>
      </table>

      <div class="pagination">
        <button @click="prevPage" :disabled="page === 1">Previous</button>
        <span>Page {{ page }} of {{ totalPages }}</span>
        <button @click="nextPage" :disabled="page === totalPages">Next</button>
      </div>
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'

const users = ref([])
const page = ref(1)
const resultPerPage = 10
const total = ref(0)
const totalPages = ref(1)
const loading = ref(true)
const error = ref(null)

const orderBy = ref('firstName')
const order = ref('ASC')

const fetchUsers = async () => {
  loading.value = true
  error.value = null
  try {
    const res = await axios.get('http://localhost:5000/user/list', {
      params: {
        page: page.value,
        resultPerPage,
        order: order.value,
        orderBy: orderBy.value
      }
    })

    users.value = res.data.data
    total.value = res.data.total
    totalPages.value = Math.ceil(total.value / resultPerPage)
  } catch (err) {
    error.value = 'Failed to fetch users. Please try again later.'
    console.error(err)
  } finally {
    loading.value = false
  }
}

const sortBy = (field) => {
  if (orderBy.value === field) {
    order.value = order.value === 'ASC' ? 'DESC' : 'ASC'
  } else {
    orderBy.value = field
    order.value = 'ASC'
  }
  fetchUsers()
}

const nextPage = () => {
  if (page.value < totalPages.value) {
    page.value += 1
  }
}
const prevPage = () => {
  if (page.value > 1) {
    page.value -= 1
  }
}

onMounted(fetchUsers)
watch([page], fetchUsers)
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 40px auto;
  padding: 20px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

h1 {
  margin-bottom: 24px;
  font-size: 28px;
  color: #333;
  text-align: center;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 16px;
  cursor: pointer;
}

th,
td {
  text-align: left;
  padding: 12px 16px;
  border-bottom: 1px solid #ddd;
  color: #444;
}

th {
  background-color: #f5f5f5;
  color: #444;
  user-select: none;
}

tr:hover {
  background-color: #f0f8ff;
}

.loader,
.error {
  text-align: center;
  font-size: 18px;
  padding: 20px;
  color: #666;
}

.error {
  color: #e74c3c;
}

.pagination {
  margin-top: 20px;
  text-align: center;
}

.pagination button {
  padding: 10px 20px;
  margin: 0 10px;
  font-size: 16px;
  background: #007bff;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s;
}

.pagination button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.pagination span {
  font-size: 18px;
  color: #444;
}
</style>
