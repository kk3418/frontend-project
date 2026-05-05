<script setup lang="ts">
import BaseTextInput from './TextInput.vue'

interface Props {
  length?: number,
}

interface SingleInput {
  value: number | string,
  id: number,
}

const props = withDefaults(defineProps<Props>(), {
  length: 6
})
// const otpNumber = defineModel<string | number>({ default: '' })

const optNumbers = reactive<Array<SingleInput>>([])
const inputRefs = ref<InstanceType<typeof BaseTextInput>[]>([])

for (let i = 0; i < props.length; i++) {
  optNumbers.push({
    value: '',
    id: i
  })
}

const handleInput = (e: Event, index: number) => {
  const input = e.target as HTMLInputElement
  const filtered = input.value.replace(/\D/g, '')

  const target = optNumbers[index]
  if (target) {
    target.value = filtered
  }

  if (filtered.length >= 1 && index < props.length - 1) {
    inputRefs.value[index + 1]?.focus()
  }
}
</script>
<template>
  <div class="flex gap-6 p-6">
    <div
      v-for="(item, index) in optNumbers"
      :key="item.id"
      class="w-20"
    >
      <BaseTextInput
        :ref="(el) => inputRefs[index] = el as InstanceType<typeof BaseTextInput>"
        v-model="item.value"
        class="text-center"
        :max-length="1"
        inputmode="numeric"
        @input="handleInput($event, index)"
      />
    </div>
  </div>
</template>
