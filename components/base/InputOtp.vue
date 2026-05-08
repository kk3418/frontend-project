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
  n: 1,
  disabled: false,
  error: false,
  errorMessage: ''
})

const columnsPerRow = computed(() => Math.ceil(props.length / props.n))

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
  const input = e.target as HTMLInputElement
  handleInput(input.value, index)
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

const handleInput = (value: string | number, index: number) => {
  const filtered = String(value).replace(/\D/g, '')

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
    <div
      class="grid justify-center gap-4"
      :style="{ gridTemplateColumns: `repeat(${columnsPerRow}, 5rem)` }"
    >
      <div
        v-for="(item, index) in model"
        :key="item.id"
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
          @update:model-value="handleInput($event, index)"
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
