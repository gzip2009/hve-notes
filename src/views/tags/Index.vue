<template>
  <div class="">
    <v-card flat>
      <v-card-title>
        <span class="headline">🏷️ {{ $t('tag') }}</span>
        <v-spacer></v-spacer>
        <v-btn depressed color="primary" @click="newTag">{{ $t('newTag') }}</v-btn>
      </v-card-title>
      <v-card-text>
        <v-chip
          small
          @click.stop="tag.used ? null : updateTag(tag, index)"
          v-for="(tag, index) in site.tags"
          :key="tag.name"
          :close="!tag.used"
          @input="handleDelete(tag.name)"
        >
          {{ tag.name }}
        </v-chip>
      </v-card-text>
    </v-card>

    <v-dialog v-model="visible" :width="320">
      <v-card>
        <v-card-title>
          🏷️
        </v-card-title>
        <v-card-text>
          <v-text-field :label="$t('tagName')" v-model="form.name"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat @click="visible = false">{{ $t('cancel') }}</v-btn>
          <v-btn flat color="primary" :disabled="!canSubmit" @click="saveTag">{{ $t('save') }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </div>
</template>

<script lang="ts">
import { ipcRenderer, Event } from 'electron'
import Vue from 'vue'
import Component from 'vue-class-component'
import { State } from 'vuex-class'
import { Site } from '../../store/modules/site'

@Component
export default class Tags extends Vue {
  @State('site') site!: Site

  visible = false

  form = {
    name: null,
    index: -1,
  }

  get canSubmit() {
    return this.form.name
  }

  newTag() {
    this.form.name = null
    this.form.index = -1
    this.visible = true
  }

  updateTag(tag: any, index: number) {
    console.log(tag)
    this.visible = true
    this.form.name = tag.name
    this.form.index = index
  }

  saveTag() {
    ipcRenderer.send('tag-save', { ...this.form, used: false })
    ipcRenderer.once('tag-saved', (event: Event, result: any) => {
      this.$bus.$emit('site-reload')
      this.$bus.$emit('snackbar-display', '标签已保存')
      this.visible = false
    })
  }
  async handleDelete(tagValue: string) {
    const confirm = await this.$dialog.confirm({
      text: `${this.$t('deleteWarning')}`,
      title: `${this.$t('warning')}`,
    })
    if (confirm) {
      console.log('clicked', tagValue)
      ipcRenderer.send('tag-delete', tagValue)
      ipcRenderer.once('tag-deleted', (event: Event, result: any) => {
        this.$bus.$emit('site-reload')
        this.$bus.$emit('snackbar-display', '标签已删除')
        this.visible = false
      })
    }
  }
}
</script>

<style scoped>
</style>
