<template>
  <div id="app" :style="theme">
    <router-view></router-view>
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  name: 'Whalebird',
  computed: {
    ...mapState({
      theme: (state) => {
        return {
          '--theme-background-color': state.App.theme.background_color,
          '--theme-selected-background-color': state.App.theme.selected_background_color,
          '--theme-global-header-color': state.App.theme.global_header_color,
          '--theme-side-menu-color': state.App.theme.side_menu_color,
          '--theme-primary-color': state.App.theme.primary_color,
          '--theme-regular-color': state.App.theme.regular_color,
          '--theme-secondary-color': state.App.theme.secondary_color,
          '--theme-border-color': state.App.theme.border_color,
          '--theme-header-menu-color': state.App.theme.header_menu_color,
          '--theme-wrapper-mask-color': state.App.theme.wrapper_mask_color
        }
      }
    })
  },
  created () {
    this.$store.dispatch('App/watchShortcutsEvents')
    this.$store.dispatch('App/loadPreferences')
  },
  destroyed () {
    this.$store.dispatch('App/removeShortcutsEvents')
  }
}
</script>

<style lang="scss">
body { font-family: 'Noto Sans', sans-serif; }

html, body, #app {
  --theme-background-color: #ffffff;
  --theme-selected-background-color: #f2f6fc;
  --theme-global-header-color: #4a5664;
  --theme-side-menu-color: #373d48;
  --theme-primary-color: #303133;
  --theme-regular-color: #606266;
  --theme-secondary-color: #909399;
  --theme-border-color: #ebeef5;
  --theme-header-menu-color: #ffffff;
  --theme-wrapper-mask-color: rgba(255, 255, 255, 0.7);

  background-color: var(--theme-background-color);
  color: var(--theme-primary-color);

  a:link,
  a:visited,
  a:hover,
  a:active,
  a:focus {
    color: #409eff;
  }
}

html, body, #app, #global_header {
  height: 100%;
  margin: 0;
}

p {
  margin: 8px 0;
}

.clearfix:after {
  content:" ";
  display:block;
  clear:both;
}
</style>
