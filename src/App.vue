<template>
  <div class="container py-5" style="background-color: #f5f0e8; min-height: 100vh;">
    <div class="text-center mb-5">
      <h1 class="display-6 fw-bold">Employee Management System</h1>
    </div>

    <div class="row g-4">
      <div class="col-lg-4">
        <div class="card shadow-sm border-0 h-100">
          <div class="card-body">
            <h5 class="card-title mb-3">{{ editMode ? 'Edit Employee' : 'Add Employee' }}</h5>
            <div class="row g-3">
              <div class="col-12">
                <label class="form-label">Name</label>
                <input v-model="form.name" type="text" class="form-control" placeholder="Employee name" />
              </div>
              <div class="col-12">
                <label class="form-label">Designation</label>
                <input v-model="form.designation" type="text" class="form-control" placeholder="Designation" />
              </div>
              <div class="col-12">
                <label class="form-label">Department</label>
                <input v-model="form.department" type="text" class="form-control" placeholder="Department" />
              </div>
              <div class="col-12">
                <label class="form-label">Salary</label>
                <input v-model="form.salary" type="number" class="form-control" placeholder="Salary" />
              </div>
            </div>

            <div class="mt-4 d-flex flex-wrap gap-2">
              <button class="btn btn-primary" @click="submitForm">
                {{ editMode ? 'Update Employee' : 'Add Employee' }}
              </button>
              <button v-if="editMode" class="btn btn-outline-secondary" @click="cancelEdit">Cancel</button>
            </div>
          </div>
        </div>

        <div class="row g-3 mt-3">
          <div class="col-sm-6">
            <div class="card bg-light border-0 shadow-sm p-3 text-center">
              <div class="text-muted small">Total employees</div>
              <div class="h2 mb-0">{{ totalEmployees }}</div>
            </div>
          </div>
          <div class="col-sm-6">
            <div class="card bg-light border-0 shadow-sm p-3 text-center">
              <div class="text-muted small">Average salary</div>
              <div class="h2 mb-0">₹ {{ averageSalary }}</div>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-8">
        <div class="card shadow-sm border-0 h-100">
          <div class="card-body">
            <div class="d-flex flex-column flex-md-row align-items-start align-items-md-center justify-content-between mb-4 gap-2">
              <div>
                <h5 class="card-title mb-1">Employee Records</h5>
                <p class="text-muted mb-0">Manage all employee entries from one easy dashboard.</p>
              </div>
              <span class="badge bg-primary fs-6">{{ totalEmployees }} records</span>
            </div>

            <div class="table-responsive">
              <div v-if="loading" class="text-center py-5">
                <div class="spinner-border text-primary" role="status">
                  <span class="visually-hidden">Loading...</span>
                </div>
              </div>
              <div v-else-if="employees.length === 0" class="alert alert-info mb-0">
                No employees found. Add a new employee to get started.
              </div>
              <table v-else class="table table-hover align-middle mb-0">
                <thead class="table-primary">
                  <tr>
                    <th class="text-secondary">#</th>
                    <th class="text-secondary">Name</th>
                    <th class="text-secondary">Designation</th>
                    <th class="text-secondary">Department</th>
                    <th class="text-secondary">Salary (₹)</th>
                    <th class="text-secondary">Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(emp, index) in employees" :key="emp.id">
                    <td>{{ index + 1 }}</td>
                    <td>{{ emp.name }}</td>
                    <td>{{ emp.designation }}</td>
                    <td>{{ emp.department }}</td>
                    <td>₹ {{ emp.salary }}</td>
                    <td>
                      <button class="btn btn-sm btn-outline-warning me-1" @click="editEmployee(emp)">Edit</button>
                      <button class="btn btn-sm btn-outline-danger" @click="deleteEmployee(emp.id)">Delete</button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
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
  computed: {
    totalEmployees() {
      return this.employees.length
    },
    averageSalary() {
      if (!this.employees.length) return 0
      const total = this.employees.reduce((sum, emp) => sum + Number(emp.salary), 0)
      return Math.round(total / this.employees.length)
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
