<script setup>
import { onMounted, computed } from "vue";

const state = reactive({
  colors: [],
});

function generateColor() {
  return "#" + Math.random().toString(16).substr(-6);
}

function updateColor(index, isLocked) {
  if (!isLocked) {
    state.colors[index].hexCode = generateColor();
  }
}

function updateAllColors() {
  state.colors = []; // reset colors
  for (let i = 0; i < 5; i += 1) {
    state.colors.push({ hexCode: generateColor(), locked: false });
  }
}

function lockColor(index) {
  state.colors[index].locked = !state.colors[index].locked;
  localStorage.setItem("colors", JSON.stringify(state.colors));
}

const activeColor = ref("");
function copyToClipboard(value) {
  activeColor.value = value;
  navigator.clipboard.writeText(value);
  toggleNotifyActive();
}

function getAllColorCodes() {
  let hexCodes = [];
  for (let i = 0; i < state.colors.length; i++) {
    hexCodes.push(state.colors[i].hexCode);
  }
  console.log(hexCodes);
  return hexCodes;
}

const notifyActive = ref(false);
function toggleNotifyActive() {
  notifyActive.value = true;
  setTimeout(() => (notifyActive.value = false), 2000);
}

const countLocked = computed(() => {
  return state.colors.filter((v) => v.locked == true).length >= 5;
});

onMounted(() => {
  updateAllColors();
  if (localStorage.getItem("colors")) {
    state.colors = JSON.parse(localStorage.getItem("colors"));
  }
});
</script>

<template>
  <main class="h-full">
    <p
      v-if="notifyActive"
      class="bg-dark-200 text-light-300 p4 text-center text-3xl font-bold absolute w-full z-5"
    >
      Color
      <span class="p1" :style="{ backgroundColor: activeColor }">{{
        activeColor
      }}</span>
      copied!
    </p>

    <a
      href="https://github.com/nulloneguy/vue-colors"
      class="absolute top-10 right-7 bg-dark-300 rounded-100 z-10"
    >
      <div class="text-5xl bg-light-300" i="carbon-logo-github" />
    </a>

    <ul class="grid grid-cols-5 h-full text-light-300">
      <li
        v-for="(color, index) in state.colors"
        class="flex flex-col items-center justify-center h-full cursor-pointer drop-shadow-2xl hover:scale-110 hover:z-1 ease-out duration-10"
        :style="{ background: color.hexCode }"
        :key="index"
        @click="updateColor(index, color.locked)"
      >
        <button
          class="p-4 bg-dark-400 rounded-xl drop-shadow-xl"
          @click="lockColor(index)"
          @click.stop
        >
          <div class="text-7xl" i="carbon-locked" v-if="color.locked" />
          <div class="text-7xl" i="carbon-unlocked" v-else />
        </button>
        <div
          class="p3 text-xl mt-2 bg-dark-300 rounded-xl drop-shadow-xl"
          @click="copyToClipboard(color.hexCode)"
          @click.stop
        >
          {{ color.hexCode }}
        </div>
        <p class="text-ligh-300 font-bold text-xl mt-5">Light text</p>
        <p class="text-dark-700 font-bold text-xl">Dark text</p>
      </li>
    </ul>
    <div
      v-if="countLocked"
      class="absolute bottom-0 w-full p6 bg-dark-300 text-light-300 font-bold text-center flex items-center justify-center z-5"
    >
      You have generated a color scheme. Do you want to copy it?
      <button
        class="py-2 px-6 ml-5 bg-blue-400 rounded-xl"
        @click="copyToClipboard(getAllColorCodes())"
      >
        Copy
      </button>
    </div>
  </main>
</template>
