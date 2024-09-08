<template>
  <v-layout class="d-flex flex-column">
    <v-main>
      <template v-if="prepared">
        <v-empty-state v-if="groupsMap.size === 0" icon="mdi-cancel" title="无数据" text="请在右下角的”设置“中添加配置"></v-empty-state>
        <v-container>
          <v-row align="center" no-gutters v-for="groupId in groupsMap.keys()">
            <v-col cols="2" v-if="props.config.preferences.showGroupNames"> {{ groupsMap.get(groupId).name }} </v-col>
            <v-col v-for="studentId in groupsMap.get(groupId).members">
              <v-card class="pa-2 ma-2 flex-row justify-center" elevation=1 :key="studentId"
                :color="currentColor === 'exported' ? '' : (studentsMap.get(studentId)[currentColor] ? currentColor : '')"
                @click="toggleStatus(studentId)">
                {{ studentsMap.get(studentId).name }}
                <template v-for="recordItem in recordItemsMap.keys()">
                  <v-icon :icon="recordItemsMap.get(recordItem).icon" size="small" :color="recordItem"
                    v-if="studentsMap.get(studentId)[recordItem]"></v-icon>
                </template>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </template>


    </v-main>
    <v-sheet class="d-flex flex-row align-center justify-between rounded-t-lg position-fixed bottom-0" elevation="3"
      width="100%" height="64px">
      <v-radio-group inline v-model="currentColor" class="mx-3">
        <v-radio v-for="recordItem in recordItemsMap.keys()" :color="recordItem"
          :label="recordItemsMap.get(recordItem).name" :value="recordItem"></v-radio>
        <v-radio v-show="false" label="export" value='exported'></v-radio>
      </v-radio-group>

      <v-bottom-sheet>
        <template v-slot:activator="{ props }">
          <v-btn v-bind="props" text="导出数据" @click="exportToText" class="mx-3"></v-btn>
        </template>
        <v-card>
          <v-textarea v-model="exportTextContent" label="导出数据" no-resize class="mx-2 mt-2 pa-2"
            color="grey-darken-4"></v-textarea>
          <v-card-actions>
            <v-btn prepend-icon="mdi-content-copy" @click="copyText">复制文本</v-btn>
          </v-card-actions>
        </v-card>
      </v-bottom-sheet>
      <v-btn v-if="!needReloading" icon="mdi-cog" variant="text" class="mx-2" @click="settingsDialog = true"></v-btn>
      <v-btn v-if="needReloading" icon="mdi-restart" color="blue" class="mx-2" @click="reloadPage"></v-btn>
    </v-sheet>

  </v-layout>

  <v-snackbar v-model="textCopiedSnackbar" timeout=1500>
    文本已复制
  </v-snackbar>
  <v-snackbar v-model="needReloadingSnackbar" timeout="4000">设置已保存，重载页面后生效</v-snackbar>

  <v-dialog v-model="settingsDialog" transition="dialog-bottom-transition" fullscreen>
    <Config @close="settingsDialog = false"
      @submit="needReloading = true; settingsDialog = false; needReloadingSnackbar = true;"></Config>
  </v-dialog>

</template>

<script setup>
const props = defineProps({
  config: 'Object'
})

import { writeText } from '@tauri-apps/plugin-clipboard-manager'
import { ref } from 'vue'

const currentColor = ref('exported')
const exportTextContent = ref('')
const textCopiedSnackbar = ref(false)
const settingsDialog = ref(false)
const needReloading = ref(false)
const prepared = ref(false)
const needReloadingSnackbar = ref(false)

var groupsMap = ref(new Map())
var studentsMap = ref(new Map())
var recordItemsMap = ref(new Map())

prepareData()

function prepareData() {
  for (let groupIndex in props.config.students) {
    let members = [];
    for (let name of props.config.students[groupIndex].members) {
      let studentId = studentsMap.value.size + 1;
      studentsMap.value.set(studentId, { name, blue: false, orange: false, red: false });
      members.push(studentId);
    }
    groupsMap.value.set(groupIndex, { name: props.config.students[groupIndex].name, members });
  }
  for (let recordItem of props.config.recordItems) {
    recordItemsMap.value.set(recordItem.color, {
      name: recordItem.name,
      icon: 'mdi-' + recordItem.icon
    })
  }
  currentColor.value = recordItemsMap.value.keys().next().value
  prepared.value = true
}

function toggleStatus(studentId) {
  if (currentColor.value === 'exported') return
  let studentData = studentsMap.value.get(studentId)
  studentData[currentColor.value] = !studentData[currentColor.value]
  studentsMap.value.set(studentId, studentData)
}

function exportToText() {
  let texts = {}
  recordItemsMap.value.forEach(({ name }, color) => {
    texts[color] = name + '：'
  })
  studentsMap.value.forEach(studentData => {
    for (let color in texts) {
      if (studentData[color]) texts[color] += studentData.name + ' '
    }
  })
  let exportText = ''
  for (let color in texts) {
    exportText += texts[color] + '\n'
  }
  exportTextContent.value = exportText.slice(0, -1)
  currentColor.value = 'exported'
}

async function copyText() {
  await writeText(exportTextContent.value)
  snackbar.value = true
}

function reloadPage() {
  location.reload()
}
</script>
