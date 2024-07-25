<script lang="ts" setup>
import { ref, watch, defineEmits } from 'vue'

const checkedOptions = ref<string[]>([])
const divisibleBy = ref<number | null>(3)
const divisionCheckbox = ref<HTMLInputElement | null>()

const emit = defineEmits(['update:checkedOptions', 'update:divisibleBy'])

watch(checkedOptions, (newValue) => {
  emit('update:checkedOptions', newValue)
})

watch(divisibleBy, (newValue) => {
  if (newValue === 0 || newValue === null || isNaN(newValue)) {
    if (divisionCheckbox.value) {
      divisionCheckbox.value.checked = false
      divisionCheckbox.value.disabled = true
    }
  } else {
    if (divisionCheckbox.value) {
      divisionCheckbox.value.disabled = false
    }
  }
  emit('update:divisibleBy', newValue)
})

function handleInput(event: Event) {
  const target = event.target as HTMLInputElement
  const value = target.value.trim()
  if (value === '') {
    divisibleBy.value = null
  } else {
    const parsedValue = parseInt(value, 10)
    divisibleBy.value = isNaN(parsedValue) ? null : parsedValue
  }
}
</script>

<template>
  <div class="option">
    <div>
    <input type="checkbox" id="division" value='÷x' v-model="checkedOptions" ref="divisionCheckbox" />
    <label for="division">Divisible By x = Random Amounts</label>
  </div>
  <div>
    <label for="x">x=</label>
    <input id="x" type="text" :value="divisibleBy !== null ? divisibleBy : ''" :style="{ width: '32px'}" @input="handleInput" />
  </div>
    <!-- More options could be added here -->
  </div>
</template>

<style scoped>
.option {
  display: flex;
  align-items: center;
  justify-content: center;
  border: solid;
  padding: 8px;
  column-gap: 2rem;
}
</style>