<template>
  <div class="interval-wrap">
    <div class="interval-radio">
      <input type="radio" v-model="interval" name="interval" value="daily"/>Daily
    </div>
    <div class="interval-select">
      <select v-if="interval === 'daily'" v-model="month">
        <option v-for="m in monthPicker" :key="m" :value="m">
          {{ displayMonth(m) }}
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
      interval: {
        type: String,
        default: ""
      },
      selectedMonth: {
        type: String,
        default: ""
      }
    },
    data() {
      return {
        month: ""
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
    },
    watch: {
      month(m) {
        this.$emit("setMonth", m)
      }
    },
    mounted() {
      if (this.selectedMonth) this.month = this.selectedMonth
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