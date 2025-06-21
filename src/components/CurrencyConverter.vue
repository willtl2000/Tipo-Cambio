<template>
  <q-card class="q-pa-lg q-mx-auto" style="max-width: 400px">
    <q-card-section>
      <div class="text-h6">Conversor de Monedas</div>
    </q-card-section>

    <q-card-section>
      <q-input
        filled
        v-model.number="amount"
        label="Monto a convertir"
        type="number"
        min="0"
        :error="amountError"
        :error-message="'Ingresa un monto válido'"
        class="q-mb-md"
      />

      <q-select
        filled
        v-model="from"
        :options="currenciesOptions"
        label="Moneda de Origen"
        emit-value
        map-options
        :error="currencyError && !from"
        class="q-mb-md"
      />

      <q-select
        filled
        v-model="to"
        :options="currenciesOptions"
        label="Moneda de Destino"
        emit-value
        map-options
        :error="currencyError && !to"
        class="q-mb-md"
      />

      <div class="q-gutter-sm row items-center q-mb-md">
        <q-btn
          icon="swap_horiz"
          color="primary"
          @click="swapCurrencies"
          round
          dense
          size="sm"
          :disable="!from || !to"
          title="Intercambiar monedas"
        />
        <q-btn
          label="Convertir"
          color="primary"
          @click="convertCurrency"
          class="q-ml-sm"
          :loading="loading"
        />
      </div>

      <q-banner v-if="result" class="bg-green-2 text-primary q-mb-md">
        {{ result }}
      </q-banner>
      <q-banner v-if="errorMessage" class="bg-red-2 text-negative q-mb-md">
        {{ errorMessage }}
      </q-banner>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const amount = ref(null)
const from = ref(null)
const to = ref(null)
const currenciesOptions = ref([])
const loading = ref(false)
const result = ref('')
const errorMessage = ref('')

const amountError = ref(false)
const currencyError = ref(false)

onMounted(async () => {
  await loadCurrencies()
})

// Cargar monedas desde la API
async function loadCurrencies() {
  try {
    const response = await fetch('https://api.frankfurter.app/currencies')
    const data = await response.json()
    currenciesOptions.value = Object.entries(data).map(([code, name]) => ({
      label: `${name} (${code})`,
      value: code,
    }))
  } catch {
    errorMessage.value = 'No se pudo cargar la lista de monedas.'
  }
}

function swapCurrencies() {
  const temp = from.value
  from.value = to.value
  to.value = temp
  result.value = ''
}

async function convertCurrency() {
  result.value = ''
  errorMessage.value = ''

  // Validaciones
  amountError.value = !(amount.value > 0)
  currencyError.value = !from.value || !to.value

  if (amountError.value || currencyError.value) {
    errorMessage.value = 'Completa correctamente el formulario.'
    return
  }

  loading.value = true
  try {
    const url = `https://api.frankfurter.app/latest?amount=${amount.value}&from=${from.value}&to=${to.value}`
    const response = await fetch(url)
    const data = await response.json()

    if (data && data.rates && data.rates[to.value] !== undefined) {
      result.value = `${amount.value} ${from.value} equivalen a ${data.rates[to.value]} ${to.value}`
    } else {
      errorMessage.value = 'No se pudo obtener el tipo de cambio.'
    }
  } catch {
    errorMessage.value = 'Error en la conversión. Inténtalo de nuevo.'
  } finally {
    loading.value = false
  }
}
</script>
