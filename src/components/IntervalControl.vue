<template>
  <div class="interval-wrap">
    <div class="interval-radio">
      <input type="radio" v-model="interval" name="interval" value="daily"/>Daily
      <input type="radio" v-model="interval" name="interval" value="monthly"/>Monthly
    </div>
    <div class="interval-select">
      <select v-if="interval === 'daily' && monthPicker.length" v-model="month">
        <option v-for="m in monthPicker" :key="m" :value="m">
          {{ displayMonth(m) }}
        </option>
      </select>
      <select v-else-if="interval === 'monthly' && yearPicker.length" v-model="year">
        <option v-for="y in yearPicker" :key="y" :value="y">
          {{ y }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
  import moment from "moment-mini"

  export default {
    props: {
      priceData: {
        type: Object,
        default: () => {}
      },
      selectedInterval: {
        type: String,
        default: ""
      },
      selectedMonth: {
        type: String,
        default: ""
      },
      selectedYear: {
        type: String,
        default: ""
      }
    },
    data() {
      return {
        interval: "",
        month: "",
        year: ""
      }
    },
    computed: {
      monthPicker() {
        let months = []
        for (let m in this.priceData) {
          const month = m.substring(5, 7)          
          if (!months.includes(month)) {
            months.push(month)
          } 
        }

        return months
      },
      yearPicker() {
        let years = []
        for (let y in this.priceData) {
          const year = y.substring(0, 4)          
          if (!years.includes(year)) {
            years.push(year)
          } 
        }

        return years
      }
    },
    watch: {
      month(m) {
        this.$emit("setMonth", m)
      },
      year(y) {
        this.$emit("setYear", y)
      },  
      interval(i) {
        this.$emit("setInterval", i)
      }      
    },
    mounted() {
      if (this.selectedInterval) this.interval = this.selectedInterval
      if (this.selectedMonth) this.month = this.selectedMonth
      if (this.selectedYear) this.year = this.selectedYear
    },
    methods: {
      displayMonth(m) {
        return moment(m, "MM").format("MMMM")
      }
    }
  }
</script>

<style scoped>
  .interval-wrap {
    display: inline-block;
  }

  .interval-radio {
    margin-left: 32px;
    display: inline-block;
  }

  .interval-select {
    margin-left: 32px;
    display: inline-block;
  }
</style>