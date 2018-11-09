<template>
  <div class="main-wrapper">
    <price-chart 
      :width="1060"
      :height="620"
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
        month: "10",
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

        for (let d in filtered) {
          filtered[d]["1. open"] = Math.round(filtered[d]["1. open"]) 
          filtered[d]["2. high"] = Math.round(filtered[d]["2. high"])
          filtered[d]["3. low"] = Math.round(filtered[d]["3. low"])
          filtered[d]["4. close"] = Math.round(filtered[d]["4. close"])
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
    width: 1060px;
    margin: auto;
  }
</style>