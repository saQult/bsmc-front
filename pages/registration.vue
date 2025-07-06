<script setup lang="ts">
import * as v from 'valibot'
import type { FormSubmitEvent } from '@nuxt/ui'
import { RecaptchaV2, useRecaptcha  } from 'vue3-recaptcha-v2'

const state = reactive({
  username: '',
  email: '',
  password: '',
  repeatedPassword: '',
})
const widgetId = ref<number | null>(null)

const { handleGetResponse, handleReset } = useRecaptcha();
const show = ref(false)
const showRepeated = ref(false)


const schema = v.object({
  username: v.pipe(v.string(),
      v.nonEmpty("Введите никнейм"),
      v.minLength(4, 'Никнейм должен больше 3 символов')),
  email: v.pipe(v.string(), v.email('Неверная электронная почта'), v.nonEmpty("Введите электронную почту")),
  password: v.pipe(
      v.string(),
      v.nonEmpty("Введите пароль"),
      v.minLength(8, 'Пароль должен быть минимум 8 символов')),
  repeatedPassword: v.pipe(
      v.string(),
      v.nonEmpty("Введите пароль"),
      v.minLength(8, 'Пароль должен быть минимум 8 символов'))
  })

type Schema = v.InferOutput<typeof schema>


const handleWidgetId = (id: number) => {
  widgetId.value = id;

  handleGetResponse(id);
};
const handleErrorCallback = () => {

};
const handleExpiredCallback = () => {

};
const handleLoadCallback = (response: unknown) => {

};
const ready = ref(false);
onMounted(() => {ready.value = true})
const toast = useToast()
async function onSubmit(event: FormSubmitEvent<Schema>) {
  if (event.data.password !== event.data.repeatedPassword) {
    toast.add({ title: 'Ошибка!', description: 'Пароли не совпадают.', color: 'error' });
    return;
  }

  if (widgetId.value === null) {
    toast.add({ title: 'Ошибка!', description: 'Капча не загружена.', color: 'error' });
    return;
  }

  const token = handleGetResponse(widgetId.value);

  if (!token) {
    toast.add({ title: 'Ошибка!', description: 'Подтвердите, что вы не робот.', color: 'error' });
    return;
  }

  try {
    const response = await fetch('https://backend.mrekk.ru/api/users/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json', // Указываем, что отправляем JSON
      },
      body: JSON.stringify({
        username: event.data.username,
        email: event.data.email,
        password: event.data.password,
        captchaToken: token
      }),
    });

    if (response.ok) {
      const data = await response.json();
      toast.add({ title: 'Успех!', description: 'Регистрация прошла успешно.', color: 'success' });
      return navigateTo('/login');
    } else {
      const errorData = await response.json();
      toast.add({ title: 'Ошибка!', description: errorData.message || 'Ошибка при регистрации, возможно такие никнейм или почта уже заняты', color: 'error' });
      handleReset(widgetId.value);
    }
  } catch (error) {
    toast.add({ title: 'Ошибка!', description: 'Произошла ошибка при отправке запроса', color: 'error' });
    handleReset(widgetId.value);
  }
}
onMounted(async () => {
  const token = localStorage.getItem('token');
  if(!token)
    return;
  const response = await fetch('https://backend.mrekk.ru/api/users/get', {
    method: 'GET',
    headers: {'token': token},
  });
  if(response.ok) {
    return navigateTo('/cabinet');
  }
})
</script>

<template>
  <main class="bg">
    <div class="flex items-center justify-center min-h-screen">
      <UForm
          :schema="schema"
          :state="state"
          class="space-y-4 glass max-w-sm px-5 py-5 rounded-xl w-[350px]"
          @submit="onSubmit"
      >
        <h3 class="font-bold text-lg">Регистрация</h3>
        <UFormField name="username">
          <UInput v-model="state.username"
                  variant="soft"
                  color="primary"
                  placeholder="Никнейм"
                  icon="i-lucide-user-round"
                  class="w-[300px]"
                  :ui="{base: 'bg-white/80 text-stone-700'}"
          />
        </UFormField>
        <UFormField name="email">
          <UInput v-model="state.email"
                  variant="soft"
                  color="primary"
                  placeholder="Электронная почта"
                  icon="i-lucide-at-sign"
                  class="w-[300px]"
                  :ui="{base: 'bg-white/80 text-stone-700'}"
          />
        </UFormField>

        <UFormField name="password">
          <UInput v-model="state.password"
                  variant="soft"
                  color="neutral"
                  placeholder="Пароль"
                  icon="i-lucide-lock"
                  :type="show ? 'text' : 'password'"
                  class="w-[300px]"
                  :ui="{base: 'bg-white/80 text-stone-700',
                      trailing: 'pe-1'}"
          >
            <template #trailing>
              <UButton
                  color="neutral"
                  variant="link"
                  size="sm"
                  :icon="show ? 'i-lucide-eye-off' : 'i-lucide-eye'"
                  :aria-label="show ? 'Hide password' : 'Show password'"
                  :aria-pressed="show"
                  aria-controls="password"
                  @click="show = !show"
              />
            </template>
          </UInput>
        </UFormField>
        <UFormField name="repeatedPassword">
          <UInput v-model="state.repeatedPassword"
                  variant="soft"
                  color="neutral"
                  placeholder="Повторите пароль"
                  icon="i-lucide-lock"
                  :type="showRepeated ? 'text' : 'password'"
                  class="w-[300px]"
                  :ui="{base: 'bg-white/80 text-stone-700',
                      trailing: 'pe-1'}"
          >
            <template #trailing>
              <UButton
                  color="neutral"
                  variant="link"
                  size="sm"
                  :icon="showRepeated ? 'i-lucide-eye-off' : 'i-lucide-eye'"
                  :aria-label="showRepeated ? 'Hide password' : 'Show password'"
                  :aria-pressed="showRepeated"
                  aria-controls="password"
                  @click="showRepeated = !showRepeated"
              />
            </template>
          </UInput>
        </UFormField>
        <UButton type="submit" class="btn bg-teal-600/50 cursor-pointer w-full text-center justify-center hover:bg-teal-600">
          Зарегистрироваться
        </UButton>
        <UFormField>
          <RecaptchaV2
              @widget-id="handleWidgetId"
              @error-callback="handleErrorCallback"
              @expired-callback="handleExpiredCallback"
              @load-callback="handleLoadCallback"
          />
        </UFormField>
      </UForm>
    </div>
  </main>
</template>



<style>
.bg {
  background-image: url("public/images/bg/login.jpg");
  background-size: cover;
}
</style>

