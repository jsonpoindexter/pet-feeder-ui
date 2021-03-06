<template>
  <v-card class="ma-5" min-width="344" outlined ref="card">
    <v-card-title>
      <v-text-field flat :value="node.name" :disabled="!clickedEdit" ref="title" v-model="name" />
      <v-icon class="ml-sm-4" @click="onEdit">{{ clickedEdit ? 'save' : 'edit' }}</v-icon>
    </v-card-title>
    <v-card-subtitle class="text-left">{{ node.ip }}</v-card-subtitle>
    <v-card-text v-for="(schedule, index) in node.schedule" :key="`${schedule}-${index}`">
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
      <v-switch
        class="pa-3"
        v-if="node.schedule[index] >= currentEpoch && node.schedule[index] < tomorrowStart"
        label="Tomorrow"
        :input-value="!(node.schedule[index] >= currentEpoch)"
        @change="toggleTomorrow(index)"
      />
    </v-card-text>
    <v-container>
      <v-btn @click="addFeedTime">Add Feed Time</v-btn>
    </v-container>
    <v-container>
      <v-btn @click="saveFeedTimes">Save Feed Times</v-btn>
    </v-container>
    <v-container>
      <v-btn @click="feedNow">Feed Now</v-btn>
    </v-container>
  </v-card>
</template>
<script lang="ts">
import axios from 'axios'
import { Node } from "@/store/types";
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

  get currentEpoch(): number {
    return Math.round(new Date().setSeconds(0, 0) / 1000)
  }

  // Tomorrows's date at 00:00 in the morning
  get tomorrowStart(): number {
    return new Date().setHours(0, 0, 0, 0) / 1000 + 24 * 60 * 60
  }

  toggleTomorrow(index: number) {
    // Add or subtract 24hr
    this.node.schedule[index] < this.tomorrowStart
      ? (this.node.schedule[index] += 24 * 60 * 60)
      : (this.node.schedule[index] -= 24 * 60 * 60)
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
          await axios.post(`https://${this.node.ip}/name`, null, { params: { name: this.name } })
          this.node.name = this.name
        } catch (err) {
          console.log(err)
        }
      }
    }
  }

  addFeedTime() {
    this.time = this.formatHourMinutes(this.currentEpoch)
    this.node.schedule.push(this.currentEpoch)
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
    const [hours, minutes] = this.time.split(':')
    date.setHours(parseInt(hours), parseInt(minutes), 0, 0)
    // date.setMinutes))
    // If user selects time that has already passed for today, set day for tomorrow
    let epoch = Math.round(date.valueOf() / 1000)
    if (epoch < this.currentEpoch) epoch += 24 * 60 * 60
    this.node.schedule[index] = epoch
    this.$refs.menu[index].save(this.time)
  }

  async saveFeedTimes() {
    try {
      await axios.post(`https://${this.node.ip}/schedule`, this.node.schedule)
    } catch (err) {
      console.log(err)
    }
  }

  async feedNow() {
    try {
      await axios.post(`https://${this.node.ip}/feed`)
    } catch (err) {
      console.log(err)
    }
  }
}
</script>
<style scoped></style>
