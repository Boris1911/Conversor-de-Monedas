<template>
  <q-card class="q-pa-md" style="max-width: 400px; margin: auto;">
    <q-card-section>
      <div class="text-h6">Conversor de Monedas</div>
    </q-card-section>
    <q-card-section>
      <q-input
        v-model.number="amount"
        label="Monto"
        type="number"
        min="0.01"
        :error="amountError"
        :error-message="amountErrorMsg"
        dense
        outlined
        class="q-mb-md"
      />
      <div class="row q-gutter-sm q-mb-md">
        <q-select
          v-model="fromCurrency"
          :options="currencyOptions"
          label="De"
          option-label="label"
          option-value="value"
          emit-value
          map-options
          dense
          outlined
          style="flex:1"
        />
        <q-btn icon="swap_horiz" flat round @click="swapCurrencies" :disable="!fromCurrency || !toCurrency" />
        <q-select
          v-model="toCurrency"
          :options="currencyOptions"
          label="A"
          option-label="label"
          option-value="value"
          emit-value
          map-options
          dense
          outlined
          style="flex:1"
        />
      </div>
      <q-btn label="Convertir" color="primary" @click="convert" class="full-width" />
      <q-banner v-if="error" class="q-mt-md bg-red-2 text-red-10">{{ error }}</q-banner>
      <q-banner v-if="result" class="q-mt-md bg-green-2 text-green-10">{{ result }}</q-banner>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const amount = ref(null);
const fromCurrency = ref(null);
const toCurrency = ref(null);
const currencyOptions = ref([]);
const error = ref('');
const result = ref('');
const amountError = ref(false);
const amountErrorMsg = ref('');

onMounted(async () => {
  try {
    const res = await fetch('https://api.frankfurter.app/currencies');
    const data = await res.json();
    currencyOptions.value = Object.entries(data).map(([value, label]) => ({ value, label: `${value} - ${label}` }));
  } catch {
    error.value = 'No se pudieron cargar las monedas.';
  }
});

function swapCurrencies() {
  const temp = fromCurrency.value;
  fromCurrency.value = toCurrency.value;
  toCurrency.value = temp;
  result.value = '';
  error.value = '';
}

async function convert() {
  error.value = '';
  result.value = '';
  amountError.value = false;
  amountErrorMsg.value = '';

  if (!amount.value || amount.value <= 0) {
    amountError.value = true;
    amountErrorMsg.value = 'El monto debe ser mayor que cero.';
    return;
  }
  if (!fromCurrency.value || !toCurrency.value) {
    error.value = 'Debes seleccionar ambas monedas.';
    return;
  }
  if (fromCurrency.value === toCurrency.value) {
    error.value = 'Las monedas deben ser diferentes.';
    return;
  }
  try {
    const res = await fetch(`https://api.frankfurter.app/latest?amount=${amount.value}&from=${fromCurrency.value}&to=${toCurrency.value}`);
    const data = await res.json();
    const rate = data.rates[toCurrency.value];
    result.value = `${amount.value} ${fromCurrency.value} equivalen a ${rate.toFixed(2)} ${toCurrency.value}`;
  } catch {
    error.value = 'No se pudo realizar la conversión.';
  }
}
</script>

<style scoped>
.full-width { width: 100%; }
</style>
