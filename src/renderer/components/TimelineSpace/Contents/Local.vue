<template>
  <div id="local">
    <div class="local-timeline" v-for="message in timeline" v-bind:key="message.id">
      <toot :message="message" v-on:update="updateToot"></toot>
    </div>
    <div class="loading-card" v-loading="lazyLoading" :element-loading-background="backgroundColor">
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import Toot from './Cards/Toot'

export default {
  name: 'local',
  components: { Toot },
  computed: {
    ...mapState({
      timeline: state => state.TimelineSpace.Contents.Local.timeline,
      lazyLoading: state => state.TimelineSpace.Contents.Local.lazyLoading,
      backgroundColor: state => state.App.theme.background_color
    })
  },
  created () {
    const loading = this.$loading({
      lock: true,
      text: 'Loading',
      spinner: 'el-icon-loading',
      background: 'rgba(0, 0, 0, 0.7)'
    })
    this.initialize()
      .then(() => {
        loading.close()
      })
      .catch(() => {
        loading.close()
      })
    document.getElementById('scrollable').addEventListener('scroll', this.onScroll)
  },
  beforeDestroy () {
    this.$store.dispatch('TimelineSpace/Contents/Local/stopLocalStreaming')
  },
  destroyed () {
    this.$store.commit('TimelineSpace/Contents/Local/updateTimeline', [])
    if (document.getElementById('scrollable') !== undefined && document.getElementById('scrollable') !== null) {
      document.getElementById('scrollable').removeEventListener('scroll', this.onScroll)
    }
  },
  methods: {
    async initialize () {
      try {
        await this.$store.dispatch('TimelineSpace/Contents/Local/fetchLocalTimeline')
      } catch (err) {
        this.$message({
          message: 'Could not fetch timeline',
          type: 'error'
        })
      }
      this.$store.dispatch('TimelineSpace/Contents/Local/startLocalStreaming')
        .catch(() => {
          this.$message({
            message: 'Failed to start streaming',
            type: 'error'
          })
        })
    },
    updateToot (message) {
      this.$store.commit('TimelineSpace/Contents/Local/updateToot', message)
    },
    onScroll (event) {
      if (((event.target.clientHeight + event.target.scrollTop) >= document.getElementById('local').clientHeight - 10) && !this.lazyloading) {
        this.$store.dispatch('TimelineSpace/Contents/Local/lazyFetchTimeline', this.timeline[this.timeline.length - 1])
          .catch(() => {
            this.$message({
              message: 'Could not fetch timeline',
              type: 'error'
            })
          })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.loading-card {
  height: 60px;
}

.loading-card:empty {
  height: 0;
}
</style>
