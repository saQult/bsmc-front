<script setup lang="ts">
import { RecaptchaV2, useRecaptcha } from 'vue3-recaptcha-v2'

const props = defineProps<{
  modelValue?: string | null
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', token: string | null): void
  (e: 'update:widget-id', id: number): void
  (e: 'error'): void
  (e: 'expired'): void
  (e: 'loaded', response: unknown): void
}>()

const { handleGetResponse } = useRecaptcha()
const widgetId = ref<number | null>(null)

const handleWidgetId = (id: number) => {
  widgetId.value = id
  emit('update:widget-id', id)

  // Получаем токен сразу после ивента (если уже прошли капчу)
  const token = handleGetResponse(id)
  if (token) emit('update:modelValue', token)
}

const handleErrorCallback = () => {
  emit('error')
}

const handleExpiredCallback = () => {
  emit('expired')
  emit('update:modelValue', null)
}

const handleLoadCallback = (response: unknown) => {
  emit('loaded', response)
}
</script>

<template>
  <RecaptchaV2
      @widget-id="handleWidgetId"
      @error-callback="handleErrorCallback"
      @expired-callback="handleExpiredCallback"
      @load-callback="handleLoadCallback"
  />
</template>
