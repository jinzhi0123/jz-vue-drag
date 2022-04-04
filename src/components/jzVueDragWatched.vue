<template>
    <div  class="drag-watched-area" ref="dragWatchedArea" :style="dragWatchedAreaStyle">
        <slot></slot>
    </div>
</template>

<script setup lang="ts">
import { ref, Ref, computed } from 'vue'
const props = defineProps({
    zIndex: {
        type: Number,
        default: 0
    },
    innerAreaW: {
        type: [Number, String],
        default: '100%',
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && value.indexOf('%') != -1)
        }
    },
    innerAreaH: {
        type: [Number, String],
        default: '100%',
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && (value.indexOf('%') != -1))
        }
    },
    innerAreaT: {
        type: [Number, String],
        default: 0,
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && (value.indexOf('%') != -1))
        }
    },
    innerAreaL: {
        type: [Number, String],
        default: 0,
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && value.indexOf('%') != -1)
        }
    },
})
defineExpose({
    getPosition
})
type positionType = {
    top: number,
    bottom: number,
    left: number,
    right: number,
    width: number,
    height: number,
}
const dragWatchedArea = ref() as Ref<HTMLElement>
const dragWatchedAreaStyle = computed(() => {
    return { 'z-index': props.zIndex }
})
function getPosition(): positionType {
    const rect = dragWatchedArea.value.getBoundingClientRect();
    const position: positionType = {
        top: rect.top,
        bottom: rect.bottom,
        left: rect.left,
        right: rect.right,
        width: rect.width,
        height: rect.height
    }
    return position
}
</script>

<style scoped>
.drag-watched-area {
    position: absolute;
    width: fit-content;
    height: fit-content;
    background-color: rgba(255,255,0,0.5);
    z-index: 1;
}
</style>