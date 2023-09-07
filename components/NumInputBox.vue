<!-- backup -->
<script lang="ts" setup>
const props = withDefaults(defineProps<{
    name: string,
    initial: any,
    disabled?: boolean
}>(), {
    disabled: false
})
const data = ref(props.initial)
</script>

<template>
    <input type="number" :name="props.name" :id="props.name" v-model="data" :disabled="props.disabled">
</template>

<style scoped></style>
