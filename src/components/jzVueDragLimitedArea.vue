<template>
    <div class="drag-limited-area" :style="dragLimitedRreaStyle" ref="dragLimitedArea">
        <slot></slot>
    </div>
</template>

<script setup lang="ts">
import { ref, Ref, computed, onMounted, provide, PropType } from 'vue'

const props = defineProps({
    dragLimitedRreaW: {
        type: Number,
        required: true,
    },
    dragLimitedRreaH: {
        type: Number,
        required: true,
    }

})
// const dragLimitedArea = ref() as Ref<HTMLElement>
const dragLimitedRreaW = computed(() => props.dragLimitedRreaW)
const dragLimitedRreaH = computed(() => props.dragLimitedRreaH)
// onMounted(() => {
//     console.log(dragLimitedArea.value.getBoundingClientRect().height)
// })
provide('dragLimitedRreaW', dragLimitedRreaW.value)
provide('dragLimitedRreaH', dragLimitedRreaH.value)


const dragLimitedRreaStyle = computed(() => {
    return { width: dragLimitedRreaW.value + 'px', height: dragLimitedRreaH.value + 'px' }
})

onMounted(() => {

})
</script>

<style scoped>
.drag-limited-area {
    position: absolute;
    margin: 70px;
    background-color: rgba(170, 170, 170, 0.2);
    /* z-index: -1; */
}
</style>