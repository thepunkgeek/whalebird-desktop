<template>
<div id="account_profile"
     v-loading="loading"
     element-loading-text="Loading..."
     element-loading-spinner="el-icon-loading"
     element-loading-background="rgba(0, 0, 0, 0.8)">
  <div class="header-background" v-bind:style="{ backgroundImage: 'url(' + account.header + ')' }">
    <div class="header">
      <div class="follow-follower" v-if="relationship !== null && relationship !== ''">
        <div class="follower-status">
          <span class="status" v-if="relationship.followed_by">Follows you</span>
          <span class="status" v-else>Doesn't follow you</span>
        </div>
        <div class="follow-status">
          <div v-if="relationship.following" class="unfollow" @click="unfollow(account)">
            <icon name="user-times" scale="1.5"></icon>
          </div>
          <div v-else class="follow" @click="follow(account)">
            <icon name="user-plus" scale="1.5"></icon>
          </div>
        </div>
        <div class="clearfix"></div>
        <div class="more">
          <popper trigger="click" :options="{placement: 'bottom'}">
            <div class="popper">
              <ul class="menu-list">
                <li role="button" @click="openBrowser(account)">
                  Open in Browser
                </li>
              </ul>
            </div>

            <el-button slot="reference" type="text">
              <icon name="ellipsis-h"></icon>
            </el-button>
          </popper>
        </div>
      </div>
      <div class="icon">
        <img :src="account.avatar" />
      </div>
      <div class="username">
        {{ username(account) }}
      </div>
      <div class="account">
        @{{ account.acct }}
      </div>
      <div class="note" v-html="account.note" @click.capture.prevent="noteClick"></div>
    </div>
  </div>
  <el-row class="basic-info">
    <el-col :span="8" :class="activeTab === 1 ? 'info info-active' : 'info'" @click="changeTab">
      <el-button type="text" class="tab" @click="changeTab(1)">
        <div class="title">Toots</div>
        <div class="count">{{ account.statuses_count }}</div>
      </el-button>
    </el-col>
    <el-col :span="8" :class="activeTab === 2 ? 'info info-active' : 'info'">
      <el-button type="text" class="tab" @click="changeTab(2)">
        <div class="title">Follows</div>
        <div class="count">{{ account.following_count }}</div>
      </el-button>
    </el-col>
    <el-col :span="8" :class="activeTab === 3 ? 'info info-active' : 'info'">
      <el-button type="text" class="tab" @click="changeTab(3)">
        <div class="title">Followers</div>
        <div class="count">{{ account.followers_count }}</div>
      </el-button>
    </el-col>
  </el-row>
  <div class="timeline">
    <timeline :account="account" v-if="activeTab === 1"></timeline>
    <follows :account="account" v-if="activeTab === 2"></follows>
    <followers :account="account" v-if="activeTab === 3"></followers>
  </div>
</div>
</template>

<script>
import { mapState } from 'vuex'
import { shell } from 'electron'
import Timeline from './AccountProfile/Timeline'
import Follows from './AccountProfile/Follows'
import Followers from './AccountProfile/Followers'

export default {
  name: 'account-profile',
  components: {
    Timeline,
    Follows,
    Followers
  },
  data () {
    return {
      activeTab: 1
    }
  },
  computed: {
    ...mapState({
      account: state => state.TimelineSpace.Contents.SideBar.AccountProfile.account,
      relationship: state => state.TimelineSpace.Contents.SideBar.AccountProfile.relationship,
      loading: state => state.TimelineSpace.Contents.SideBar.AccountProfile.loading,
      theme: (state) => {
        return {
          '--theme-mask-color': state.App.theme.wrapper_mask_color,
          '--theme-border-color': state.App.theme.border_color,
          '--theme-primary-color': state.App.theme.primary_color
        }
      }
    })
  },
  watch: {
    account: function () {
      this.activeTab = 1
    }
  },
  methods: {
    username (account) {
      if (account.display_name !== '') {
        return account.display_name
      } else {
        return account.username
      }
    },
    noteClick (e) {
      const link = findLink(e.target)
      if (link !== null) {
        shell.openExternal(link)
      }
    },
    follow (account) {
      this.$store.dispatch('TimelineSpace/Contents/SideBar/AccountProfile/follow', account)
        .catch(() => {
          this.$message({
            message: 'Could not follow this user',
            type: 'error'
          })
        })
    },
    unfollow (account) {
      this.$store.dispatch('TimelineSpace/Contents/SideBar/AccountProfile/unfollow', account)
        .catch(() => {
          this.$message({
            message: 'Could not unfollow this user',
            type: 'error'
          })
        })
    },
    changeTab (index) {
      this.activeTab = index
    },
    openBrowser (account) {
      shell.openExternal(account.url)
    }
  }
}

function findLink (target) {
  if (target.localName === 'a') {
    return target.href
  }
  if (target.parentNode === undefined || target.parentNode === null) {
    return null
  }
  if (target.parentNode.getAttribute('class') === 'note') {
    return null
  }
  return findLink(target.parentNode)
}
</script>

<style lang="scss" scoped>
#account_profile {
  height: 100%;

  .header-background {
    background-position: 50% 50%;
    background-size: cover;
  }

  .header {
    background-color: var(--theme-wrapper-mask-color);
    text-align: center;
    padding: 12px;
    box-sizing: border-box;
    word-wrap: break-word;
    font-size: 14px;

    .follow-follower {
      .follower-status {
        float: left;

        .status {
          border-radius: 4px;
          background-color: rgba(0, 0, 0, 0.3);
          padding: 4px 8px;
        }
      }
      .follow-status {
        float: right;

        .follow {
          cursor: pointer;
        }

        .unfollow {
          color: #409eff;
          cursor: pointer;
        }
      }

      .more {
        float: right;

        .menu-list {
          padding: 0;
          font-size: 0.8em;
          list-style-type: none;
          line-height: 20px;
          text-align: left;
          color: #303133;

          li {
            box-sizing: border-box;
            padding-left: 0.5em;
            padding-bottom: 0.5em;

            &:hover {
              background-color: #f2f6fc;
              cursor: pointer;
            }
          }
        }
      }
    }

    .icon {
      padding: 12px;

      img {
        width: 72px;
        border-radius: 8px;
      }
    }

    .username {
      overflow: hidden;
      text-overflow: ellipsis;
      font-size: 24px;
      margin: 0 auto 12px auto;
    }

    .account {
      color: #409eff;
    }
  }

  .basic-info {
    .info {
      border-top: solid 1px var(--theme-border-color);
      border-bottom: solid 1px var(--theme-border-color);
      border-left: solid 1px var(--theme-border-color);
      padding: 0 4px;

      .tab {
        margin: 0;
        padding: 0;
        width: 100%;
        text-align: left;
        line-height: 20px;
      }

      .title {
        font-size: 11px;
        color: #909399;
      }

      .count {
        font-weight: 800;
        font-size: 16px;
        color: var(--theme-primary-color);
      }
    }

    .info-active {
      border-bottom: solid 1px #409eff;

      .count {
        color: #409eff;
      }
    }

    .info:first-of-type {
      border-left: none;
    }
  }

  .timeline {
    font-size: 12px;
  }
}
</style>
