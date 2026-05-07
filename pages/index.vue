<script setup lang="ts">
import type { SingleInput } from '~/components/base/InputOtp.vue'

interface optRes {
  success: boolean,
  data?: {
    verified: boolean,
    timestamp: string,
  }
}

const otp = ref<SingleInput[]>([])
const error = ref(false)
const OTP_LENGTH = 6

const submit = async (value: SingleInput[]) => {
  const otpValue = value.map(i => i.value).join('')

  const res = await $fetch<optRes>('/api/examples/verify-otp-simple', {
    method: 'POST',
    body: { otp: otpValue }
  })

  if (res.success && res.data?.verified) {
    error.value = false
  } else {
    error.value = true
  }
}
</script>

<template>
  <div class="grid h-screen w-screen place-content-center">
    <BaseInputOtp
      v-model="otp"
      :length="OTP_LENGTH"
      :error="error"
      error-message="Invalid Code"
      @complete="submit"
    />
    <div class="flex justify-center">
      <button
        class="mt-4 w-20 rounded-full border-2 border-black bg-white px-4 py-2 text-sm text-black outline-0"
        @click="submit(otp)"
      >
        Submit
      </button>
    </div>
  </div>
</template>
