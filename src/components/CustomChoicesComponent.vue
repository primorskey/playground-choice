<script setup>
import { onMounted, ref } from "vue";

const emit = defineEmits(["closePopup", "toggleDisplayedLabel"]);

const props = defineProps({
    app_settings: {
        type: Object,
        default: () => {},
    },
});

const all_choices = ref([]),
    new_title = ref(""),
    new_content = ref("");

const deleteChoice = (index) => {
    all_choices.value.splice(index, 1);
};

const addChoice = () => {
    if (
        new_title.value.trim().length === 0 ||
        new_content.value.trim().length === 0
    ) {
        return;
    }
    all_choices.value.push({
        title: new_title.value,
        content: new_content.value,
    });
    new_title.value = "";
    new_content.value = "";
};

const saveChoices = () => {
    if (all_choices.value.length === 0) {
        return;
    }
    localStorage.setItem("choices", JSON.stringify(all_choices.value));
    emit("closePopup", true);
};

const resetChoices = () => {
    localStorage.removeItem("choices");
    emit("closePopup", true);
};

onMounted(async () => {
    let saved_choices = localStorage.getItem("choices");
    if (saved_choices) {
        all_choices.value = JSON.parse(saved_choices);
    } else {
        let json_req = await fetch("/json/sample.json");
        let json_data = await json_req.json();
        localStorage.setItem("choices", JSON.stringify(json_data));
        all_choices.value = json_data;
    }
});
</script>

<template>
    <div
        class="w-fit min-w-[350px] h-fit max-h-[60vh] flex flex-col bg-white rounded-lg mx-auto mt-[15%] relative"
    >
        <div
            @click="emit('closePopup')"
            class="c-clickable absolute top-1 left-1"
        >
            <div
                class="c-clickable ml-auto text-sm text-red-400"
                @click="resetChoices()"
            >
                Reset Choices
            </div>
        </div>
        <div
            @click="emit('closePopup')"
            class="c-clickable absolute top-6 left-1"
        >
            <div
                class="c-clickable ml-auto text-sm text-green-600"
                @click="emit('toggleDisplayedLabel')"
            >
                {{ app_settings.INVERT_TITLE ? "Show Title" : "Show Content" }}
            </div>
        </div>
        <div
            @click="emit('closePopup')"
            class="c-clickable absolute top-1 right-1"
        >
            <img src="/icons/close.svg" class="c-clickable h-6" />
        </div>
        <div class="c-popup-content p-4 pt-6 flex flex-col gap-4 overflow-auto">
            <div class="text-center font-semibold">CHOICES</div>

            <div class="grid grid-flow-row auto-rows-auto gap-2">
                <div
                    v-for="(choice, index) in all_choices"
                    :key="index"
                    class="grid grid-cols-3 gap-8 text-center items-center"
                >
                    <div>{{ choice.title }}</div>
                    <div>{{ choice.content }}</div>
                    <div class="mx-auto">
                        <img
                            src="/icons/delete.svg"
                            class="c-clickable h-6"
                            @click="deleteChoice(index)"
                        />
                    </div>
                </div>
                <div class="grid grid-cols-3 gap-8 text-center items-center">
                    <input
                        type="text"
                        class="c-text-input"
                        v-model="new_title"
                    />
                    <input
                        type="text"
                        class="c-text-input"
                        v-model="new_content"
                    />
                    <div class="mx-auto">
                        <img
                            src="/icons/plus.svg"
                            class="c-clickable h-6"
                            @click="addChoice()"
                        />
                    </div>
                </div>
            </div>
            <div>
                <div
                    class="c-save-button c-clickable rounded-full flex-1"
                    @click="saveChoices()"
                >
                    Save
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.c-save-button {
    background-color: var(--c-green);
    text-align: center;
    padding: 5px 25px 5px 25px;
    color: white;
}
</style>
