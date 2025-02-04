<script setup>
import { ref } from "vue";

const props = defineProps({
  card_type: {
    type: String,
    default: "unselected",
  },
  title: {
    type: String,
    default: "",
  },
  content: {
    type: String,
    default: "",
  },
  enable_hover: {
    type: Boolean,
    default: false,
  },
});

const active = ref(false);
</script>

<template>
  <div
    class="c-card rounded-lg h-fit min-h-[100px] grid grid-flow-row auto-rows-auto text-center items-center"
    :class="[
      card_type === 'unselected' ? 'c-unselected-card' : 'c-selected-card',
    ]"
    @mouseover="enable_hover ? (active = true) : null"
    @mouseleave="active = false"
  >
    <div>{{ card_type === "unselected" ? "?" : title }}</div>
    <div v-if="active">{{ content }}</div>
  </div>
</template>

<style scoped>
.c-card {
  padding: 5px 25px 5px 25px;
}
.c-selected-card {
  background-color: var(--c-green);
  color: white;
}

.c-unselected-card {
  background-color: var(--c-gray-normal);
  color: white;
}
</style>
