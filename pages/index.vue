<script setup>
import { ref, onMounted } from 'vue'

const currentImage = ref(1)

onMounted(() => {
  setInterval(() => {
    currentImage.value = currentImage.value === 9 ? 1 : currentImage.value + 1
  }, 5000) // каждые 5 сек
})
</script>

<template class="bg-black">
  <Header class="text-white"/>

  <div class="relative h-[100vh] overflow-hidden">
    <!-- background layers -->
    <div class="fixed inset-0 bg-black -z-10 pointer-events-none"></div>

    <div class="fixed inset-0 -z-10 overflow-hidden">
      <transition-group name="fade" tag="div">
        <img
            v-for="(image, index) in [currentImage]"
            :key="image"
            :src="`/images/bg/index/${image}.jpg`"
            alt="Background"
            class="absolute inset-0 w-full h-full object-cover blur"
        />
      </transition-group>
    </div>

    <div class="fixed inset-0 bg-black/50 -z-10 pointer-events-none"></div>

    <!-- centered content -->
    <div class="flex flex-col items-center justify-center h-full text-white relative z-10">
      <h1 class="text-3xl font-bold text-center drop-shadow-lg">Black Souls 2 в мире Майнкрафта жееесть</h1>
      <UButton color="neutral"
               size="xl"
               icon="i-lucide-rocket"
               class="btn px-3 py-2 mt-5 cursor-pointer rounded bg-emerald-500/50 text-white w-[200px] justify-center
                      hover:bg-emerald-400/20 hover:ring-2 ring-emerald-400
                      active:bg-emerald-100/20"
               to="/registration">Регистрация</UButton>
      <UButton color="neutral"
               size="xl"
               icon="i-lucide-folder-down"
               class="btn px-3 py-2 mt-5 cursor-pointer rounded bg-emerald-500/50 text-white w-[200px] justify-center
                      hover:bg-emerald-400/20 hover:ring-2 ring-emerald-400
                      active:bg-emerald-100/20">Скачать лаунчер</UButton>
    </div>
  </div>
</template>


<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s ease;
  background: black;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  background: black;
}
</style>
