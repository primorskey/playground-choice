<script setup>
import { onMounted, ref } from "vue";
import CardComponent from "./components/CardComponent.vue";
import CustomChoicesComponent from "./components/CustomChoicesComponent.vue";

const all_choices = ref([]),
  not_yet_selected = ref([]),
  already_selected = ref([]),
  settings_displayed = ref(false),
  json_default = ref([]);

const generateRandomInt = (min, max) => {
  return Math.floor(Math.random() * (max - min + 1) + min);
};

const initializeChoice = () => {
  let saved_choices = localStorage.getItem("choices");
  if (saved_choices) {
    all_choices.value = JSON.parse(saved_choices);
  } else {
    localStorage.setItem("choices", JSON.stringify(json_default.value));
    all_choices.value = json_default.value;
  }
  already_selected.value = [];
  not_yet_selected.value = JSON.parse(JSON.stringify(all_choices.value));
};

const randomizeChoice = () => {
  let minIndex = 0;
  let maxIndex = not_yet_selected.value.length;
  if (minIndex === maxIndex) {
    return;
  }
  let selected_index = generateRandomInt(minIndex, maxIndex - 1);
  already_selected.value.push(not_yet_selected.value[selected_index]);
  not_yet_selected.value.splice(selected_index, 1);
};

const showSettingsPage = () => {
  settings_displayed.value = true;
};

const closeSettingsPage = (reinitialize = false) => {
  if (reinitialize) {
    initializeChoice();
  }
  settings_displayed.value = false;
};

onMounted(async () => {
  let json_url = import.meta.env.BASE_URL + "json/sample.json";
  let json_req = await fetch(json_url);
  json_default.value = await json_req.json();

  initializeChoice();
});
</script>

<template>
  <div class="c-main-app h-[100vh] w-[100vw]">
    <div
      v-if="settings_displayed"
      class="c-blur h-[100vh] w-[100vw] fixed z-20"
    >
      <CustomChoicesComponent
        @closePopup="closeSettingsPage"
      ></CustomChoicesComponent>
    </div>
    <div class="flex flex-row gap-4 p-4">
      <div class="c-selector-container flex flex-col flex-1 gap-4">
        <div class="flex flex-row gap-4 w-[600px]">
          <div
            class="c-main-button rounded-full flex-1"
            @click="randomizeChoice()"
          >
            Select
          </div>
          <div class="c-main-button rounded-full" @click="initializeChoice()">
            Reinitialize
          </div>
        </div>

        <div class="grid grid-cols-6 gap-4">
          <card-component
            v-for="(choice, index) in not_yet_selected"
            :key="index"
            card_type="unselected"
            :title="choice.title"
            :content="choice.content"
          ></card-component>
        </div>
      </div>
      <div
        v-if="already_selected.length > 0"
        class="c-already-selected-container min-w-1/3 grid grid-cols-3 gap-4 justify-evenly h-fit"
      >
        <card-component
          v-for="(selected, index) in already_selected"
          :key="index"
          card_type="selected"
          :title="selected.title"
          :content="selected.content"
          :enable_hover="true"
        ></card-component>
      </div>
    </div>
    <div class="fixed bottom-4 left-4">
      <img
        src="/icons/settings.svg"
        class="c-clickable h-8"
        @click="showSettingsPage()"
      />
    </div>
  </div>
</template>

<style scoped>
.c-blur {
  background: rgba(0, 0, 0, 0.3);
}

.c-main-button {
  background-color: red;
  text-align: center;
  padding: 5px 25px 5px 25px;
}
.c-main-button:hover {
  cursor: pointer;
}
</style>
