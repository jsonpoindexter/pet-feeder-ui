<template>
  <div>
    <div class="text-center">
      <v-text-field label="network address" v-model="networkAddress" placeholder="Enter network address" />
      <v-btn :loading="searching" medium :disabled="searching || searchDisabled" @click="onSearch">Search</v-btn>
    </div>
    <template>
      <v-container class="d-flex flex-row flex-wrap justify-center">
        <NodeCard v-for="node in nodes" :key="node.ip" :node="node" />
      </v-container>
    </template>
  </div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import axios from 'axios'
import NodeCard from '@/components/NodeCard.vue'
import { Node } from '@/store/types'

@Component({
  components: { NodeCard },
})
export default class NodeSearch extends Vue {
  networkAddress = '192.168.1'
  nodesString = window.localStorage.getItem('nodes')
  savedIps: string[] = this.nodesString ? JSON.parse(this.nodesString) : [] // saved IP address of previously discovered nodes
  nodes: Node[] = [] // Petfeeder nodes
  searching = false // Used for Search btn animation and enable/disabled
  searchDisabled = false // Used only for enable/disabled

  async created() {
    // Fetch previously discovered nodes
    this.searchDisabled = true
    await this.fetchNodes(this.savedIps)
    this.searchDisabled = false
  }
  async onSearch() {
    this.searching = true
    this.nodes = []
    const length = 254
    const lowerBound = 1
    const ipAddresses: string[] = Array.from(new Array(length), (x, i) => `${this.networkAddress}.${i + lowerBound}`)
    await this.fetchNodes(ipAddresses)
    window.localStorage.setItem('nodes', JSON.stringify(this.nodes.map(node => node.ip)))
    this.searching = false
  }

  async fetchNodes(ipAddresses: string[]) {
    await Promise.all(
      ipAddresses.map(async ip => {
        try {
          const response = await axios.get(`https://${ip}/schedule`, {
            // TODO: implement increase timeout
            timeout: 5000,
          })
          this.nodes.push({
            ip,
            ...response.data,
          })

          // eslint-disable-next-line no-empty
        } catch (err) {}
      }),
    )
  }
}
</script>
<style scoped></style>
