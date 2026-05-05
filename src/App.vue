<template>
  <div class="container py-4">
    <h2 class="mb-4">Employee Management System</h2>

    <div class="card mb-4">
      <div class="card-header">
        {{ editMode ? 'Edit Employee' : 'Add Employee' }}
      </div>
      <div class="card-body">
        <div class="row g-2">
          <div class="col-md-6">
            <label class="form-label">Name</label>
            <input v-model="form.name" type="text" class="form-control" placeholder="Employee Name" />
          </div>
          <div class="col-md-6">
            <label class="form-label">Designation</label>
            <input v-model="form.designation" type="text" class="form-control" placeholder="Designation" />
          </div>
          <div class="col-md-6">
            <label class="form-label">Department</label>
            <input v-model="form.department" type="text" class="form-control" placeholder="Department" />
          </div>
          <div class="col-md-6">
            <label class="form-label">Salary</label>
            <input v-model="form.salary" type="number" class="form-control" placeholder="Salary" />
          </div>
        </div>
        <div class="mt-3">
          <button class="btn btn-primary me-2" @click="submitForm">
            {{ editMode ? 'Update' : 'Add Employee' }}
          </button>
          <button v-if="editMode" class="btn btn-secondary" @click="cancelEdit">Cancel</button>
        </div>
      </div>
    </div>

   
    <div class="card">
      <div class="card-header">Employee Records</div>
      <div class="card-body p-0">
        <div v-if="loading" class="text-center p-3">Loading...</div>
        <div v-else-if="employees.length === 0" class="text-center p-3 text-muted">No employees found.</div>
        <table v-else class="table table-bordered table-hover mb-0">
          <thead class="table-light">
            <tr>
              <th>#</th>
              <th>Name</th>
              <th>Designation</th>
              <th>Department</th>
              <th>Salary (₹)</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(emp, index) in employees" :key="emp.id">
              <td>{{ index + 1 }}</td>
              <td>{{ emp.name }}</td>
              <td>{{ emp.designation }}</td>
              <td>{{ emp.department }}</td>
              <td>{{ emp.salary }}</td>
              <td>
                <button class="btn btn-sm btn-warning me-1" @click="editEmployee(emp)">Edit</button>
                <button class="btn btn-sm btn-danger" @click="deleteEmployee(emp.id)">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

const API_URL = 'https://69fa0338c509a40d3aa3bb6a.mockapi.io/api/employees' 

export default {
  name: 'App',
  data() {
    return {
      employees: [],
      loading: false,
      editMode: false,
      editId: null,
      form: {
        name: '',
        designation: '',
        department: '',
        salary: ''
      }
    }
  },
  mounted() {
    this.fetchEmployees()
  },
  methods: {
    async fetchEmployees() {
      this.loading = true
      try {
        const res = await axios.get(API_URL)
        this.employees = res.data
      } catch (err) {
        alert('Failed to fetch employees.')
      } finally {
        this.loading = false
      }
    },

    async submitForm() {
      if (!this.form.name || !this.form.designation || !this.form.department || !this.form.salary) {
        alert('Please fill all fields.')
        return
      }
      try {
        if (this.editMode) {
          await axios.put(`${API_URL}/${this.editId}`, this.form)
        } else {
          await axios.post(API_URL, this.form)
        }
        this.resetForm()
        this.fetchEmployees()
      } catch (err) {
        alert('Operation failed.')
      }
    },

    editEmployee(emp) {
      this.editMode = true
      this.editId = emp.id
      this.form = { name: emp.name, designation: emp.designation, department: emp.department, salary: emp.salary }
    },

    async deleteEmployee(id) {
      if (!confirm('Delete this employee?')) return
      try {
        await axios.delete(`${API_URL}/${id}`)
        this.fetchEmployees()
      } catch (err) {
        alert('Delete failed.')
      }
    },

    cancelEdit() {
      this.resetForm()
    },

    resetForm() {
      this.editMode = false
      this.editId = null
      this.form = { name: '', designation: '', department: '', salary: '' }
    }
  }
}
</script>