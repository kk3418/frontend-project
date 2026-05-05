<script setup lang="ts">
interface optRes {
  success: boolean,
  data?: {
    verified: boolean,
    timestamp: string,
  }
}

const error = ref(false)

const submit = async (value: string) => {
  const res = await $fetch<optRes>('/api/examples/verify-otp-simple', {
    method: 'POST',
    body: { otp: value }
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
      :length="6"
      :error="error"
      error-message="Invalid Code"
      @complete="submit"
    />
    <div class="flex justify-center">
      <button class="mt-4 w-20 rounded-full border-2 border-black bg-white px-4 py-2 text-sm text-black outline-0">
        Submit
      </button>
    </div>
  </div>
</template>
