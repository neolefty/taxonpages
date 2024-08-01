<template>
  <VCard>
    <ClientOnly>
      <VSpinner
        v-if="isLoading"
        logo-class="w-8 h-8"
        legend=""
      />
      <VCardHeader>
        <h2 class="text-md">Collection Objects</h2>
        <PanelDropdown panel-key="panel:collection-objects" />
      </VCardHeader>
    </ClientOnly>
  </VCard>
</template>

<script setup>

const isLoading = ref(false)
</script>

