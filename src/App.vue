<template>
  <div id="app">
    <marking-picture
      ref="marker"
      :options="config"
      :confirmation="confirmation"
      :ids="ids"
      :data.sync="data"
    >
      <template #top="{ props }">
        <div style="background: white; padding: 5px; border-radius: 5px">
          {{ props.name }}
        </div>
      </template>
      <template #middle="{ props }">
        <div
          style="
            background: white;
            border-radius: 50px;
            height: 50px;
            width: 50px;
            display: flex;
            justify-content: center;
            align-items: center;
          "
        >
          {{ props.name }}
        </div>
      </template>
      <template #top-right>
        <div v-for="lamp in ['red', 'green', 'yellow', 'orange']" :key="lamp">
          <div
            :style="{
              background: lamp,
              padding: '8px',
              'border-radius': '8px',
              color: 'black',
              'margin-bottom': '4px'
            }"
          ></div>
        </div>
      </template>
      <template #bottom-right>
        <div
          style="
            color: white;
            background: black;
            height: 25px;
            width: 25px;
            border-radius: 25px;
            text-align: center;
            vertical-align: middle;
            line-height: 1.5;
            cursor: pointer;
          "
        >
          x
        </div>
      </template>
    </marking-picture>
    <div>
      <button style="margin-right: 5px" @click="saveShape">Save shape</button>
      <button @click="undo">Undo</button>
    </div>
  </div>
</template>

<script>
import MarkingPicture from './components/MarkingPicture.vue'

export default {
  name: 'App',
  components: {
    MarkingPicture
  },
  data() {
    return {
      config: {
        bgUrl: require('./assets/contoh toyota.png'),
        // 1264 521
        canvas: {
          height: 521,
          width: 1264
        },
        additional: {
          top: {
            x: 0,
            y: -30
          },
          right: {
            x: -21,
            y: 5
          }
        }
        // 'https://images.unsplash.com/photo-1506521781263-d8422e82f27a?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxzZWFyY2h8MXx8cGFya2luZyUyMGxvdHxlbnwwfHwwfHw%3D&w=1000&q=80'
      },
      data: []
    }
  },
  watch: {
    data(value) {
      console.log(value)
    }
  },
  methods: {
    confirmation() {
      return confirm('Complete drawing?')
    },
    ids() {
      return Math.floor(Math.random() * 10000)
    },
    saveShape() {
      this.$refs.marker.finishingShape()
    },
    undo() {
      this.$refs.marker.undo()
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: start;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
