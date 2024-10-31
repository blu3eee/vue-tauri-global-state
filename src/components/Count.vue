<script setup lang="ts">
import { invoke } from "@tauri-apps/api/core";
import { listen } from "@tauri-apps/api/event";
import { onMounted, ref, watch } from "vue";

const count = ref(0);
const isLoop = ref(false);
async function getCount() {
  count.value = await invoke("get_count");
}

async function setCount(value: number) {
  console.log("setCount called");
  await invoke("set_count", { count: value });
}

function increaseCount() {
  setCount(count.value + 1);
}

function setLoop() {
  isLoop.value = !isLoop.value;
}

let interval: NodeJS.Timeout | null = null;

function startLoop() {
  if (interval === null) {
    setInterval(() => {
      if (isLoop.value) {
        increaseCount();
      } else {
        stopLoop();
      }
    }, 1000 / 60);
  }
}

function stopLoop() {
  if (interval !== null) {
    clearInterval(interval);
    interval = null;
  }
}

watch(isLoop, (newValue) => {
  console.log("isLoop changed:", newValue);
  if (newValue) {
    startLoop();
  } else {
    stopLoop();
  }
});
onMounted(async () => {
  await getCount();
  // Listen for count_changed events
  await listen("count_changed", (event: any) => {
    count.value = event.payload.count;
  });
});
</script>

<template>
  <div>
    <div>Count: {{ count }}</div>
    <button @click="increaseCount">Increase Count</button>
    <button @click="setLoop">{{ isLoop ? "Stop loop" : "Start loop" }}</button>
  </div>
</template>
