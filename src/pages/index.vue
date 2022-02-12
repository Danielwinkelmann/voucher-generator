<template>
  <div>
    <h1
      class="font-extrabold text-8xl text-transparent bg-clip-text bg-gradient-to-br from-purple-400 to-red-500"
    >
      Voucher Generator
    </h1>
    <ul>
      <li>combinationPossible: {{ combinationPossible }}</li>
      <li>customCharSet: <input v-model="customCharSet" type="text"></li>
      <li>customPattern: <input v-model="customPattern" type="text"></li>
      <li>replaceCharacter: <input v-model="replaceCharacter" type="text"></li>
      <li>count: <input v-model="count" type="number"></li>
      <li>characters: {{ characters }}</li>
      <li>individualCharacterCount: {{ individualCharacterCount }}</li>
      <li>generateCharacterCount: {{ generateCharacterCount }}</li>
      <li>possibleCombinations: {{ possibleCombinations }}</li>
      <li>useNumbers: <button @click="useNumbers = !useNumbers" v-text="useNumbers ? 'true' : 'false'" /></li>
      <li>useUppercase: <button @click="useUppercase = !useUppercase" v-text="useUppercase ? 'true' : 'false'" /></li>
      <li>useLowercase: <button @click="useLowercase = !useLowercase" v-text="useLowercase ? 'true' : 'false'" /></li>
      <li>useSymbols: <button @click="useSymbols = !useSymbols" v-text="useSymbols ? 'true' : 'false'" /></li>
      <li>removeSimilar: <button @click="removeSimilar = !removeSimilar" v-text="removeSimilar ? 'true' : 'false'" /></li>
      <li>charSetNumber: {{ charSetNumber }}</li>
      <li>charSetSymbol: {{ charSetSymbol }}</li>
      <li>charSetUppercase: {{ charSetUppercase }}</li>
      <li>charSetLowercase: {{ charSetLowercase }}</li>
      <li>charSetSimilar: {{ charSetSimilar }}</li>
      <li>pending: {{ pending }}</li>
      <li>
        <button class="bg-teal-400 hover:bg-teal-500 text-teal-100 rounded-lg px-2 py-1" @click="generateVouchers">
          generate
        </button>
      </li>
    </ul>
    <ul class="grid grid-cols-5 gap-4 mt-8">
      <li v-for="voucher of vouchers" :key="voucher" class="bg-slate-300 p-2 rounded-lg font-bold">
        {{ voucher }}
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import type { Ref } from 'vue'

const customCharSet = ref('')
const customPattern = ref('Test-####')
const pending = ref(false)
const replaceCharacter = ref('#')
const count = ref(10)
const limit = ref(9999)
const vouchers: Ref<string[]> = ref([])
const useNumbers = ref(false)
const useUppercase = ref(false)
const useLowercase = ref(false)
const useSymbols = ref(false)
const removeSimilar = ref(false)
const charSetNumber = '0123456789'
const charSetSymbol = '@#$%-'
const charSetUppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ' // without similar
const charSetLowercase = 'abcdefghijklmnopqrstuvwxyz' // without similar
const charSetSimilar = 'Oo0iIl'

const characters = computed(() => {
  const similarRegExp = new RegExp(`[${charSetSimilar}]`, 'gm')
  if (customCharSet.value) return customCharSet.value.replace(/\s/g, '')
  const set = []
  if (useNumbers.value) set.push(charSetNumber)
  if (useUppercase.value) set.push(charSetUppercase)
  if (useLowercase.value) set.push(charSetLowercase)
  if (useSymbols.value) set.push(charSetSymbol)
  const characters = set.length === 0 ? charSetUppercase : set.join('')
  return removeSimilar.value ? set.length > 1 ? characters.replace(similarRegExp, '') : characters : characters
})

const generateRandomCryptoNumber = () => {
  if (window)
    return crypto.getRandomValues(new Uint32Array(1))[0] / 4294967296
  else
    return Math.random() // to avoid SSR Issues
}

const getRandomNumber = (from: number, to: number) => Math.round(generateRandomCryptoNumber() * to) + from
const getRandomCharacter = () => characters.value[getRandomNumber(0, characters.value.length - 1)]

const generateVoucher = () => {
  const charArray = []
  for (const character of customPattern.value) {
    if (character === replaceCharacter.value)
      charArray.push(getRandomCharacter())
    else
      charArray.push(character)
  }
  return charArray.join('')
}

const generateCharacterCount = computed(() => customPattern.value.split('').filter(e => e === replaceCharacter.value).length)

const individualCharacterCount = computed(() => {
  const set = new Set()
  for (const value of characters.value) set.add(value)
  return set.size
})

const possibleCombinations = computed(() => Math.pow(individualCharacterCount.value, generateCharacterCount.value))
const combinationPossible = computed(() => possibleCombinations.value >= count.value && count.value <= limit.value)
const generateVouchers = () => {
  vouchers.value = []
  if (!combinationPossible.value) return
  pending.value = true
  if (customPattern.value) {
    while (count.value > vouchers.value.length) {
      const voucher = generateVoucher()
      if (!vouchers.value.includes(voucher)) vouchers.value.push(voucher)
    }
    pending.value = false
  }
}

</script>

<route lang="yaml">
meta:
  layout: default
</route>
