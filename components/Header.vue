<script setup lang="ts">
import ColorModeButton from "~/components/ColorModeButton.vue";

const isOpen = ref(false)
const toggleMenu = () => isOpen.value = !isOpen.value
const close = () => isOpen.value = false

const auth = ref(false)
onMounted(async () => {
  const token = localStorage.getItem('token');
  if(!token)
    return;
  const response = await fetch('https://backend.mrekk.ru/api/users/get', {
    method: 'GET',
    headers: {'token': token},
  });
  if(response.ok) {
    auth.value = true;
  }
})
function logout(){
  localStorage.removeItem('token');
  localStorage.removeItem('username');
  window.location.href = '/';
}
</script>


<template>
  <header class="glass shadow-sm">
    <div class="container mx-auto px-4 py-3 flex items-center justify-between">
      <!-- Логотип -->
      <NuxtLink to="/" class="text-xl font-semibold hover:opacity-80">
        BSMC
      </NuxtLink>

      <nav class="hidden md:flex space-x-6">
        <NuxtLink to="/" class="hover:text-primary transition">Главная</NuxtLink>
        <NuxtLink to="/cabinet" class="hover:text-primary" @click="close">Личный кабинет</NuxtLink>
        <NuxtLink to="https://discord.gg/6vGWywcW5n" class="hover:text-primary transition">Discord</NuxtLink>
        <NuxtLink to="https://t.me/blacksoulsmc" class="hover:text-primary transition">Telegram</NuxtLink>
      </nav>

      <div class="flex items-center space-x-3">
        <!-- Условный рендеринг кнопки -->
        <UButton
            v-if="auth"
            @click="logout"
            variant="soft"
            class="hidden md:inline-flex"
        >
          Выйти
        </UButton>

        <UButton
            v-else
            to="/login"
            variant="soft"
            class="hidden md:inline-flex"
        >
          Войти
        </UButton>

        <UButton
            icon="i-heroicons-bars-3"
            variant="ghost"
            class="md:hidden"
            @click="toggleMenu"
        />
      </div>
    </div>

    <Transition
        name="slide-fade"
        enter-active-class="transition-all duration-300 ease-out"
        leave-active-class="transition-all duration-200 ease-in"
        enter-from-class="opacity-0 -translate-y-2"
        enter-to-class="opacity-100 translate-y-0"
        leave-from-class="opacity-100 translate-y-0"
        leave-to-class="opacity-0 -translate-y-2"
    >
      <div v-if="isOpen" class="md:hidden">
        <nav class="flex flex-col px-4 py-3 space-y-2">
          <NuxtLink to="/" class="hover:text-primary" @click="close">Главная</NuxtLink>
          <NuxtLink to="/cabinet" class="hover:text-primary" @click="close">Личный кабинет</NuxtLink>
          <NuxtLink to="https://discord.gg/6vGWywcW5n" class="hover:text-primary" @click="close">Discord</NuxtLink>
          <NuxtLink to="https://t.me/blacksoulsmc" class="hover:text-primary" @click="close">Telegram</NuxtLink>
          <UButton to="/login" block class="mt-2" @click="close" variant="soft">
            Войти
          </UButton>
        </nav>
      </div>
    </Transition>
  </header>
</template>



<style scoped>

</style>