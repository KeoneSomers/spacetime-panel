<script setup lang="ts">
import { ref, watchEffect } from "vue";
import { codeToHtml } from "shiki";

const props = defineProps<{ content: string }>();
const html = ref("");

watchEffect(async() => {
    html.value = await codeToHtml(props.content ?? "", {
        lang: "javascript",
        theme: "dark-plus",
    });
});
</script>

<template>
    <div
        v-html="html"
        class="w-full rounded-xl bg-zinc-900 overflow-auto"
    ></div>
</template>

<style>
.shiki {
    padding: 1rem;
}
</style>
