<template>
  <div>
    <!-- GITHUB GIST TABS -->
    <span class="nav-group-item cursor-pointer"
          v-for="(tab, index) in githubGistTabs"
          :key="index"
          @click="addTab(tab)"
    >
      <span class="icon icon-github-circled"></span>
      {{ tab.name }}
    </span>
  </div>
</template>

<script>
import Github from '../services/Github'
import Notification from '../services/Notification'
import Storage from '../services/Storage'

export default {
  props: ['cloud', 'packageInfo'],

  data () {
    return {
      githubGistTabs: []
    }
  },

  methods: {
    getGist (gistId) {
      Github.getGist(gistId)
        .then((response) => {
          let data = response.data

          if (!data || typeof data !== 'object') {
            Notification({title: 'Error', description: 'Something did not work.', type: 'warning'})
          } else {
            if (!this.cloud.githubGists.includes(String(gistId))) {
              this.cloud.githubGists.push(String(gistId))

              Storage.save('cloud', this.cloud)
            }

            this.githubGistTabs = JSON.parse(data.files['.' + this.packageInfo.name].content).tabs
          }
        }).catch((err) => {
          if (err.response.status === 404) {
            Notification({title: 'Error', description: err.response.data.message + '.', type: 'warning'})
          } else {
            Notification({title: 'Error', description: 'Something did not work.', type: 'warning'})
          }
        })
    },

    addTab (tab) {
      try {
        let tabTemplate = {
          name: 'Unamed Tab',
          content: 'Empty'
        }

        tab = Object.assign(tabTemplate, tab)
        this.$emit('addTab', tab)
        return true
      } catch (e) {
        return false
      }
    }
  },

  created () {
    if (this.cloud.githubGists && this.cloud.githubGists.length >= 1) {
      for (let githubGist of this.cloud.githubGists) {
        this.getGist(githubGist)
      }
    }
  }
}
</script>

<style>
</style>
