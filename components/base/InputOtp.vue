<script setup lang="ts">
import BaseTextInput from './TextInput.vue'

interface Props {
  length?: 4 | 5 | 6 | 7 | 8,
  disabled?: boolean,
  error?: boolean,
  errorMessage?: string,
}

interface SingleInput {
  value: number | string,
  id: number,
}

const props = withDefaults(defineProps<Props>(), {
  length: 6,
  disabled: false,
  error: false,
  errorMessage: ''
})

const emit = defineEmits<{
  complete: [value: string]
}>()

const optNumbers = reactive<Array<SingleInput>>([])
const inputRefs = ref<InstanceType<typeof BaseTextInput>[]>([])

for (let i = 0; i < props.length; i++) {
  optNumbers.push({
    value: '',
    id: i
  })
}

const checkComplete = () => {
  const allFilled = optNumbers.every(o => o.value !== '')
  if (allFilled) {
    emit('complete', optNumbers.map(o => o.value).join(''))
  }
}

const handleKeydown = (e: KeyboardEvent, index: number) => {
  if (e.key === 'Backspace') {
    e.preventDefault()
    if (optNumbers[index].value !== '') {
      optNumbers[index].value = ''
    }
    if (index > 0) {
      inputRefs.value[index - 1]?.focus()
    }
  }
}

const handlePaste = (e: ClipboardEvent) => {
  e.preventDefault()
  const digits = (e.clipboardData?.getData('text') ?? '').replace(/\D/g, '')
  if (!digits || digits.length !== props.length) {
    return
  }

  let currentIndex = 0
  digits.split('').forEach((digit) => {
    optNumbers[currentIndex].value = digit
    currentIndex += 1
  })

  inputRefs.value[currentIndex]?.focus()
  checkComplete()
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

  checkComplete()
}
</script>
<template>
  <div>
    <div class="flex justify-center gap-6">
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
          :disabled="props.disabled"
          :error="props.error"
          @keydown="handleKeydown($event, index)"
          @paste="handlePaste($event)"
          @input="handleInput($event, index)"
        />
      </div>
    </div>
    <p
      v-if="props.error && props.errorMessage"
      class="mt-2 text-center text-sm text-red-500"
    >
      {{ props.errorMessage }}
    </p>
  </div>
</template>
