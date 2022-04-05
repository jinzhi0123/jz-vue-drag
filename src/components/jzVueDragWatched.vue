<template>
    <div class="drag-watched-area" ref="dragWatchedArea" :style="dragWatchedAreaStyle">
        <div class="drag-watched-inner-area" ref="innerArea" :style="innerAreaStyle"></div>
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
    checkOverlapArea: {
        type: String,
        default: 'containerArea',
        validator(value) {
            return ['containerArea', 'innerArea'].includes(value as string)
        }
    }
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

const innerAreaW = computed(() => {
    if (typeof (props.innerAreaW) === 'number') return props.innerAreaW + 'px'
    if (typeof (props.innerAreaW) === 'string') return props.innerAreaW
})
const innerAreaH = computed(() => {
    if (typeof (props.innerAreaH) === 'number') return props.innerAreaH + 'px'
    if (typeof (props.innerAreaH) === 'string') return props.innerAreaH
})

const innerAreaT = computed(() => {
    if (typeof (props.innerAreaT) === 'number') return props.innerAreaT + 'px'
    if (typeof (props.innerAreaT) === 'string') return props.innerAreaT
})
const innerAreaL = computed(() => {
    if (typeof (props.innerAreaL) === 'number') return props.innerAreaL + 'px'
    if (typeof (props.innerAreaL) === 'string') return props.innerAreaL
})


const dragWatchedArea = ref() as Ref<HTMLElement>
const innerArea = ref() as Ref<HTMLElement>
const dragWatchedAreaStyle = computed(() => {
    return { 'z-index': props.zIndex }
})
const innerAreaStyle = computed(() => {
    return { width: innerAreaW.value, height: innerAreaH.value, top: innerAreaT.value, left: innerAreaL.value }
})
function getPosition(): positionType {
    let rect = new DOMRect
    if (props.checkOverlapArea == 'containerArea') {
        rect = dragWatchedArea.value.getBoundingClientRect();
    }
    if (props.checkOverlapArea == 'innerArea') {
        rect = innerArea.value.getBoundingClientRect();
    }
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
    background-color: rgba(255, 255, 0, 0.5);
    z-index: 1;
}
.drag-watched-inner-area {
    background-color: rgba(58,58,255,0.5);
    position: absolute;
}
</style>