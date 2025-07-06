<script setup lang="ts">
import { RecaptchaV2, useRecaptcha } from 'vue3-recaptcha-v2';
import * as v from 'valibot';
import type { FormSubmitEvent } from "@nuxt/ui";

const skinUrl = computed(() =>"https://backend.mrekk.ru/api/users/getskin/" + localStorage.getItem('username'));
const fileInput = ref<HTMLInputElement | null>(null);
const show = ref(false)
const showRepeated = ref(false)
const schema = v.object({
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

const state = reactive({
  password: '',
  repeatedPassword: '',
})

const { handleGetResponse, handleReset } = useRecaptcha();
const captchaToken = ref<string | null>(null)

const username = computed(() =>  localStorage.getItem("username"))

const widgetId = ref<number | null>(null)

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

async function uploadSkin(event: Event) {
  const captchaToken = handleGetResponse(widgetId.value);  // Получаем токен капчи

  if (!captchaToken) {
    toast.add({ title: 'Ошибка!', description: 'Подтвердите, что вы не робот.', color: 'error' });
    return;
  }

  const input = event.target as HTMLInputElement;
  const file = input.files?.[0];

  if (!file) {
    toast.add({ title: 'Ошибка!', description: 'Выберите файл.', color: 'error' });
    return;
  }

  const validTypes = ['image/png'];
  if (!validTypes.includes(file.type)) {
    toast.add({ title: 'Ошибка!', description: 'Неверный формат файла. Требуется PNG.', color: 'error' });
    return;
  }

  const image = new Image();
  const token = localStorage.getItem("token");
  const reader = new FileReader();
  reader.onload = () => {
    image.src = reader.result as string;

    image.onload = async () => {
      if (image.width > 64 || image.height > 64) {
        toast.add({ title: 'Ошибка!', description: 'Максимальный размер изображения должен 64x64.', color: 'error' });
        return;
      }

      const formData = new FormData();
      formData.append('skin', file);
      formData.append('captchaToken', captchaToken);

      if(!token) {return}
      try {
        console.log(1)
        const response = await fetch('https://backend.mrekk.ru/api/users/uploadskin', {
          method: 'POST',
          headers: {
            'token': token,
          },
          body: formData,
        });

        if (response.ok) {
          toast.add({ title: 'Успех!', description: 'Скин успешно загружен.', color: 'success' });
          setInterval(() => window.location.reload(), 3000);
        } else {
          toast.add({ title: 'Ошибка!', description: 'Не удалось загрузить скин.', color: 'error' });
        }
      } catch (error) {
        toast.add({ title: 'Ошибка!', description: 'Произошла ошибка при загрузке.', color: 'error' });
      }
    };

    image.onerror = () => {
      toast.add({ title: 'Ошибка!', description: 'Ошибка при загрузке изображения.', color: 'error' });
    };
  };

  reader.readAsDataURL(file); // Считываем файл как URL
  handleReset(widgetId.value);
}
onMounted(async () => {
  const token = localStorage.getItem('token');
  const route = useRoute()
  if(!token)
    return navigateTo('/login');
  const response = await fetch('https://backend.mrekk.ru/api/users/get', {
    method: 'GET',
    headers: {'token': token},
  });
  if(!response.ok) {
    return navigateTo('/login');
  }
})
async function changePassword(event: Event) {

  console.log(state.password);
}
</script>

<template>
  <main class="bg">
    <Header/>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-x-4 gap-y-2 mt-5">
      <div class="skin flex flex-col justify-center items-center">
        <h1 class="text-2xl font-bold">{{ username }}</h1>
        <Skin3D :skinKey="123" :skinUrl="skinUrl"></Skin3D>
        <div class="flex items-center justify-center w-[300px]">
          <label for="dropzone-file"
                 class="glass flex flex-col items-center justify-center w-[300px] h-[150px]
                        border-2 border-gray-300 border-dashed rounded-lg cursor-pointer bg-gray-50
                        hover:bg-gray-100 dark:border-gray-600">
            <div class="flex flex-col items-center justify-center pt-5 pb-6">
              <svg class="w-8 h-8 mb-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 16">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 13h3a3 3 0 0 0 0-6h-.025A5.56 5.56 0 0 0 16 6.5 5.5 5.5 0 0 0 5.207 5.021C5.137 5.017 5.071 5 5 5a4 4 0 0 0 0 8h2.167M10 15V6m0 0L8 8m2-2 2 2"/>
              </svg>
              <p class="mb-2 text-sm text-center">
                <span class="font-semibold">Нажмите чтобы загрузить скин</span>
                или перетащите файл скина</p>
              <p class="text-xs">PNG, JPG (макс. 64х64)</p>
            </div>
            <UInput
                id="dropzone-file"
                type="file"
                class="hidden"
                ref="fileInput"
                @change="uploadSkin"
            />
          </label>
        </div>
      </div>
      <div class="change-form flex flex-col items-center mb-5 mt-5">
        <UForm
            :schema="schema"
            :state="state"
            class="space-y-4 glass max-w-sm px-5 py-5 rounded-xl"
            @submit="changePassword"
        >
          <h1 class="text-2xl font-bold">Смена пароля</h1>

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
            Изменить
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
    </div>

  </main>
</template>

<style scoped>
.bg{
  background-image: url("public/images/bg/cabinet.jpg");
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  min-height: 100vh;
}
</style>
