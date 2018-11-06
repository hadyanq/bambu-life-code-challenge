<template>
  <div class="main-wrapper">
    <price-chart 
      :ceiling="highestPrice"
      :floor="lowestPrice"
      :interval="filteredByMonth"
    />
  </div>
</template>

<script>
  import priceData from "./priceData.json"
  import PriceChart from "./components/PriceChart.vue"

  export default {
    components: { PriceChart },
    data() {
      return {
        priceData: priceData["Time Series (Daily)"],
        month: "09",
        stock: "MSFT"
      }
    },
    computed: {
      filteredByMonth() {
        let filtered = {}
        for (let d in this.priceData) {
          const removeDays = d.substring(0, 7)
          if (removeDays === `2018-${this.month}`) {
            filtered[d] = this.priceData[d]
          }
        }

        return filtered
      },
      highestPrice() {
        const days = Object.keys(this.filteredByMonth)
        let currentD  = days[0]
        let ceiling = { [currentD]: this.filteredByMonth[currentD] }

        for (let d in this.filteredByMonth) {
          if (
            Number(this.filteredByMonth[d]["2. high"]) > 
            Number(ceiling[currentD]["2. high"])
          ) {
            currentD = d
            ceiling = { [currentD]: this.filteredByMonth[currentD] }
          }
        }

        return ceiling
      },
      lowestPrice() {        
        const days = Object.keys(this.filteredByMonth)
        let currentD  = days[0]
        let floor = { [currentD]: this.filteredByMonth[currentD] }

        for (let d in this.filteredByMonth) {
          if (
            Number(this.filteredByMonth[d]["3. low"]) < 
            Number(floor[currentD]["3. low"])
          ) {
            currentD = d
            floor = { [currentD]: this.filteredByMonth[currentD] }
          }
        }

        return floor
      }
    }
  }
</script>

<style>
  .main-wrapper {
    width: 860px;
    margin: auto;
  }

  .chart-wrapper {
    background-color: lightsteelblue;
  }  
</style>