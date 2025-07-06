<script setup lang="ts">
import { useEventListener, useLocalStorage } from "@vueuse/core";
import type {
  PlayerAnimation,
  SkinLoadOptions,
} from "skinview3d";
import { computed, nextTick, onMounted, reactive, ref, watch } from "vue";
import type { Background, Layers } from "vue-skinview3d";
import { SkinView3d } from "vue-skinview3d";
import {
  FlyingAnimation,
  IdleAnimation,
  RunningAnimation,
  WalkingAnimation,
} from "vue-skinview3d/animations";


const availableAnimations = {
  idle: new IdleAnimation(),
  walk: new WalkingAnimation(),
  run: new RunningAnimation(),
  fly: new FlyingAnimation(),
};
const BUILTIN_SKINS = [
  "img/hatsune_miku.png",
];

const renderPaused = ref(false);
const DEFAULT_WIDTH = 300;
const DEFAULT_HEIGHT = 500;
const width = ref(DEFAULT_WIDTH);
const height = ref(DEFAULT_HEIGHT);
const fov = ref(70);
const zoom = ref(0.6);
const globalLight = ref(3);
const cameraLight = ref(0.6);
const autoRotate = ref(false);
const autoRotateSpeed = ref(2);
const animationType = ref(null as null | keyof typeof availableAnimations);
const animationSpeed = ref(1);
const animationPlaying = ref(true);
const animation = computed<PlayerAnimation | null>(() => {
  const animationClass =
      animationType.value && availableAnimations[animationType.value];
  animationClass && (animationClass.speed = animationSpeed.value);
  animationClass && (animationClass.paused = !animationPlaying.value);

  return animationClass;
});
const controls = reactive({
  rotate: true,
  zoom: true,
  pan: false,
});
const layers = reactive<Layers>({
  inner: {
    head: true,
    body: true,
    leftArm: true,
    rightArm: true,
    leftLeg: true,
    rightLeg: true,
  },
  outer: {
    head: true,
    body: true,
    leftArm: true,
    rightArm: true,
    leftLeg: true,
    rightLeg: true,
  },
});

const skinOptions = reactive<SkinLoadOptions>({});
const skinRef = ref<HTMLElement | null>(null);
const enableWideUI = useLocalStorage("enableWideUI", false);
const wideUIClass = computed(() => ({ "wide-ui": enableWideUI.value }));
onMounted(adjustUI);
watch(enableWideUI, adjustUI);
useEventListener("resize", onResize);

function onResize() {
  if (enableWideUI.value) {
    height.value = skinRef.value!.offsetHeight;
    width.value = skinRef.value!.offsetWidth;
  }
}
async function adjustUI() {
  await nextTick();
  if (enableWideUI.value) {
    onResize();
  } else {
    height.value = DEFAULT_HEIGHT;
    width.value = DEFAULT_WIDTH;
  }
}
</script>
<script lang="ts">
export default {
  props:['skinUrl', 'skinKey'],
  data(){
    return{
    }
  },
  methods:{
    reRender(){
      this.skinKey++;
    }
  }
}
</script>

<template>
  <SkinView3d
      :key="skinKey"
      :animation="animation"
      :auto-rotate="autoRotate"
      :auto-rotate-speed="autoRotateSpeed"
      :camera-light="cameraLight"
      :enable-pan="controls.pan"
      :enable-rotate="controls.rotate"
      :enable-zoom="controls.zoom"
      :fov="fov"
      :global-light="globalLight"
      :height="height"
      :layers="layers"
      :render-paused="renderPaused"
      :skin-options="skinOptions"
      :skin-url="skinUrl"
      :width="width"
      :zoom="zoom"
  />
</template>

<style scoped>
</style>