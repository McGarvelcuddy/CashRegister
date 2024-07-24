<script setup>
import { ref, watch, defineEmits } from 'vue'

const checkedOptions = ref([])


const emit = defineEmits(['update:checkedOptions'])

watch(checkedOptions, (newValue) => {
  emit('update:checkedOptions', newValue)
})
</script>

<template>
  <div>Selected Options: {{ checkedOptions }}</div>

  <input type="checkbox" id="divisibleThree" value="/3" v-model="checkedOptions" />
  <label for="divisibleThree">Divisible by 3 = Random</label>

  <input type="checkbox" id="john" value="John" v-model="checkedOptions" />
  <label for="john">John</label>

  <input type="checkbox" id="mike" value="Mike" v-model="checkedOptions" />
  <label for="mike">Mike</label>
</template>