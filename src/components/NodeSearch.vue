<template>
  <div>
    <div class="text-center">
      <v-text-field label="network address" v-model="networkAddress" placeholder="Enter network address" />
      <v-btn :loading="searching" medium :disabled="searching" @click="onSearch">Search</v-btn>
    </div>
    <template>
      <v-container class="d-flex flex-row flex-wrap justify-center">
        <NodeCard v-for="node in nodes" :key="node.ip" :node="node" />
      </v-container>
    </template>
  </div>
</template>
<script>
import { Component, Vue } from 'vue-property-decorator'
import axios from 'axios'
import NodeCard from '@/components/NodeCard'

@Component({
  components: { NodeCard },
})
export default class NodeSearch extends Vue {
  networkAddress = '192.168.1'
  nodes = []
  searching = false
  async onSearch() {
    this.searching = true
    this.nodes = []
    const length = 254
    const lowerBound = 1
    const array = Array.from(new Array(length), (x, i) => i + lowerBound)
    await Promise.all(
      array.map(async num => {
        try {
          const response = await axios.get(`https://${this.networkAddress}.${num}/schedule`, {
            // TODO: implement increase timeout
            timeout: 5000,
          })
          this.nodes.push({
            ip: `${this.networkAddress}.${num}`,
            ...response.data,
          })
          // eslint-disable-next-line no-empty
        } catch (err) {}
      }),
    )
    this.searching = false
  }
}
</script>
<style scoped></style>
