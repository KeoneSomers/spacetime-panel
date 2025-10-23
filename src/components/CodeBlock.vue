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

code {
    counter-reset: step;
    counter-increment: step 0;
}

code .line::before {
    content: counter(step);
    counter-increment: step;
    width: 1rem;
    margin-right: 1.5rem;
    display: inline-block;
    text-align: right;
    color: rgba(115, 138, 148, .4)
}
</style>
