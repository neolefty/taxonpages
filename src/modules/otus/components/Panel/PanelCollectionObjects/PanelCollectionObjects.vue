<template>
  <VCard>
    <ClientOnly>
      <VSpinner
          v-if="isLoading"
          logo-class="w-8 h-8"
          legend=""
      />
    </ClientOnly>
    <VCardHeader class="flex justify-between">
      <h2 class="text-md">Collection Objects</h2>
      <PanelDropdown panel-key="panel:collection-objects" />
    </VCardHeader>
    <VCardContent class="text-sm">
      <p v-html="content"/>
    </VCardContent>
  </VCard>
</template>

<script setup>
import { ref, watch } from 'vue'
import PanelDropdown from '../PanelDropdown.vue'

const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms))

const props = defineProps({
  otuId: {
    type: [String, Number],
    required: true
  }
})

const content = ref("Loading...")
const isLoading = ref(false)

watch(
    () => props.otuId,
    async () => {
      if (!props.otuId) {
        content.value = 'No OTU specified.'
        return
      }

      isLoading.value = true
      await sleep(2000)
      content.value = `Collection objects content for ${props.otuId}.`
      isLoading.value = false
    },
    {immediate: true}
)
</script>
