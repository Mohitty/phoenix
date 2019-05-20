<template>
  <oc-app-side-bar :disableAction="false" uk-sticky="offset: 120" @close="close()" class="uk-padding-small">
    <template v-if="this.items.length === 0">
      <template slot="title">
        <div class="uk-inline">
          <oc-icon name="folder" size="large" />
        </div>
        <div class="uk-inline">
          <template v-if="currentFolder.path === ''">
            Home
          </template>
          <template v-else>
            {{ currentFolder.name }}
          </template>
        </div>
      </template>
    </template>
    <template v-else>
      <template slot="title" v-if="items.length === 1">
        <div class="uk-inline">
          <oc-icon :name="fileTypeIcon(items[0])" size="large" />
        </div>
        <div class="uk-inline">
          <div class="uk-flex uk-flex-middle">
            <span class="uk-margin-small-right">{{ getTabName }}</span> <oc-icon name="link" aria-label="Close"/>
          </div>
          <div>
            <oc-star class="uk-inline" :shining="items[0].starred"/> {{ items[0].size | fileSize }}, {{ formDateFromNow(items[0].mdate) }}
          </div>
        </div>
      </template>
      <template slot="title" v-else>
        <span class="uk-text-lead" v-translate>Multiple files selected</span>
      </template>
      <template slot="content">
        <oc-tabs>
            <oc-tab-item :active="key == activeTab" @click="activeTab = key" v-for="(tab, key) of fileSideBars" :key="tab.name">
              {{ tab.component.title($gettext) }}
            </oc-tab-item>
        </oc-tabs>
        <component v-if="fileSideBars.length > 0" v-bind:is="activeTabComponent.component" @reload="$emit('reload')"></component>
      </template>
    </template>
  </oc-app-side-bar>
</template>

<script>
import Mixins from '../mixins'
import { mapActions, mapGetters } from 'vuex'

export default {
  mixins: [Mixins],
  props: ['items'],
  name: 'FileDetails',
  data () {
    return {
      /** String name of the tab that is activated */
      activeTab: 0
    }
  },
  methods: {
    ...mapActions('Files', ['deleteFiles', 'toggleSidebar']),
    close () {
      this.$emit('reset')
      this.toggleSidebar(false)
    },
    showSidebar (app) {
      this.activeTab = app
    },
    downloadFiles () {
      this.downloadFile(this.items[0])
    },
    deleteSelectedFiles () {
      this.deleteFiles({
        client: this.$client,
        files: this.items
      })
    }
  },
  computed: {
    ...mapGetters(['getToken', 'fileSideBars']),
    ...mapGetters('Files', ['currentFolder']),
    getTabName () {
      if (this.items.length === 0) {
        return ''
      }
      if (this.items.length > 1) {
        return this.$gettext('Multiple Files')
      } else {
        // return (this.items[0].name.length > 16) ? `${this.items[0].name.substr(0, 10)}...` : this.items[0].name
        return this.items[0].name
      }
    },
    activeTabComponent () {
      return this.fileSideBars[this.activeTab]
    }
  }
}
</script>

<style scoped>
  .oc-app-side-bar {
    border-left: 1px solid #ccd4e0;
    height: 100%;
  }
</style>
