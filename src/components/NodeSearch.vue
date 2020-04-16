<template>
  <div>
    <div>
      <input
        type="text"
        v-model="networkAddress"
        placeholder="Enter network address"
      />
      <button @click="onSearch">Search</button>
    </div>
    <template v-if="searching">
      <h1 class="loading">
        Searching<span>.</span><span>.</span><span>.</span>
      </h1>
    </template>
    <template v-else>
      <div v-for="node in nodes" :key="node.ip">
        {{ node }}
      </div>
    </template>
  </div>
</template>
<script>
import { Component, Vue } from "vue-property-decorator";
import axios from "axios";
@Component({})
export default class NodeSearch extends Vue {
  networkAddress = "192.168.1";
  nodes = [];
  searching = false;
  async onSearch() {
    this.searching = true;
    this.nodes = [];
    const length = 254;
    const lowerBound = 1;
    const array = Array.from(new Array(length), (x, i) => i + lowerBound);
    await Promise.all(
      array.map(async num => {
        try {
          const response = await axios.get(
            `http://${this.networkAddress}.${num}/schedule`,
            {
              // TODO: implement increase timeout
              timeout: 1000
            }
          );
          this.nodes.push({
            ip: `${this.networkAddress}.${num}`,
            ...response.data
          });
          // eslint-disable-next-line no-empty
        } catch (err) {}
      })
    );
    this.searching = false;
  }
}
</script>
<style scoped></style>
