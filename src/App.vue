<template>
  <div>
    <header>
      <h2>{{ title }}</h2>
    </header>

    <div class="sidebar">
      <ul>
        <template v-for="i in symbols">
          <li :key="i" @click="symbol = i">{{ i }}</li>
        </template>
      </ul>
    </div>

    <div class="chart">
      <price-chart 
        v-if="priceData"
        viewBox="-20 0 1080 540"
        :width="1060"
        :height="540"
        :ceiling="highestPrice"
        :floor="lowestPrice"
        :interval="filteredByMonth"
      />
      <div class="placeholder" v-else>
        <spinner v-if="loading" :width="100"/>
        <h3 v-else>Please select a symbol</h3>
      </div>
    </div>
  </div>
</template>

<script>
  const apiKey = "79IE7TI76DT1E968"
  const apiUrl = "https://www.alphavantage.co"

  import axios from "axios"
  import Spinner from "./components/Spinner.vue"
  import PriceChart from "./components/PriceChart.vue"

  export default {
    components: { Spinner,  PriceChart },
    data() {
      return {
        title: "Bambu.Life Code Challenge",
        priceData: null,
        month: "11",
        symbol: "",
        symbols: [
          "MSFT", "AAPL", "INTC", "NFLX",
          "ORCL", "CMCSA", "GOOG", "LUV",
          "HOG", "GOOGL", "AMZN"
        ],
        loading: false
      }
    },
    watch: {
      async symbol() {
        this.title = this.symbol
        this.loading = true
        this.priceData = null

        try {
          const endpoint = `${apiUrl}/query?function=TIME_SERIES_DAILY&symbol=${this.symbol}&apikey=${apiKey}`
          const resp = await axios.get(endpoint).then(resp => resp.data)
          this.priceData = resp["Time Series (Daily)"]
        } catch (e) {
          console.log(e)
        } finally {
          this.loading = false
        }
      }
    },
    computed: {
      filteredByMonth() {
        let filtered = {}
        const ordered = {}

        for (let d in this.priceData) {
          const removeDays = d.substring(0, 7)
          if (removeDays === `2018-${this.month}`) {
            filtered[d] = this.priceData[d]
          }
        }

        Object.keys(filtered).sort().forEach((key) => ordered[key] = filtered[key])        
        return ordered
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

<style scoped>
  header {
    background-color: darkred;
    padding: 2px;
    margin-bottom: 20px;
  }

  header h2 {
    margin-left: 20px;
    font-size: 14px;
    color: white;
  }

  .placeholder {
    width: 1060px;
    height: 540px;
    text-align: center;
  }

  .placeholder h3 {
    padding-top: 102px;    
    font-weight: bold;
    color: white;
  }

  .sidebar {
    float: right;
    width: 290px;
  }

  .sidebar ul {    
    list-style: none;
  }

  .sidebar li {
    background-color: darkcyan;
    padding: 12px;
    color: white;
    font-weight: bold;
    text-align: center;
    margin-bottom: 2px;
  }

  .sidebar li:hover {
    cursor: pointer;
    background-color: lightsteelblue;
  }
</style>