<template>
  <v-card class="mx-auto" max-width="344" outlined ref="card">
    <v-card-title>
      <v-text-field flat :value="node.name" :disabled="!clickedEdit" ref="title" v-model="name" />
      <v-icon class="ml-sm-4" @click="onEdit">{{ clickedEdit ? 'save' : 'edit' }}</v-icon>
    </v-card-title>
    <v-card-subtitle class="text-left">{{ node.ip }}</v-card-subtitle>
    <v-card-text v-for="(schedule, index) in node.schedule" :key="schedule">
      <v-menu
        v-model="menu[index]"
        ref="menu"
        :close-on-content-click="false"
        :nudge-right="40"
        :return-value.sync="time"
        transition="scale-transition"
        offset-y
        max-width="290px"
        min-width="290px"
      >
        <template v-slot:activator="{ on }">
          <v-text-field
            :value="formatHourMinutes(node.schedule[index])"
            :label="`Feed Time ${index + 1}`"
            prepend-icon="access_time"
            append-icon="delete"
            @click:append="removeFeedTime(index)"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        <v-time-picker v-if="menu[index]" v-model="time" full-width @click:minute="setFeedTime(index)"></v-time-picker>
      </v-menu>
    </v-card-text>
    <v-container>
      <v-btn @click="addFeedTime">Add Feed Time</v-btn>
    </v-container>
    <v-container>
      <v-btn @click="saveFeedTimes">Save Feed Times</v-btn>
    </v-container>
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
  time = ''
  menu: boolean[] = []
  clickedEdit = false
  $refs!: {
    title: HTMLFormElement
    menu: HTMLFormElement[]
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
        } catch (err) {
          console.log(err)
        }
      }
    }
  }
  addFeedTime() {
    const date = new Date()
    this.node.schedule.push(Math.round(date.getTime() / 1000))
  }

  formatHourMinutes(epoch: number) {
    const date = new Date(epoch * 1000)
    return `${date.getHours().toLocaleString('en-US', {
      minimumIntegerDigits: 2,
      useGrouping: false,
    })}:${date.getMinutes().toLocaleString('en-US', { minimumIntegerDigits: 2, useGrouping: false })}`
  }

  removeFeedTime(index: number) {
    this.node.schedule.splice(index, 1)
  }

  setFeedTime(index: number) {
    const date = new Date()
    date.setHours(parseInt(this.time.split(':')[0]))
    date.setMinutes(parseInt(this.time.split(':')[1]))
    this.node.schedule[index] = Math.round(date.getTime() / 1000)
    this.$refs.menu[index].save(this.time)
  }

  async saveFeedTimes() {
    try {
      await axios.post(`http://${this.node.ip}/schedule`, this.node.schedule)
    } catch (err) {
      console.log(err)
    }
  }
}
</script>
<style scoped></style>
