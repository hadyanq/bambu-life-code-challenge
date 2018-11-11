<template>
  <svg class="chart-wrapper" :viewBox="viewBox" :width="width" :height="height">  
    <!-- background grids and bottom date lines -->
    <template v-for="i in grids">
      <line 
        :key="i.day"
        :x1="gridSpacing * i.multiplier" 
        :x2="gridSpacing * i.multiplier"   
        y1="0" 
        :y2="height - 32"
        stroke="grey" 
        stroke-width="1"
      />

      <text 
        :key="i.day + '-label'" 
        :x="(gridSpacing * i.multiplier) - 9" 
        :y="height - 10"
        v-html="formatDate(i.day)"
      />
    </template>

    <template v-for="i in bars">
      <!-- high-low bars -->
      <line
        :key="`${i.index}-line`"
        :x1="gridSpacing * i.multiplier"
        :x2="gridSpacing * i.multiplier"
        :y1="i.high"
        :y2="i.low"
        :stroke="Number(i.priceClose) > Number(i.priceOpen) ? `green` : `red`"
        stroke-width="3"
      />

      <!-- open lines -->
      <line
        :key="`${i.index}-open`"
        :x1="(gridSpacing * i.multiplier) - 6"
        :x2="(gridSpacing * i.multiplier)"
        :y1="i.open"
        :y2="i.open"
        :stroke="Number(i.priceClose) > Number(i.priceOpen) ? `green` : `red`"
        stroke-width="2"
      />

      <!-- close lines -->
      <line
        :key="`${i.index}-close`"
        :x1="(gridSpacing * i.multiplier)"
        :x2="(gridSpacing * i.multiplier) + 6"
        :y1="i.close"
        :y2="i.close"
        :stroke="Number(i.priceClose) > Number(i.priceOpen) ? `green` : `red`"
        stroke-width="2"
      />
    </template>

    <!-- left vertical price points -->
    <template v-for="i in pricePoints">
      <line 
        :key="i.multiplier + '-price'" 
        x1="-18" 
        x2="-12"
        :y1="priceSpacing * i.multiplier" 
        :y2="priceSpacing * i.multiplier"
        stroke="grey" 
        stroke-width="3"
      />

      <text 
        :key="i.multiplier + '-label'" 
        x="-6" 
        :y="(priceSpacing * i.multiplier) + 6"
        v-html="`$` + Math.round(i.price)"
      />
    </template>
  </svg>
</template>

<script>
  import moment from "moment-mini"

  export default {
    props: {
      viewBox: {
        type: String,
        default: ""
      },
      width: {
        type: Number,
        default: 1060
      }, 
      height: {
        type: Number,
        default: 620
      }, 
      ceiling: {
        type: Object,
        default: () => {}
      },
      floor: {
        type: Object,
        default: () => {}
      },
      interval: {
        type: Object,
        default: () => {}
      },
      dateFormat: {
        type: String,
        default: "DD"
      }
    },
    computed: {
      priceSpacing() {
        return this.height / (Object.keys(this.interval).length + 2)
      },
      pricePoints() {
        const ceil = Math.ceil(this.ceiling[Object.keys(this.ceiling)[0]]["2. high"])
        const floor = Math.floor(this.floor[Object.keys(this.floor)[0]]["3. low"])
        const subtractor =  (ceil - floor) / Object.keys(this.interval).length
        let multiplier = 1
        let pricePoints = [{ price: ceil, multiplier }]
        let topPrice = ceil
        
        while (topPrice - subtractor > floor) {
          multiplier++
          pricePoints.push({ price: topPrice - subtractor, multiplier })
          topPrice -= subtractor
        }
        
        multiplier++
        pricePoints.push({ price: floor, multiplier })        
        return pricePoints
      },
      gridSpacing() {      
        return this.width / (Object.keys(this.interval).length + 1)  
      },
      grids() {
        let grids = []
        let multiplier = 1

        for (let day in this.interval) {
          grids.push({ day, multiplier })
          multiplier++
        }
        
        return grids
      },
      bars() {
        let bars = []
        let index = 0
      
        for (let bar in this.interval) {                     
          bars.push({ 
            index,
            multiplier: index + 1,
            open: this.priceToYAxis(this.interval[bar]["1. open"]),
            high: this.priceToYAxis(this.interval[bar]["2. high"]), 
            low: this.priceToYAxis(this.interval[bar]["3. low"]), 
            close: this.priceToYAxis(this.interval[bar]["4. close"]),  
            priceOpen: this.interval[bar]["1. open"],
            priceClose: this.interval[bar]["4. close"]
          })
          index++
        } 
                
        return bars
      }
    },
    methods: {
      formatDate(longDate) {        
        const formatted = moment(longDate, "YYYY-MM-DD").format(this.dateFormat)
        return formatted
      },
      priceToYAxis(price) {
        const min = this.floor[Object.keys(this.floor)[0]]["3. low"]
        const max = this.ceiling[Object.keys(this.ceiling)[0]]["2. high"]         
        const drawableRange = this.priceSpacing * (this.pricePoints.length - 1)        
        const percentage = 1 - ((price - min) / (max - min))

        return (percentage * drawableRange) + this.priceSpacing 
      }
    }
  }
</script>

<style>
  .chart-wrapper {
    background-color: lightsteelblue;
  }  
</style>