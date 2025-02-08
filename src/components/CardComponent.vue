<script setup>
import { ref, computed } from "vue";

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
    highlighted: {
        type: Boolean,
        default: false,
    },
    app_settings: {
        type: Object,
        default: () => {},
    },
});

const active = ref(false);

const displayed_label = computed(() => {
    if (props.app_settings.INVERT_TITLE) {
        return props.content;
    }
    return props.title;
});

const displayed_content = computed(() => {
    if (props.app_settings.INVERT_TITLE) {
        return props.title;
    }
    return props.content;
});
</script>

<template>
    <div
        class="c-card rounded-lg h-fit min-h-[100px] grid grid-flow-row auto-rows-auto text-center items-center"
        :class="[
            card_type === 'unselected'
                ? 'c-unselected-card'
                : 'c-selected-card',
            highlighted === true ? 'c-highlighted-card' : '',
        ]"
        @mouseover="enable_hover ? (active = true) : null"
        @mouseleave="active = false"
    >
        <div>{{ card_type === "unselected" ? "?" : displayed_label }}</div>
        <div v-if="active">{{ displayed_content }}</div>
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

.c-highlighted-card {
    background-color: var(--c-gray-light);
    transform: scale(1.1);
    transition: all 150ms;
}
</style>
