<template>
  <v-card width="80%" class="ma-2 pa-2">
    <v-card-title class="d-flex flex-row justify-space-between align-center">
      <span>{{ group.name }}</span>
      <v-btn variant="text" :icon="expanded ? 'mdi-chevron-up' : 'mdi-chevron-down'" @click="expanded = !expanded"></v-btn>
    </v-card-title>
    <v-card-text v-show="expanded">
      <template v-for="student in group.members">{{ student + ' ' }}</template>
    </v-card-text>

  </v-card>
</template>

<script setup>
const props = defineProps({
  group: 'object'
})
import { ref } from 'vue'

const expanded = ref(false)



</script>