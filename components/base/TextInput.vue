<script setup lang="ts">
interface Props {
  placeholder?: string,
  type?: 'text' | 'account' | 'email',
  disabled?: boolean,
  maxLength?: number,
  inputmode?: 'text' | 'email' | 'search' | 'tel' | 'url' | 'none' | 'numeric' | 'decimal'
}

const model = defineModel<string | number>({ default: '' })
const props = withDefaults(defineProps<Props>(), {
  type: 'text',
  placeholder: '',
  disabled: false,
  maxLength: 1,
  inputmode: 'text'
})

const inputRef = ref<HTMLInputElement | null>(null)

defineExpose({
  focus: () => inputRef.value?.focus()
})
</script>

<template>
  <div class="relative w-full">
    <input
      ref="inputRef"
      v-model="model"
      :placeholder="props.placeholder"
      :type="props.type"
      :maxlength="props.maxLength"
      class="w-full rounded-full border-2 border-black bg-white px-3 py-2 text-sm text-black outline-0"
      :disabled="props.disabled"
      :inputmode="props.inputmode"
      autocomplete="off"
    >
  </div>
</template>
