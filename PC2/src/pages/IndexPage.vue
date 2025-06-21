<template>
  <q-page class="flex flex-center column q-gutter-md">
    <img
      alt="Quasar logo"
      src="~assets/quasar-logo-vertical.svg"
      style="width: 200px; height: 200px"
    >
    <CurrencyConverter />
  </q-page>
</template>

<script setup>
import CurrencyConverter from 'src/components/CurrencyConverter.vue'
</script>
