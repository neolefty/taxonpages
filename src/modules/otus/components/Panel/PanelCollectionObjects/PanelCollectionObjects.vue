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
import { useOtuPageRequest } from "@/modules/otus/helpers/useOtuPageRequest.js"
import TaxonWorks from "@/modules/otus/services/TaxonWorks.js"

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
      useOtuPageRequest('panel:collection-objects', () =>
        TaxonWorks.getCollectionObjects(props.otuId)
      ).then(({data}) => {
        content.value = `Collection objects for ${props.otuId}: ${JSON.stringify(data)}`
      }).catch(
          e => content.value = `Error: ${JSON.stringify(e)}`
      ).finally(() => isLoading.value = false)
    },
    {immediate: true}
)
</script>
