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

      <v-chip
        color="red"
        class="mt-3"
      >
        {{ metadata === '' ? 'Waiting for the number of viewers...' : count + ' viewers' }}
      </v-chip>
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
      timestamp: new Date().getTime(),
      metadata: '',
      count: 0
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
      const playerEvent = this.player.getIVSEvents().PlayerEventType
      this.player.getIVSPlayer().addEventListener(playerEvent.TEXT_METADATA_CUE, (cue) => {
        this.metadata = cue.text
        this.count = this.metadata.split(',')[0]
      })
    }
  }
}
</script>
