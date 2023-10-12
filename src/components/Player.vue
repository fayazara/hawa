<template>
  <div class="flex items-center gap-3 text-gray-600 mb-1">
    <Icon :name="icon" class="text-lg" />
    <SliderRoot
      v-model="sliderValue"
      class="relative flex items-center select-none touch-none w-full h-5 flex-1"
      :min="0"
      :max="100"
      :step="1"
    >
      <SliderTrack
        class="bg-gray-200 relative grow rounded-full h-5 overflow-hidden border border-gray-300"
      >
        <SliderRange class="absolute bg-white h-full" />
      </SliderTrack>
      <SliderThumb
        class="w-5 h-5 flex items-center justify-center gap-0.5 rounded-full bg-white shadow border border-gray-300 focus:outline-none focus:bg-gray-100"
        aria-label="Volume"
      >
      </SliderThumb>
    </SliderRoot>
  </div>
</template>
<script setup>
import { onMounted, onUnmounted, ref, watch, computed } from "vue";
import { Howl } from "howler";
import Icon from "./Icon.vue";
import { SliderRange, SliderRoot, SliderThumb, SliderTrack } from "radix-vue";
import { inject } from "vue";

const props = defineProps({
  src: {
    type: String,
    required: true,
  },
  name: {
    type: String,
    required: true,
  },
  icon: {
    type: String,
    required: true,
  },
});

const sliderValue = ref([0]);
const sound = ref(null);
const volume = computed(() => sliderValue.value[0]);
const emitter = inject("emitter");

watch(volume, (value) => {
  if (sound.value) {
    sound.value.volume(value / 100);
    if (value === 0) {
      sound.value.stop();
    } else if (!sound.value.playing()) {
      sound.value.play();
    }
  }
});

onMounted(() => {
  sound.value = new Howl({
    src: [props.src],
    volume: volume.value / 100,
    loop: true,
  });

  sound.value.on("loaderror", () => {
    console.error("Error loading audio");
  });

  if (volume.value > 0) {
    sound.value.play();
  }

  emitter.on("stopAll", () => {
    sound.value.stop();
    sliderValue.value = [0];
  });
});

onUnmounted(() => {
  if (sound.value) {
    sound.value.stop();
    sound.value.unload();
  }
});
</script>
