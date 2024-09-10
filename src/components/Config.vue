<template>

  <v-card>
    <v-toolbar class="position-fixed" style="z-index: 1;">
      <v-btn icon="mdi-close" @click="$emit('close')"></v-btn>
      <v-toolbar-title>设置</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items>
        <v-btn text="保存" prepend-icon="mdi-check" @click="saveData"></v-btn>
      </v-toolbar-items>
    </v-toolbar>
    <div class="d-flex flex-row align-start justify-start overflow-hidden" v-if="prepared"
      style="height: calc(100vh - 64px); margin-top: 64px;">
      <v-sheet class="d-flex overflow-y-auto overflow-x-hidden" height="calc(100vh - 64px)">
        <v-list activatable width="240" mandatory v-model:activated="selectedTab">
          <v-list-item value="groups" prepend-icon="mdi-account-multiple" title="小组管理"></v-list-item>
          <v-list-item value="recorditems" prepend-icon="mdi-view-list" title="记录项目"></v-list-item>
          <v-list-item value="misc" prepend-icon="mdi-shape" title="其他"></v-list-item>

        </v-list>
      </v-sheet>

      <v-sheet class="d-flex flex-column flex-grow-1 overflow-y-auto overflow-x-hidden" color="background" height="calc(100vh - 64px)">
        <div class="ma-2 pa-2">
          <template v-if="selectedTab[0] === 'groups'">
            <div class="d-flex flex-column justify-start align-center">
              <ConfigGroupCard v-for="group in nameTableConfig.students" :group></ConfigGroupCard>
              <!-- <v-expansion-panels multiple>
                <v-expansion-panel v-for="group in nameTableConfig.students">
                  <v-expansion-panel-title>
                    <template v-slot:default="{ expanded }">
                      {{ group.name }}
                      <v-btn icon="mdi-pencil" v-if="expanded" variant="text" size="small" class="mx-3"></v-btn>

                    </template>
</v-expansion-panel-title>
<v-expansion-panel-text>

</v-expansion-panel-text>
</v-expansion-panel>
</v-expansion-panels> -->
              <v-empty-state icon="mdi-traffic-cone" title="Under Construction"></v-empty-state>
            </div>
          </template>

          <template v-if="selectedTab[0] === 'recorditems'">
            <div class="d-flex flex-column justify-start align-center">
              <v-card v-for="item in nameTableConfig.recordItems" :color="item.color" class="ma-2 pa-2" rounded="lg"
                width="80%">
                <v-card-title class="d-flex flex-row justify-space-between">{{ item.name }} <v-icon
                    :icon="'mdi-' + item.icon" size="small" class="my-1"></v-icon></v-card-title>
              </v-card>
              <v-empty-state icon="mdi-traffic-cone" title="Under Construction"></v-empty-state>
            </div>

          </template>

          <template v-if="selectedTab[0] === 'misc'">
            <v-list lines="two">
              <v-list-item title="显示小组名称" density="compact"
                @click="nameTableConfig.preferences.showGroupNames = !nameTableConfig.preferences.showGroupNames">
                <template v-slot:append>
                  <v-list-item-action end><v-switch v-model="nameTableConfig.preferences.showGroupNames" hide-details
                      inset color="primary"></v-switch></v-list-item-action>
                </template>
              </v-list-item>
              <v-list-item title="居中显示内容" density="compact"
                @click="nameTableConfig.preferences.justifyCenter = !nameTableConfig.preferences.justifyCenter">
                <template v-slot:append>
                  <v-list-item-action end><v-switch v-model="nameTableConfig.preferences.justifyCenter" hide-details
                      inset color="primary"></v-switch></v-list-item-action>
                </template>
              </v-list-item>
              <v-divider></v-divider>
              <v-list-item title="导出配置" subtitle="导出配置至JSON文件" @click="saveConfig"></v-list-item>
              <v-list-item title="导入配置" subtitle="从JSON文件导入配置" @click="loadConfig"></v-list-item>
            </v-list>
          </template>
        </div>
      </v-sheet>
    </div>


  </v-card>

</template>

<script setup>
const emit = defineEmits(['close', 'submit'])

import { Store } from '@tauri-apps/plugin-store'
import { open, save } from '@tauri-apps/plugin-dialog'

import { invoke } from '@tauri-apps/api/core'
import { ref, onMounted } from 'vue'
import ConfigGroupCard from './ConfigGroupCard.vue';

let store = new Store('config.bin')
const prepared = ref(false)

const nameTableConfig = ref({})
const selectedTab = ref(['groups'])


const contentHeight = ref(0)
onMounted(() => {
  contentHeight.value = window.innerHeight - 64
  window.onresize = () => {
    contentHeight.value = window.innerHeight - 64
  }
})


async function saveData() {
  store.set('config', nameTableConfig.value)
  emit('submit')
}

async function saveConfig() {
  let saveFile = await save({
    filters: [
      {
        extensions: ["json"],
        name: "JSON"
      }
    ]
  })
  if (saveFile === null) return
  if (!saveFile.endsWith(".json")) {
    invoke('save_config', { path: saveFile + ".json", data: JSON.stringify(nameTableConfig.value) })
  } else {
    invoke('save_config', { path: saveFile, data: JSON.stringify(nameTableConfig.value) })
  }
}


async function loadConfig() {
  let openFile = await open({
    multiple: false,
    directory: false,
    filters: [
      {
        extensions: ["json"],
        name: "JSON"
      }
    ]
  })
  let config = await invoke('read_config', { path: openFile.path })
  nameTableConfig.value = JSON.parse(config)
  saveData()
}


prepareData()

async function prepareData() {
  let configStoreData = await store.get('config')
  nameTableConfig.value = configStoreData
  prepared.value = true
}

</script>
