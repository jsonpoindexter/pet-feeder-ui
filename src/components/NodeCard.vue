<template>
  <v-card class="mx-auto" max-width="344" outlined ref="card">
    <v-card-title>
      <v-text-field flat :value="node.name" :disabled="!clickedEdit" ref="title" v-model="name" />
      <v-icon class="ml-sm-4" @click="onEdit">{{ clickedEdit ? 'save' : 'edit' }}</v-icon>
    </v-card-title>
    <v-card-subtitle class="text-left">{{ node.ip }}</v-card-subtitle>
    <v-card-text v-for="schedule in node.schedule.filter(schedule => schedule)" :key="schedule">
      Feed: {{ schedule }}
    </v-card-text>
  </v-card>
</template>
<script lang="ts">
interface Node {
  name: string
  ip: string
  schedule: number[]
}
import axios from 'axios'
import { Component, Prop, Vue } from 'vue-property-decorator'
@Component({})
export default class NodeCard extends Vue {
  @Prop() node!: Node
  name = this.node.name
  clickedEdit = false
  $refs!: {
    title: HTMLFormElement
  }
  async onEdit(event: { target: { innerText: string } }) {
    if (event.target.innerText === 'edit') {
      this.clickedEdit = true
      this.$nextTick(() => {
        this.$refs.title.focus()
      })
    }
    if (event.target.innerText === 'save') {
      this.clickedEdit = false
      if (this.name !== this.node.name) {
        try {
          await axios.post(`http://${this.node.ip}/name`, null, { params: { name: this.name } })
          this.node.name = this.name
        } catch(err) { console.log(err)}
      }
    }
  }
}
</script>
<style scoped></style>
