<script setup lang="ts">
import BaseTextInput from './TextInput.vue'

interface Props {
  length?: 4 | 5 | 6 | 7 | 8,
  n?: number,
  disabled?: boolean,
  error?: boolean,
  errorMessage?: string,
}

export interface SingleInput {
  value: number | string,
  id: number,
}

const props = withDefaults(defineProps<Props>(), {
  length: 6,
  disabled: false,
  error: false,
  errorMessage: ''
})

const model = defineModel<Array<SingleInput>>({ default: () => [] })

watch(() => model.value, (newValue) => {
  if (newValue.length === 0) {
    model.value = Array.from({ length: props.length }, (_, i) => ({ value: '', id: i }))
  }
}, { immediate: true })

const inputRefs = ref<InstanceType<typeof BaseTextInput>[]>([])

const isComposing = ref(false)
const handleCompositionStart = () => {
  isComposing.value = true
}
const handleCompositionUpdate = (e: Event) => {
  const input = e.target as HTMLInputElement
  input.value = ''
}
const handleCompositionEnd = (e: Event, index: number) => {
  isComposing.value = false
  handleInput(e, index)
}
const handleKeydown = (e: KeyboardEvent, index: number) => {
  if (e.key === 'Backspace') {
    if (index > 0 && model.value[index].value === '') {
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

  digits.split('').forEach((digit, i) => {
    if (model.value[i]) {
      model.value[i].value = digit
    }
  })

  inputRefs.value[props.length - 1]?.focus()
}

const handleInput = (e: Event, index: number) => {
  const input = e.target as HTMLInputElement
  const filtered = input.value.replace(/\D/g, '')

  const target = model.value[index]
  if (target) {
    target.value = filtered
  }

  if (filtered.length >= 1 && index < props.length - 1) {
    inputRefs.value[index + 1]?.focus()
  }

}
</script>
<template>
  <div>
    <div class="flex justify-center gap-6">
      <div
        v-for="(item, index) in model"
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
          @compositionstart="handleCompositionStart()"
          @compositionupdate="handleCompositionUpdate($event)"
          @compositionend="handleCompositionEnd($event, index)"
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
