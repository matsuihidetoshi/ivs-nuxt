<template>
  <v-row
    justify="center"
    align="center"
  >
    <v-col cols="12">
      <v-card>
        <video
          id="video-player"
          class="video-js vjs-fluid vjs-big-play-centered"
          controls
          playsinline
          autoplay
        />
      </v-card>
    </v-col>
  </v-row>
</template>
<script>
import videojs from 'video.js'

export default {
  data () {
    return {
      ivs: null,
      streamUrl: process.env.streamUrl,
      player: null,
      timestamp: new Date().getTime()
    }
  },
  mounted () {
    if (this.ivs === null) {
      this.ivs = require('amazon-ivs-player')
      this.ivs.registerIVSTech(videojs, {
        wasmBinary: '/_nuxt/amazon-ivs-wasmworker.min.wasm',
        wasmWorker: '/_nuxt/amazon-ivs-wasmworker.min.js'
      })
      this.ivs.registerIVSQualityPlugin(videojs)
    }

    this.startStream()
  },
  beforeDestroy () {
    this.player.dispose()
  },
  methods: {
    startStream () {
      this.player = videojs('video-player', {
        techOrder: ['AmazonIVS']
      })
      this.player.enableIVSQualityPlugin()
      this.player.src(this.streamUrl)
    }
  }
}
</script>
