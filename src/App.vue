<template>
  <v-app>
    <Nametable :config="nameTableConfig" v-if="prepared"></Nametable>
  </v-app>
</template>

<script setup>
import { Store } from '@tauri-apps/plugin-store'
import { ref } from 'vue'

let store = new Store('config.bin')
const prepared = ref(false)

const defaultConfig = {
  students: [],
  recordItems: [
    {
      color: 'blue',
      name: '迟到',
      icon: 'clock-alert'
    },
    {
      color: 'orange',
      name: '第一段早读',
      icon: 'medal'
    },
    {
      color: 'red',
      name: '第二段早读',
      icon: 'trophy'
    }
  ],
  preferences: {
    showGroupNames: true,
    justifyCenter: false
  }
}

const nameTableConfig = ref({})

loadData()

async function loadData() {
  let storeExist = await store.has('config')
  if (storeExist) {
    let configStoreData = await store.get('config')
    nameTableConfig.value = configStoreData
  } else {
    nameTableConfig.value = defaultConfig
    await store.set('config', defaultConfig)
    //await store.set('config', nameTableConfigOld.value)
    store.save()
  }
  prepared.value = true
}

//
</script>
