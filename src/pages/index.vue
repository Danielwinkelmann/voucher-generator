<template>
  <div>
    <h1
      class="font-extrabold text-5xl sm:text-8xl text-transparent bg-clip-text tracking-tighter hover:tracking-tight transition-all duration-1000 bg-gradient-to-br to-blue-500 via-purple-500 from-red-500 bg-size-200 bg-pos-0 hover:bg-pos-100"
    >
      Voucher Generator
    </h1>
    <div class="flex justify-center w-full space-x-4 mt-4 mx-auto">
      <button
        class="bg-teal-400 hover:bg-teal-500 text-teal-100 rounded-lg px-2 py-1"
        @click="generateVouchers"
      >
        generate
      </button>
      <button
        class="bg-red-400 hover:bg-red-500 text-teal-100 rounded-lg px-2 py-1"
        @click="vouchers = []"
      >
        clear
      </button>
    </div>
    <!--   <ul>
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
    </ul> -->
    <ul class="grid grid-cols-5 gap-4 mt-8">
      <li
        v-for="voucher of vouchers"
        :key="voucher"
        class="bg-slate-300 p-2 rounded-lg font-bold"
      >
        {{ voucher }}
      </li>
    </ul>
    <ul>
      <li>count: <input v-model="count" type="number"></li>
      <li>useNumbers: <button @click="useNumbers = !useNumbers" v-text="useNumbers ? 'true' : 'false'" /></li>
      <li>useUppercase: <button @click="useUppercase = !useUppercase" v-text="useUppercase ? 'true' : 'false'" /></li>
      <li>useLowercase: <button @click="useLowercase = !useLowercase" v-text="useLowercase ? 'true' : 'false'" /></li>
      <li>useSymbols: <button @click="useSymbols = !useSymbols" v-text="useSymbols ? 'true' : 'false'" /></li>
      <li>removeSimilar: <button @click="removeSimilar = !removeSimilar" v-text="removeSimilar ? 'true' : 'false'" /></li>
    </ul>
    <pre>{{ blackList }}</pre>
    <input v-model="separatedSymbol" type="text">
    <textarea v-model="blackListTextArea" rows="5" type="text" />
    <pre>generatedPattern:{{ customPattern }}</pre>
    <input accept=".csv" type="file" @change="onFileSelect">
    <button @click="clipboard()">
      Copy
    </button>
  </div>
</template>

<script setup lang="ts">
import type { Ref } from 'vue'
const { copy } = useClipboard()

const blackListTextArea = ref('')
const separatedSymbol = ref('\n')
const blackList: Ref<string[]> = ref([])
const customCharSet = ref('')
const customPattern = ref('#####')
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

const clipboard = () => copy(vouchers.value.join(separatedSymbol.value))
watch(blackListTextArea, () => blackList.value = blackListTextArea.value.split(separatedSymbol.value))
const onFileSelect = (event: any) => {
  const reader = new FileReader()
  const files: FileList = event.target.files
  if (files.length > 0) {
    const file: File = files[0]
    reader.readAsBinaryString(file)
    reader.onload = function(e) {
      const data: string[] = (e!.target!.result! as string).split('\n')
      blackListTextArea.value = data.slice(data[0].length === data[1].length ? 0 : 1, data.length - 1).join('\n')
    }
  }
}
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

const generatePatternMatch = () => {
  const pattern: string[] = []
  const usedCharSet = new Set<string>()
  if (blackList.value.length > 1) {
    const firstEntry = blackList.value[0]
    const length = firstEntry.length

    for (let index = 0; index < length; index++) {
      const set = new Set<string>()
      for (const word of blackList.value) {
        if (word.length === length)
          set.add(word[index])
      }
      if (set.size === 1) {
        const [first] = set
        pattern.push(first)
      }
      else if (set.size > 1) {
        pattern.push(replaceCharacter.value)
        set.forEach(c => usedCharSet.add(c))
      }
    }
  }
  customPattern.value = pattern.join('')
  const regExp = (set: string) => new RegExp(`[${set}]`, 'gm')
  const usedCharSetString = Array.from(usedCharSet).join('')
  useNumbers.value = !!usedCharSetString.match(regExp(charSetNumber))
  useUppercase.value = !!usedCharSetString.match(regExp(charSetUppercase))
  useLowercase.value = !!usedCharSetString.match(regExp(charSetLowercase))
  useSymbols.value = !!usedCharSetString.match(regExp(charSetSymbol))
  removeSimilar.value = !usedCharSetString.match(regExp(charSetSimilar))
}
watch(blackList, () => generatePatternMatch())
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
      if (!vouchers.value.includes(voucher) && !blackList.value.includes(voucher)) vouchers.value.push(voucher)
    }
    pending.value = false
  }
}

</script>

<route lang="yaml">
meta:
  layout: default
</route>
