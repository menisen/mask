<template>
  <input
    type="text"
    v-model="display"
    @input="onInput"
    ref="input"
  >
</template>

<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  value: String,
  mask: String
}>()
const emit = defineEmits(['update:value'])

let display = ref('')
let lastValue: string = ''

const tokens: { [key: string]: { pattern: RegExp, transform: (v: string) => string } } = { // Список токенов
  '#': { pattern: /[0-9]/, transform: v => v },
  'X': { pattern: /[0-9a-zA-Z]/, transform: v => v },
  'S': { pattern: /[a-zA-Z]/, transform: v => v },
  'A': { pattern: /[a-zA-Z]/, transform: v => v.toLocaleUpperCase() },
  'a': { pattern: /[a-zA-Z]/, transform: v => v.toLocaleLowerCase() },
}

const checkValidate = (value: string, token: RegExp | undefined): boolean => { // Чекаем на валидность на токена
  if (token) {
    const regExp = new RegExp(token)
    return regExp.test(value)
  } else return false
}
const transform = (position: number, char: string) => {
  let result = ''
  if (typeof tokens[props.mask.charAt(position)].transform !== "undefined") {
    result = tokens[props.mask.charAt(position)].transform(char)
  } else {
    result = char
  }
  return result
}


const onInput = (event: Event) => {
  const e = event as InputEvent
  // Сохраняем начальное и неизмененное значение
  let firstValue: string = display.value
  // Пропускать если нажата только символы и копипаст
  if (!(e.data || e.inputType === 'insertFromPaste')) {
    lastValue = firstValue
    return
  }
  // Пропускаем только цифры и латинские буквы
  if (checkValidate(e.data!, /[0-9a-zA-Z]/)) {
    display.value = lastValue
  }
  let changedPosition: number = 0
  let changedCount: number = 1
  // Если они не одинаковы сохраняем позиция до изменения
  if (lastValue !== firstValue) {
    changedPosition = lastValue?.length
    changedCount = firstValue?.length - lastValue?.length
  }
  // Недопускать чтобы не был лишним
  if (firstValue?.length > props.mask?.length) {
    display.value = display.value.slice(0, props.mask.length)
    return
  }

  let a: number = 0
  let b: number = 0
  let position = changedPosition || 0
  while((a < changedCount) && changedPosition + a < props.mask.length) {
    // Фиксируем символ нового введенного слово
    let char: string = firstValue.charAt(changedPosition + a)
    // Фиксируем текущий символ
    let symbol: string = props.mask.charAt(position)
    // Чекаем есть ли в списке токенов
    let token: RegExp | undefined = tokens[props.mask.charAt(position)]?.pattern
    // if (symbol === char) {
    //   lastValue += char
    //   a++
    // } else
    if (!tokens[symbol]) { // Если не нашли в списке токенов, то идем дальше
      lastValue += symbol
      b++
    } else {
      if (checkValidate(char, token)) { // Чекаем на валидность
        lastValue += transform(position, char)
      }
      a++ // Идем дальше по новодобавленным символам
    }
    // Изменяем значение input-а
    display.value = lastValue
    position++
  }
  emit('update:value', display.value)
}

</script>

<style scoped>
</style>
