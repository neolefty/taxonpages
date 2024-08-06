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
      <h2 class="text-md">
        Inventory
        <span v-if="Array.isArray(inventory)"> [{{inventory.length}}]</span>
      </h2>
      <PanelDropdown panel-key="panel:inventory" />
    </VCardHeader>
    <VCardContent class="text-sm">
      <details class="pb-4">
        <summary class="cursor-pointer">
          Collection Objects
          <span v-if="Array.isArray(collectionObjects)"> [{{collectionObjects.length}}]</span>
        </summary>
        <p v-if="typeof collectionObjects === 'string'" v-html="collectionObjects"/>
        <ul v-else  class="tree ml-2">
          <li v-for="collectionObject in collectionObjects" :key="collectionObject?.id">
            <details>
              <summary class="cursor-pointer">
                {{ collectionObject.id }} — length {{JSON.stringify(collectionObject).length}}
              </summary>
              <ul class="m-2 ml-6 list-disc">
                <li v-for="entry in Object.entries(collectionObject)" :key="entry[0]">
                  <em>{{ entry[0] }}:</em> {{ entry[1] }}
                </li>
              </ul>
              <p v-html="JSON.stringify(collectionObject, null, 1)"/>
            </details>
          </li>
        </ul>
      </details>
      <h3>Inventory</h3>
      <p v-if="typeof inventory === 'string'" v-html="inventory"/>
      <ul v-else class="tree ml-2">
        <li v-for="inventoryItem in inventory" :key="inventoryItem.id" class="mt-1">
          <details>
            <summary class="cursor-pointer">
              {{ makeInventoryLabel(inventoryItem) }}
            </summary>
            <ul class="m-2 ml-6 list-disc">
              <li v-for="detail in makeInventoryDetails(inventoryItem)" v-html="detail" :key="detail?.id"/>
              <li>
                <details>
                  <summary class="cursor-pointer">Full Record ({{Object.keys(inventoryItem).length}} items)</summary>
                  <dl class="m-2 ml-6">
                    <template v-for="entry in Object.entries(inventoryItem)" :key="entry[0]">
                      <dt>{{ entry[0] }}</dt>
                      <dd class="ml-4 mb-2">{{ entry[1] }}</dd>
                    </template>
                  </dl>
                </details>
              </li>
            </ul>
          </details>
        </li>
      </ul>
    </VCardContent>
  </VCard>
</template>

<script setup>
import { ref, watch } from 'vue'
import PanelDropdown from '../PanelDropdown.vue'
import { useOtuPageRequest } from "@/modules/otus/helpers/useOtuPageRequest.js"
import TaxonWorks from "@/modules/otus/services/TaxonWorks.js"

const props = defineProps({
  otuId: {
    type: [String, Number],
    required: true
  }
})

const collectionObjects = ref("Loading...")
const inventory = ref("Loading...")
const isLoading = ref({collectionObjects: false, inventory: false})

/** Based on taxonpages-orthoptera PanelSpecimentRecords. */
function makeInventoryLabel(item) {
  return [
    item.catalogNumber,
    makeLocationDescription(item),
    item.year,
    // `length ${JSON.stringify(item).length}`,
  ].filter(Boolean).join("; ")
}

function makeInventoryDetails(item) {
  return [
    makeInventoryCollectionDate(item),
    item.recordedBy && `Recorded by ${item.recordedBy}`,
    makeIdentifiedByDesccription(item),
    item.georeferencedBy && `Georeferenced by ${item.georeferencedBy}${makeGeoreferenceUncertainty(item)}`,
    // CollectionObject #1234
    item.dwc_occurrence_object_id && `${item.dwc_occurrence_object_type} #${item.dwc_occurrence_object_id}`,
  ].filter(Boolean)
}

function makeLocationDescription(item) {
  return [
    item.country && item.country,
    item.stateProvince && item.stateProvince,
    item.county && `${item.county} County`,
  ].filter(Boolean).join(", ")
}

function makeIdentifiedByDesccription(item) {
  return [
    item.identifiedBy && `Identified by ${item.identifiedBy}`,
    item.dateIdentified,
  ].filter(Boolean).join(", ")
}

function makeInventoryCollectionDate(item) {
  if (!item.year) return null
  const date = new Date(item.year, item.month, item.day)
  return `Collected ${date.toLocaleDateString()}`
}

function makeGeoreferenceUncertainty(item) {
  return item.coordinateUncertaintyInMeters && ` to within ${item.coordinateUncertaintyInMeters} meters`
}

watch(
    () => props.otuId,
    async () => {
      if (!props.otuId) {
        collectionObjects.value = 'No OTU specified.'
        inventory.value = 'No OTU specified.'
        return
      }

      isLoading.value = {...isLoading.value, collectionObjects: true}
      useOtuPageRequest('panel:inventory', () =>
        TaxonWorks.getCollectionObjects(props.otuId)
      ).then(({data}) => {
        collectionObjects.value = data
        console.log('CollectionObjects', data)
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
        TaxonWorks.getInventoryDarwinCore(props.otuId)
      ).then(({data}) => {
        inventory.value = data
        console.log('Inventory (Darwin Core)', data)
      }).catch(
          e => inventory.value = `Error: ${e}`
      ).finally(() => isLoading.value = {...isLoading.value, inventory: false})
    },
    {immediate: true}
  )

</script>
