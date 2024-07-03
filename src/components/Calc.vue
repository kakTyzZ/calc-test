<script setup>
import { ref, reactive, onMounted, watch } from 'vue'

const currency = ref('RUB')
const allCurrencies = ref('')

const localState = reactive({
  payment: 'month',
  tariffPlan: 'Стандартный'
})

const planData = reactive({
  Стандартный: {
    month: 100,
    year: 1000,
    profit: 200
  },
  Продвинутый: {
    month: 150,
    year: 1400,
    profit: 400
  }
})

function calculateValues(amountOfRubles, currency) {
  return (
    (amountOfRubles / allCurrencies.value['eur']['rub']) *
    allCurrencies.value['eur'][currency]
  ).toFixed(1)
}

function calculateProfit() {
  return (
    planData[localState.tariffPlan]['month'] * 12 -
    planData[localState.tariffPlan]['year']
  ).toFixed(1)
}

function profitFunc() {
  planData.Стандартный['profit'] = calculateProfit()
  planData.Продвинутый['profit'] = calculateProfit()
}

watch(currency, () => {
  if (currency.value === 'RUB') {
    planData.Стандартный['month'] = 100
    planData.Стандартный['year'] = 1000
    planData.Стандартный['profit'] = 200
    planData.Продвинутый['month'] = 150
    planData.Продвинутый['year'] = 1400
    planData.Продвинутый['profit'] = 400
  } else if (currency.value === 'KZT') {
    planData.Стандартный['month'] = calculateValues(100, 'kzt')
    planData.Стандартный['year'] = calculateValues(1000, 'kzt')
    planData.Продвинутый['month'] = calculateValues(150, 'kzt')
    planData.Продвинутый['year'] = calculateValues(1400, 'kzt')
    profitFunc()
  } else if (currency.value === 'CNY') {
    planData.Стандартный['month'] = calculateValues(100, 'cny')
    planData.Стандартный['year'] = calculateValues(1000, 'cny')
    planData.Продвинутый['month'] = calculateValues(150, 'cny')
    planData.Продвинутый['year'] = calculateValues(1400, 'cny')
    profitFunc()
  }
})

watch(localState, () => {
  profitFunc()
})

onMounted(async () => {
  try {
    const response = await fetch('https://2024-03-06.currency-api.pages.dev/v1/currencies/eur.json')
    allCurrencies.value = await response.json()
    console.log('allCurrencies', allCurrencies.value['eur']['rub'].toFixed(1))
    console.log('allCurrencies', allCurrencies.value['eur']['kzt'])
    console.log('allCurrencies', allCurrencies.value['eur']['cny'])
  } catch (error) {
    console.error(error)
  }
})
</script>

<template>
  <section class="w-[450px] h-[500px] border rounded-xl shadow-xl p-4 text-xl">
    <form action="">
      <h1 class="flex justify-center text-xl pb-4 rounded">
        <strong>Тарифный калькулятор</strong>
      </h1>
      <div class="flex justify-between gap-2">
        <button
          @click.prevent="localState.tariffPlan = 'Стандартный'"
          class="shadow-xl p-2 w-full border-[3px] hover:bg-gray-300 rounded-xl"
        >
          <div>Стандартный</div>
          <div>Цена: {{ planData.Стандартный[localState['payment']] }}</div>
          <!-- <div v-if="localState.payment === 'year'">Цена: 1000</div> -->
        </button>
        <button
          @click.prevent="localState.tariffPlan = 'Продвинутый'"
          class="shadow-xl p-2 w-full border-[3px] hover:bg-gray-300 rounded-xl"
        >
          <div>Продвинутый</div>
          <div>Цена: {{ planData.Продвинутый[localState['payment']] }}</div>
        </button>
      </div>

      <fieldset
        class="flex justify-around text-xl py-4 gap-8 shadow-xl border-[3px] my-3 rounded-xl"
      >
        <div>
          <input
            name="currency"
            v-model="currency"
            class="cursor-pointer"
            type="radio"
            id="CNY"
            value="CNY"
          />
          <label class="cursor-pointer" for="CNY">Юань</label>
        </div>
        <div>
          <input
            name="currency"
            v-model="currency"
            class="cursor-pointer"
            type="radio"
            id="KZT"
            value="KZT"
          />
          <label class="cursor-pointer" for="KZT">Тенге</label>
        </div>
        <div>
          <input
            name="currency"
            v-model="currency"
            class="cursor-pointer"
            type="radio"
            id="RUB"
            value="RUB"
          />
          <label class="cursor-pointer" for="RUB">Рубли</label>
        </div>
      </fieldset>
      <section class="flex justify-between text-center my-3">
        <div
          v-if="localState.payment === 'month'"
          class="flex items-center text-center gap-1 w-full"
        >
          <button
            @click="localState.payment = 'year'"
            class="rounded-xl w-full py-2 hover:bg-gray-200 border-[5px]"
          >
            Цена Год
          </button>
        </div>
        <div
          v-if="localState.payment === 'year'"
          class="flex items-center text-center gap-1 w-full"
        >
          <button
            @click="localState.payment = 'month'"
            class="rounded-xl w-full border-[5px] py-2 hover:bg-gray-200"
          >
            Цена за Месяц
          </button>
        </div>
      </section>

      <section
        class="flex flex-col justify-center items-center gap-2 shadow-xl bg-green-100 rounded"
      >
        <div>
          Выбранный тариф - <span>{{ localState.tariffPlan }}</span>
        </div>
        <div>Валюта - {{ currency }}</div>
        <div v-if="allCurrencies !== ''">
          Цена:
          <span
            >{{ planData[localState.tariffPlan][localState.payment] }} за
            {{ localState.payment === 'year' ? 'год' : 'месяц' }}</span
          >
        </div>
        <div>
          Экономия:
          <span v-if="localState.payment === 'year'">{{
            planData[localState.tariffPlan].profit
          }}</span>
          <span v-if="localState.payment === 'month'">0</span>
        </div>
      </section>
    </form>
  </section>
</template>
