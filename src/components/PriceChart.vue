<template>
  <svg class="chart-wrapper" width="860px" height="420px">
    <line x1="30" x2="30" y1="30" y2="140" stroke="green" stroke-width="3"/>
    
    <line x1="20" x2="30" y1="50" y2="50" stroke="green" stroke-width="3"/>
    <line x1="30" x2="40" y1="120" y2="120" stroke="green" stroke-width="3"/>
  </svg>
</template>

<script>
  export default {
    props: {
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
      }
    }
  }
</script>