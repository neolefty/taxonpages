<template>
  <VCard>
    <ClientOnly>
      <VSpinner
        v-if="isLoading"
        logo-class="w-8 h-8"
        legend=""
      />
    </ClientOnly>
    <VCardHeader>
      <h2 class="text-md">Collection Objects</h2>
      <PanelDropdown panel-key="panel:collection-objects" />
    </VCardHeader>
    <VCardContent class="text-sm">
      <p>Placeholder.</p>
    </VCardContent>
  </VCard>
</template>

<script setup>
import { ref } from 'vue'
import PanelDropdown from '../PanelDropdown.vue'

const isLoading = ref(false)
</script>
