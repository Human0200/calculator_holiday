<!-- src/components/VacationCalculator.vue -->
<template>
  <div class="app-container">
    <AppSidebar
      @toggle-collapse="onSidebarToggle"
      @item-click="onMenuItemClick"
      @go-home="goToHomePage"
    />

    <div class="content" :class="{ 'content-collapsed': isSidebarCollapsed }">
      <div class="container mt-3">
        <h2 class="mb-4">Калькулятор отпуска</h2>

        <form @submit.prevent="calculate">
          <div class="row mb-3">
            <div class="col-md-6">
              <label class="form-label">Дата приёма на работу:</label>
              <input type="date" v-model="startDate" class="form-control" required />
            </div>
            <div class="col-md-6">
              <label class="form-label">Расчётная дата:</label>
              <input type="date" v-model="endDate" class="form-control" required />
            </div>
          </div>

          <div class="row mb-3">
            <div class="col-md-6">
              <label class="form-label">Продолжительность ежегодного отпуска (дней):</label>
              <input type="number" v-model.number="vacationDaysPerYear" class="form-control" min="0" />
            </div>
            <div class="col-md-6">
              <label class="form-label">Использовано дней:</label>
              <input type="number" v-model.number="usedDays" class="form-control" min="0" />
            </div>
          </div>

          <div class="mb-4">
            <h5>Периоды отсутствия без уважительной причины</h5>
            <div v-for="(period, index) in unexcusedAbsences" :key="'unexcused-'+index" class="row mb-2 align-items-center">
              <div class="col-md-5 mb-2 mb-md-0">
                <input type="date" v-model="period.start" class="form-control" />
              </div>
              <div class="col-md-5 mb-2 mb-md-0">
                <input type="date" v-model="period.end" class="form-control" />
              </div>
              <div class="col-md-2">
                <button type="button" @click="removeUnexcused(index)" class="btn btn-danger btn-sm w-100">Удалить</button>
              </div>
            </div>
            <button type="button" @click="addUnexcused" class="btn btn-secondary">+ Добавить период</button>
          </div>

          <div class="mb-4">
            <h5>Отпуск по уходу за ребенком</h5>
            <div v-for="(period, index) in childcareLeaves" :key="'childcare-'+index" class="row mb-2 align-items-center">
              <div class="col-md-5 mb-2 mb-md-0">
                <input type="date" v-model="period.start" class="form-control" />
              </div>
              <div class="col-md-5 mb-2 mb-md-0">
                <input type="date" v-model="period.end" class="form-control" />
              </div>
              <div class="col-md-2">
                <button type="button" @click="removeChildcare(index)" class="btn btn-danger btn-sm w-100">Удалить</button>
              </div>
            </div>
            <button type="button" @click="addChildcare" class="btn btn-secondary">+ Добавить период</button>
          </div>

          <div class="d-grid gap-2 d-md-flex justify-content-md-end">
            <button type="submit" class="btn btn-primary me-md-2">Рассчитать</button>
          </div>
        </form>

        <div v-if="result" class="mt-4 p-3 bg-light border rounded">
          <p><strong>Вы проработали:</strong> {{ workedDays }} дней ({{ monthsWorked }} полных месяцев)</p>
          <p><strong>Положено дней отпуска:</strong> {{ totalVacationDays }}</p>
          <p><strong>Осталось дней:</strong> {{ result }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import AppSidebar from '@/components/AppSidebar.vue'

export default {
  name: 'VacationCalculator',
  components: {
    AppSidebar
  },
  data() {
    return {
      isSidebarCollapsed: false,
      startDate: '',
      endDate: new Date().toISOString().slice(0, 10),
      vacationDaysPerYear: 28,
      usedDays: 0,
      unexcusedAbsences: [],
      childcareLeaves: [],
      result: null,
      monthsWorked: 0,
      workedDays: 0,
      totalVacationDays: 0
    }
  },
  methods: {
    onSidebarToggle(collapsed) {
      this.isSidebarCollapsed = collapsed
    },
    onMenuItemClick(event, item) {
      console.log('Menu item clicked', item)
    },
    goToHomePage() {
      this.$router.push('/')
    },
    addUnexcused() {
      this.unexcusedAbsences.push({ start: '', end: '' })
    },
    removeUnexcused(index) {
      this.unexcusedAbsences.splice(index, 1)
    },
    addChildcare() {
      this.childcareLeaves.push({ start: '', end: '' })
    },
    removeChildcare(index) {
      this.childcareLeaves.splice(index, 1)
    },
    calculateDays(start, end) {
      return Math.floor((end - start) / (1000 * 60 * 60 * 24)) + 1
    },
    subtractPeriods(totalDays, periods) {
      let daysToSubtract = 0
      for (const period of periods) {
        if (!period.start || !period.end) continue
        const start = new Date(period.start)
        const end = new Date(period.end)
        if (start > end) continue
        daysToSubtract += this.calculateDays(start, end)
      }
      return Math.max(totalDays - daysToSubtract, 0)
    },
    calculate() {
      if (!this.startDate || !this.endDate) {
        alert('Введите все даты')
        return
      }
      const start = new Date(this.startDate)
      const end = new Date(this.endDate)
      if (start > end) {
        alert('Дата начала работы не может быть позже расчётной даты')
        return
      }
      let totalDays = this.calculateDays(start, end)
      totalDays = this.subtractPeriods(totalDays, this.unexcusedAbsences)
      totalDays = this.subtractPeriods(totalDays, this.childcareLeaves)
      this.monthsWorked = Math.floor(totalDays / 30)
      this.workedDays = totalDays
      this.totalVacationDays = (this.monthsWorked * (this.vacationDaysPerYear / 12)).toFixed(2)
      if (totalDays >= 15 && this.monthsWorked === 0) {
        this.totalVacationDays = (this.vacationDaysPerYear / 12).toFixed(2)
      }
      this.result = Math.max(this.totalVacationDays - this.usedDays, 0).toFixed(2)
    }
  }
}
</script>

<style scoped>
.app-container {
  display: flex;
  min-height: 100vh;
}

.content {
  flex-grow: 1;
  padding: 20px;
  transition: margin-left 0.3s;
  margin-left: 290px;
}

.content-collapsed {
  margin-left: 50px;
}

@media (max-width: 768px) {
  .content {
    margin-left: 50px;
  }
}

.form-label {
  font-weight: 500;
}

.btn-secondary {
  background-color: #6c757d;
  border-color: #6c757d;
}

.btn-primary {
  background-color: #0d6efd;
  border-color: #0d6efd;
}

.bg-light {
  background-color: #f8f9fa !important;
}
</style>