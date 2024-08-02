<template>
  <VCard>
    <ClientOnly>
      <VSpinner
          v-if="isLoading.collectionObjects || isLoading.inventory"
          logo-class="w-8 h-8"
          legend=""
      />
    </ClientOnly>
    <VCardHeader class="flex justify-between">
      <h2 class="text-md">Collection Objects</h2>
      <PanelDropdown panel-key="panel:collection-objects" />
    </VCardHeader>
    <VCardContent class="text-sm">
      <h3>Collection Objects</h3>
      <p v-html="collectionObjects"/>
      <h3>Inventory</h3>
      <p v-html="inventory"/>
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

const collectionObjects = ref("Loading...")
const inventory = ref("Loading...")
const isLoading = ref({collectionObjects: false, inventory: false})

watch(
    () => props.otuId,
    async () => {
      if (!props.otuId) {
        collectionObjects.value = 'No OTU specified.'
        inventory.value = 'No OTU specified.'
        return
      }

      isLoading.value = {...isLoading.value, collectionObjects: true}
      useOtuPageRequest('panel:collection-objects', () =>
        TaxonWorks.getCollectionObjects(props.otuId)
      ).then(({data}) => {
        collectionObjects.value = `Collection objects for ${props.otuId}: ${JSON.stringify(data)}`
        console.log({data})
      }).catch(
          e => collectionObjects.value = `Error: ${e}`
      ).finally(() => isLoading.value = {...isLoading.value, collectionObjects: false})
    },
    {immediate: true}
  )

watch(
    () => props.otuId,
    async () => {
      if (!props.otuId) {
        inventory.value = 'No OTU specified.'
        return
      }

      isLoading.value = {...isLoading.value, inventory: true}
      useOtuPageRequest('panel:inventory', () =>
        TaxonWorks.getInventory(props.otuId)
      ).then(({data}) => {
        inventory.value = `Inventory for ${props.otuId}: ${JSON.stringify(data)}`
        console.log({data})
      }).catch(
          e => inventory.value = `Error: ${e}`
      ).finally(() => isLoading.value = {...isLoading.value, inventory: false})
    },
    {immediate: true}
  )

</script>
