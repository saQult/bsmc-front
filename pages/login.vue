<script setup lang="ts">
import * as v from 'valibot';
import { RecaptchaV2, useRecaptcha } from 'vue3-recaptcha-v2';
import type { FormSubmitEvent } from '@nuxt/ui';
import { ref, reactive } from 'vue';
import router from "#app/plugins/router";

const { handleGetResponse, handleReset } = useRecaptcha();
const show = ref(false);

const schema = v.object({
  email: v.pipe(v.string(), v.email('Неверная электронная почта')),
  password: v.pipe(v.string(), v.minLength(8, 'Пароль должен быть минимум 8 символов')),
});

type Schema = v.InferOutput<typeof schema>;

const state = reactive({
  email: '',
  password: '',
});

const widgetId = ref<number | null>(null);

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

const toast = useToast()

async function onSubmit(event: FormSubmitEvent<Schema>) {
  const token = handleGetResponse(widgetId.value)

  if (!token) {
    toast.add({ title: 'Ошибка!', description: 'Подтвердите, что вы не робот.', color: 'error' })
    return
  }
  try {
    const response = await fetch('https://backend.mrekk.ru/api/users/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json', // Указываем тип данных
      },
      body: JSON.stringify({
        email: state.email,
        password: state.password,
        captchaToken: token,
      }),
    });
    if(response.ok) {
      const data:any = await response.json();

      localStorage.setItem('token', data.token);
      localStorage.setItem('username', data.username);
      toast.add({ title: 'Успешный вход!' , color: 'success' });
      return navigateTo('/cabinet');
    }
    else {
      toast.add({ title: 'Ошибка!', description:'Неправильный логин или пароль', color: 'error' });
    }
  } catch (error) {
    console.log(error);
  }
  finally {
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
          class="space-y-4 glass max-w-sm px-5 py-5 rounded-xl w-max"
          @submit="onSubmit"
      >
        <h3 class="font-bold text-lg">Авторизация</h3>

        <UFormField name="email">
          <UInput
              v-model="state.email"
              variant="soft"
              color="primary"
              placeholder="Электронная почта"
              icon="i-lucide-at-sign"
              class="w-full"
              :ui="{base: 'bg-white/80 text-stone-700'}"
          />
        </UFormField>

        <UFormField name="password">
          <UInput
              v-model="state.password"
              variant="soft"
              color="neutral"
              placeholder="Пароль"
              icon="i-lucide-lock"
              :type="show ? 'text' : 'password'"
              class="w-full"
              :ui="{base: 'bg-white/80 text-stone-700', trailing: 'pe-1'}"
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

        <UButton type="submit" class="btn bg-teal-600/50 cursor-pointer w-full text-center justify-center hover:bg-teal-600">
          Войти
        </UButton>
        <ClientOnly>
        <UFormField>
          <RecaptchaV2
              @widget-id="handleWidgetId"
              @error-callback="handleErrorCallback"
              @expired-callback="handleExpiredCallback"
          />
        </UFormField>
        </ClientOnly>
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
