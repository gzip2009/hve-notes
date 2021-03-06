<template>
  <div class="">
    <v-card flat>
      <v-card-title>
        <span class="headline">📋 {{ $t('menu') }}</span>
        <v-spacer></v-spacer>
        <v-btn depressed color="primary" @click="newMenu">{{ $t('newMenu') }}</v-btn>
      </v-card-title>
      <v-data-table :headers="headers" :items="site.menus">
        <template slot="items" slot-scope="props">
          <td>{{ props.item.name }}</td>
          <td>
            {{ props.item.openType }}
          </td>
          <td>{{ props.item.link }}</td>
          <td>
            <v-btn
              flat
              icon
              color="blue lighten-2"
              @click="editMenu(props.item, props.index)"
              small
            >
              <v-icon small>edit</v-icon>
            </v-btn>
            <v-btn
              flat
              icon
              color="red lighten-2"
              @click="deleteMenu(props.item.name)"
              small
            >
              <v-icon small>delete</v-icon>
            </v-btn>
          </td>
        </template>
      </v-data-table>

    </v-card>

    <v-dialog v-model="visible" :width="480">
      <v-card>
        <v-card-text>
          <v-text-field :label="`📋  ${$t('name')}`" v-model="form.name"></v-text-field>
          <v-radio-group v-model="form.openType" row>
            <v-radio v-for="item in menuTypes" :key="item" :label="item" :value="item"></v-radio>
          </v-radio-group>
          <v-text-field label="Link" v-model="form.link"></v-text-field>
          <v-select v-model="form.link" :items="menuLinks">

          </v-select>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat @click="visible = false">{{ $t('cancel') }}</v-btn>
          <v-btn flat color="primary" :disabled="!canSubmit" @click="saveMenu">{{ $t('save') }}</v-btn>
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
import { MenuTypes } from '../../helpers/enums'
import { IMenu } from '../../interfaces/menu'
import { IPost } from '../../interfaces/post'

interface IForm {
  name: any
  index: any
  openType: string
  link: string
}

@Component
export default class Tags extends Vue {
  @State('site') site!: any

  get headers() {
    return [
      {
        text: this.$t('name'),
        value: 'title',
        sortable: false,
      },
      {
        text: this.$t('openType'),
        value: 'openType',
        sortable: false,
      },
      {
        text: this.$t('link'),
        value: 'link',
        sortable: false,
      },
      {
        text: this.$t('actions'),
        value: 'id',
        sortable: false,
      },
    ]
  }

  visible = false

  menuTypes = MenuTypes

  form: IForm = {
    name: null,
    index: null,
    openType: MenuTypes.Internal,
    link: '',
  }

  get canSubmit() {
    return this.form.name && this.form.link
  }

  get menuLinks() {
    const posts = this.site.posts.map((item: IPost) => {
      return {
        text: `📄 ${item.data.title}`,
        value: `${this.site.setting.domain}/post/${item.fileName}/`,
      }
    })
    return [
      {
        text: '🏠 Homepage',
        value: this.site.setting.domain,
      },
      ...posts,
    ]
  }

  newMenu() {
    this.form.name = null
    this.form.index = null
    this.form.openType = MenuTypes.Internal
    this.form.link = ''
    this.visible = true
  }
  editMenu(menu: IMenu, index: number) {
    this.visible = true
    this.form.index = index
    this.form.name = menu.name
    this.form.openType = menu.openType
    this.form.link = menu.link
  }

  saveMenu() {
    console.log('click save menu', this.form)
    ipcRenderer.send('menu-save', { ...this.form })
    ipcRenderer.once('menu-saved', (event: Event, result: any) => {
      this.$bus.$emit('site-reload')
      this.$bus.$emit('snackbar-display', this.$t('menuSuccess'))
      this.visible = false
    })
  }

  async deleteMenu(menuValue: string) {
    const confirm = await this.$dialog.confirm({
      text: `${this.$t('deleteWarning')}`,
      title: `${this.$t('warning')}`,
    })
    if (confirm) {
      ipcRenderer.send('menu-delete', menuValue)
      ipcRenderer.once('menu-deleted', (event: Event, result: any) => {
        this.$bus.$emit('site-reload')
        this.$bus.$emit('snackbar-display', this.$t('menuDelete'))
        this.visible = false
      })
    }
  }
}
</script>

<style lang="stylus" scoped>
</style>
