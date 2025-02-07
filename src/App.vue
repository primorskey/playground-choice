<script setup>
import { onMounted, ref } from "vue";
import CardComponent from "./components/CardComponent.vue";
import CustomChoicesComponent from "./components/CustomChoicesComponent.vue";

const all_choices = ref([]),
    not_yet_selected = ref([]),
    already_selected = ref([]),
    settings_displayed = ref(false),
    json_default = ref([]),
    animation_countdown = ref(0),
    highlighted_index = ref(null),
    preview_answers = ref(false);

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
    highlighted_index.value = null;
};

const randomizeChoice = () => {
    let minIndex = 0;
    let maxIndex = not_yet_selected.value.length;
    if (minIndex === maxIndex) {
        return;
    }
    let selected_index = generateRandomInt(minIndex, maxIndex - 1);

    if (already_selected.value.length > 0) {
        highlighted_index.value = selected_index;
        setTimeout(() => {
            animateDoneCallback(selected_index);
        }, 500);
    } else {
        highlighted_index.value = 0;
        animateChoice(selected_index, 100, 500);
    }
};

const animateDoneCallback = (target_index) => {
    highlighted_index.value = null;
    already_selected.value.push(not_yet_selected.value[target_index]);
    not_yet_selected.value.splice(target_index, 1);
};

const animateChoice = (
    target_index,
    animation_length,
    max_animation_length,
    loops = 0
) => {
    let should_loop = false;
    if (not_yet_selected.value.length < all_choices.value.length * 0.7) {
        should_loop = loops < not_yet_selected.value.length;
    }
    let animation_multiplier =
        0.75 + (target_index / not_yet_selected.value.length) * (1.25 - 0.75);
    if (
        should_loop === false &&
        target_index < not_yet_selected.value.length * 0.2
    ) {
        should_loop = loops < not_yet_selected.value.length;
        animation_multiplier = 0.75;
    }
    let nextAnimationLength = Math.min(
        animation_length * animation_multiplier,
        max_animation_length
    );
    setTimeout(() => {
        if (highlighted_index.value !== target_index || should_loop) {
            if (highlighted_index.value === not_yet_selected.value.length) {
                highlighted_index.value = 0;
            } else {
                highlighted_index.value += 1;
            }

            animateChoice(
                target_index,
                animation_length,
                max_animation_length,
                loops + 1
            );
        } else {
            setTimeout(() => {
                animateDoneCallback(target_index);
            }, 500);
        }
    }, nextAnimationLength);
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
        <div class="gap-4 p-4 grid grid-cols-3">
            <div
                v-if="not_yet_selected.length > 0"
                class="grid gap-4 h-fit"
                :class="[
                    already_selected.length > 0
                        ? 'md:col-span-2 md:grid-cols-4 grid-cols-2'
                        : 'md:col-span-3 md:grid-cols-6 grid-cols-2',
                ]"
            >
                <card-component
                    v-for="(choice, index) in not_yet_selected"
                    :key="index"
                    card_type="unselected"
                    :title="choice.title"
                    :content="choice.content"
                    :highlighted="highlighted_index === index"
                ></card-component>
            </div>
            <div
                v-if="already_selected.length > 0"
                class="c-already-selected-container grid gap-4 h-fit"
                :class="[
                    not_yet_selected.length === 0
                        ? 'md:col-span-3 md:grid-cols-4 grid-cols-2'
                        : ' grid-cols-2',
                ]"
            >
                <card-component
                    v-for="(selected, index) in already_selected"
                    :key="index"
                    card_type="selected"
                    :title="selected.title"
                    :content="selected.content"
                    :enable_hover="preview_answers ? true : false"
                ></card-component>
            </div>
        </div>
        <div class="fixed bottom-4 left-4 flex flex-row gap-4">
            <img
                src="/icons/settings.svg"
                class="c-clickable h-8"
                @click="showSettingsPage()"
            />
            <img
                v-if="preview_answers === false"
                src="/icons/eye.svg"
                class="c-clickable h-8"
                @click="preview_answers = true"
            />
            <img
                v-if="preview_answers === true"
                src="/icons/eye_closed.svg"
                class="c-clickable h-8"
                @click="preview_answers = false"
            />
        </div>
        <div class="fixed bottom-4 right-4 md:w-1/3">
            <div class="flex md:flex-row flex-col gap-4">
                <div
                    class="c-main-button c-clickable rounded-full flex-1"
                    @click="randomizeChoice()"
                >
                    Select
                </div>
                <div
                    class="c-secondary-button c-clickable rounded-full"
                    @click="initializeChoice()"
                >
                    Reinitialize
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.c-blur {
    background: rgba(0, 0, 0, 0.3);
}

.c-main-button {
    background-color: var(--c-green);
    text-align: center;
    padding: 5px 25px 5px 25px;
    color: white;
}

.c-secondary-button {
    background-color: var(--c-gray-light);
    text-align: center;
    padding: 5px 25px 5px 25px;
}
</style>
