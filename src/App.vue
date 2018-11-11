<template>
  <div>
    <header>
      <h2>{{ title }}</h2>
      <interval-control 
        :price-data="priceData"
        :selected-interval="interval" 
        :selected-month="month"
        :selected-year="year"
        @setInterval="interval = $event"
        @setMonth="month = $event"
        @setYear="year = $event"
      />
    </header>

    <div class="sidebar">
      <ul>
        <template v-for="i in symbols">
          <li v-if="!loading" :key="i" @click="symbol = i">{{ i }}</li>
          <li v-else :key="i" class="disabled">{{ i }}</li>
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
        :interval="filtered"
        :date-format="dateFormat"
      />
      <div class="placeholder" v-else>
        <spinner v-if="loading" :width="100"/>
        <template v-else>
          <h3>
            <template v-if="err">{{ err }}</template>
            <template v-else>Please select a symbol</template>
          </h3>      
        </template>
      </div>
    </div>
  </div>
</template>

<script>
  const apiKey = "79IE7TI76DT1E968"
  const apiUrl = "https://www.alphavantage.co"

  import axios from "axios"
  import Spinner from "./components/Spinner.vue"
  import IntervalControl from "./components/IntervalControl.vue"
  import PriceChart from "./components/PriceChart.vue"

  export default {
    components: { Spinner, IntervalControl, PriceChart },
    data() {
      return {
        title: "Bambu.Life Code Challenge",
        priceData: null,
        interval: "monthly",
        month: "11",
        year: "2018",
        symbol: "",
        symbols: [
          "MSFT", "AAPL", "INTC", "NFLX",
          "ORCL", "CMCSA", "GOOG", "LUV",
          "HOG", "GOOGL", "AMZN"
        ],
        loading: false,
        err: null
      }
    },
    watch: {
      symbol() { 
        this.fetchData(this.interval, this.symbol) 
      },
      interval() { 
        this.fetchData(this.interval, this.symbol) 
      }
    },
    computed: {
      filtered() {
        let filtered = {}
        let ordered = {}

        for (let d in this.priceData) {
          switch (this.interval) {
            case "daily":
              const removeDays = d.substring(0, 7)
              if (removeDays === `2018-${this.month}`) {
                filtered[d] = this.priceData[d]
              }
              break
            case "monthly":            
              const getYear = d.substring(0, 4)
              if (getYear === this.year) {
                filtered[d] = this.priceData[d]
              }
            }
        }
        
        Object.keys(filtered).sort().forEach((key) => ordered[key] = filtered[key])        
        return ordered
      }, 
      highestPrice() {
        const days = Object.keys(this.filtered)
        let currentD  = days[0]
        let ceiling = { [currentD]: this.filtered[currentD] }

        for (let d in this.filtered) {
          if (
            Number(this.filtered[d]["2. high"]) > 
            Number(ceiling[currentD]["2. high"])
          ) {
            currentD = d
            ceiling = { [currentD]: this.filtered[currentD] }
          }
        }

        return ceiling
      },
      lowestPrice() {        
        const days = Object.keys(this.filtered)
        let currentD  = days[0]
        let floor = { [currentD]: this.filtered[currentD] }

        for (let d in this.filtered) {
          if (
            Number(this.filtered[d]["3. low"]) < 
            Number(floor[currentD]["3. low"])
          ) {
            currentD = d
            floor = { [currentD]: this.filtered[currentD] }
          }
        }

        return floor
      },
      dateFormat() {
        switch (this.interval) {
          case "daily": return "DD"
          case "monthly": return "MMM"
        }
      }
    },
    methods: {
      async fetchData(interval, symbol) {
        let apiFunction, dataKey
        this.title = symbol
        this.loading = true
        this.priceData = null

        switch (interval) {
          case "daily":
            apiFunction = "TIME_SERIES_DAILY"
            dataKey = "Time Series (Daily)"
            break
          case "monthly":
            apiFunction = "TIME_SERIES_MONTHLY"
            dataKey = "Monthly Time Series"
            break
        }

        try {
          const endpoint = `${apiUrl}/query?function=${apiFunction}&symbol=${this.symbol}`
          const resp = await axios.get(`${endpoint}&apikey=${apiKey}`).then(resp => resp.data)                    
          this.priceData = resp[dataKey]
          this.err = resp["Note"]
        } catch (e) {
          console.log(e)
        } finally {
          this.loading = false
        }
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
    display: inline-block;
    margin-left: 20px;
    font-size: 14px;
  }

  .placeholder {
    width: 1060px;
    height: 540px;
    text-align: center;
  }

  .placeholder h3 {
    padding-top: 102px;    
    font-weight: bold;
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
    font-weight: bold;
    text-align: center;
    margin-bottom: 2px;
  }

  .sidebar li:hover {
    cursor: pointer;
    background-color: lightsteelblue;
  }

  .sidebar .disabled {
    opacity: 0.5;
    background-color: grey;
  }
</style>