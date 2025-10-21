<script setup lang="ts">
import { codeToHtml } from 'shiki'

const props = defineProps({
    content: String
});

const html = await codeToHtml(props.content ?? "", {
    lang: 'javascript',
    theme: 'dark-plus'
})
</script>

<template>
    <div v-html="html" class="w-calc(100vw-4rem) rounded-xl m-4 bg-zinc-900 overflow-auto"></div>
</template>

<style>
.shiki {
    padding: 1rem;
}
</style>