<template>
  <div class="home">
    <radiance-graph :circles="7" :max="10" :scale="30" :firstValue="8" :secondValue="9" :thirdValue="5"></radiance-graph>
  </div>
</template>

<script>
// @ is an alias to /src
import RadianceGraph from '@/components/RadianceGraph.vue'

export default {
  name: 'home',
  components: {
    RadianceGraph
  }
}
</script>
