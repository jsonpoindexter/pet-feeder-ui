<template>
  <div class="home">
    <h1 class="py-4">Pet Feeder App</h1>
    <NodeSearch />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import NodeSearch from '@/components/NodeSearch.vue'
@Component({
  components: { NodeSearch },
})
export default class Home extends Vue {}
</script>
