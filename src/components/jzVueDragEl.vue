<template>
    <div
        class="drag-container"
        ref="dragContainer"
        :style="[dragContainerStyle, dragContainerPosition]"
    >
        <slot></slot>
        <div
            class="drag-draggable-area"
            :style="draggableAreaStyle"
            ref="draggableArea"
            @touchstart="dragStartTouch"
            @touchmove="dragMoveTouch"
            @touchend="dragEnd"
            @mousedown="dragStartMouse"
        ></div>
        <!-- @mouseup="dragEnd" -->
    </div>
</template>

<script setup lang="ts">
import { ref, Ref, computed, onMounted, inject, PropType } from 'vue'
const dragLimitedRreaW: number = inject('dragLimitedRreaW')!
const dragLimitedRreaH: number = inject('dragLimitedRreaH')!
const emit = defineEmits(
    ['dragstart', 'dragging', 'dragend']
)
const props = defineProps({
    draggableAreaW: {
        type: [Number, String],
        default: '100%',
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && value.indexOf('%') != -1) || value == 'center'
        }
    },
    draggableAreaH: {
        type: [Number, String],
        default: '100%',
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && (value.indexOf('%') != -1)) || value == 'center'
        }
    },
    draggableAreaT: {
        type: [Number, String],
        default: 0,
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && (value.indexOf('%') != -1)) || value == 'center'
        }
    },
    draggableAreaL: {
        type: [Number, String],
        default: 0,
        validator(value) {
            return typeof (value) == 'number' || (typeof (value) == 'string' && value.indexOf('%') != -1) || value == 'center'
        }
    },
    isDraggable: {
        type: Boolean,
        default: true,
    },
    watchComponents: {
        type: Array as PropType<Ref<jzVueDragWatchedType>[]>,
        default: []
    },
    isReset: {
        type: Boolean,
        default: false,
    },
    zIndex: {
        type: Number,
        default: 9999
    },
    isCheckBoundary: {
        type: Boolean,
        default: true
    },
    checkOverlapArea: {
        type: String,
        default: 'containerArea',
        validator(value) {
            return ['containerArea', 'draggableArea'].includes(value as string)
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
type jzVueDragWatchedType = {
    getPosition(): positionType
}
function getPosition(): positionType {
    let rect: DOMRect = new DOMRect
    if (props.checkOverlapArea == 'containerArea') {
        rect = dragContainer.value.getBoundingClientRect();
    }
    if (props.checkOverlapArea == 'draggableArea') {
        rect = draggableArea.value.getBoundingClientRect();
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
const draggableAreaW = computed(() => {
    if (typeof (props.draggableAreaW) === 'number') return props.draggableAreaW + 'px'
    if (typeof (props.draggableAreaW) === 'string') return props.draggableAreaW
})
const draggableAreaH = computed(() => {
    if (typeof (props.draggableAreaH) === 'number') return props.draggableAreaH + 'px'
    if (typeof (props.draggableAreaH) === 'string') return props.draggableAreaH
})

const draggableAreaT = computed(() => {
    if (typeof (props.draggableAreaT) === 'number') return props.draggableAreaT + 'px'
    if (props.draggableAreaT == 'center' && typeof (props.draggableAreaH) === 'string') return (100 - Number(props.draggableAreaH.replace("%", ""))) / 2 + '%'
    if (typeof (props.draggableAreaT) === 'string') return props.draggableAreaT
})
const draggableAreaL = computed(() => {
    if (typeof (props.draggableAreaL) === 'number') return props.draggableAreaL + 'px'
    if (props.draggableAreaL == 'center' && typeof (props.draggableAreaW) === 'string') return (100 - Number(props.draggableAreaW.replace("%", ""))) / 2 + '%'
    if (typeof (props.draggableAreaL) === 'string') return props.draggableAreaL
})
const dragContainer = ref() as Ref<HTMLElement>
const draggableArea = ref() as Ref<HTMLElement>
onMounted(() => {
    dragContainerW.value = dragContainer.value.offsetWidth
    dragContainerH.value = dragContainer.value.offsetHeight
    offTop.value = dragContainer.value.offsetTop
    offLeft.value = dragContainer.value.offsetLeft
    dragContainerPositionFlag.value = true;
})
const dragContainerW = ref(0)
const dragContainerH = ref(0)
const dragContainerTransition = ref('0s')
let a = 'top'
const dragContainerStyle = computed(() => {
    return { transition: dragContainerTransition.value, 'z-index': props.zIndex }
})
const dragContainerPositionFlag = ref(false)
const dragContainerPosition = computed(() => {
    if (dragContainerPositionFlag.value) { return { top: offTop.value + 'px', left: offLeft.value + 'px' } } else { return '' }
})
const draggableAreaStyle = computed(() => {
    return {
        width: draggableAreaW.value, height: draggableAreaH.value,
        top: draggableAreaT.value, left: draggableAreaL.value
    }
})
type curType = {
    x: number,
    y: number
}

const dragFlag = ref(false);  //是否按下鼠标的标记
const cur: Ref<curType> = ref({  //记录鼠标按下时的坐标
    x: 0, //记录当前鼠标的x绝对坐标
    y: 0 //记录当前鼠标的y绝对坐标
})
const offLeft = ref(0); //记录div当时的左偏移量(相对于父元素)
const offTop = ref(0); //记录div的上偏移量(相对于父元素)

const originalTop = ref(0) //原始位置
const originalLeft = ref(0)  //原始位置
let allowClickFlag = true;

function dragStartMouse(event: MouseEvent) {
    if (props.isDraggable && allowClickFlag) {
        console.log("MouseEvent")
        dragFlag.value = true;
        cur.value.x = event.clientX;
        cur.value.y = event.clientY;
        originalTop.value = dragContainer.value.offsetTop;
        originalLeft.value = dragContainer.value.offsetLeft;
        offLeft.value = dragContainer.value.offsetLeft;
        offTop.value = dragContainer.value.offsetTop;
        let emitData = {
            curX: event.clientX,
            curY: event.clientY,
            originalTop: originalTop.value,
            originalLeft: originalLeft.value
        }
        emit('dragstart', emitData)
        document.onmousemove = (event) => {
            event.preventDefault();
            let offx = event.clientX - cur.value.x;
            let offy = event.clientY - cur.value.y;
            checkBoundary(offx, offy);
            let overlap = overlapDraggingFunc()
            let emitData = {
                curX: event.clientX,
                curY: event.clientY,
                originalTop: dragContainer.value.offsetTop,
                originalLeft: dragContainer.value.offsetLeft,
                isOverlap: overlap[0],
                overlapList: overlap[1]
            }
            emit('dragging', emitData)
        }
        document.onmouseup = (event) => {
            event.preventDefault();
            document.onmousemove = null;
            console.log('onmouseup触发')
            dragFlag.value = false;
            console.log("lallalalalla")
            let overlap = overlapDraggedFunc()
            let emitData = {
                originalTop: dragContainer.value.offsetTop,
                originalLeft: dragContainer.value.offsetLeft,
                isOverlap: overlap[0],
                overlapList: overlap[1]
            }
            emit('dragend', emitData)
            document.onmouseup = null;
        }
    }
}
function dragStartTouch(event: TouchEvent) {
    if (props.isDraggable && allowClickFlag) {
        console.log("TouchEvent")
        dragFlag.value = true;
        cur.value.x = event.touches[0].clientX;
        cur.value.y = event.touches[0].clientY;
        originalTop.value = dragContainer.value.offsetTop;
        originalLeft.value = dragContainer.value.offsetLeft;
        offLeft.value = dragContainer.value.offsetLeft;
        offTop.value = dragContainer.value.offsetTop;
        let offx = event.touches[0].clientX - cur.value.x;
        let offy = event.touches[0].clientY - cur.value.y;
        checkBoundary(offx, offy)
        let emitData = {
            curX: event.touches[0].clientX,
            curY: event.touches[0].clientY,
            originalTop: originalTop.value,
            originalLeft: originalLeft.value
        }
        emit('dragstart', emitData)
    }
}

function dragMoveTouch(event: TouchEvent) {
    if (dragFlag.value) {
        let offx = event.touches[0].clientX - cur.value.x;
        let offy = event.touches[0].clientY - cur.value.y;
        checkBoundary(offx, offy)
        let overlap = overlapDraggingFunc()
        let emitData = {
            curX: event.touches[0].clientX,
            curY: event.touches[0].clientY,
            originalTop: dragContainer.value.offsetTop,
            originalLeft: dragContainer.value.offsetLeft,
            isOverlap: overlap[0],
            overlapList: overlap[1]
        }
        emit('dragging', emitData)
    }
    event.preventDefault()
}

function dragEnd() {
    dragFlag.value = false;
    let overlap = overlapDraggedFunc()
    let emitData = {
        isOverlap: overlap[0],
        overlapList: overlap[1]
    }
    emit('dragend', emitData)

}

function checkBoundary(offx: number, offy: number): void {
    let offtemp: number
    let toLeft = offLeft.value + offx;
    let toTop = offTop.value + offy;
    if (props.isCheckBoundary) {
        if (toLeft <= 0) {
            cur.value.x += 0 - offLeft.value;
            offLeft.value = 0;
        } else if (toLeft >= (offtemp = dragLimitedRreaW - dragContainerW.value!)) {
            cur.value.x += offtemp - offLeft.value;
            offLeft.value = offtemp;
        } else {
            cur.value.x += offx;
            offLeft.value = toLeft;

        }
        if (toTop <= 0) {
            cur.value.y += 0 - offTop.value;
            offTop.value = 0;
        } else if (toTop >= (offtemp = dragLimitedRreaH - dragContainerH.value!)) {
            cur.value.y += offtemp - offTop.value;
            offTop.value = offtemp;
        } else {
            cur.value.y += offy;
            offTop.value = toTop;
        }
    } else {
        cur.value.x += offx;
        offLeft.value = toLeft;
        cur.value.y += offy;
        offTop.value = toTop;
    }
}

function checkOverlap(watchedCpnt: Ref<jzVueDragWatchedType>): boolean {
    let dragElRect = getPosition();
    let dragElCoreX = dragElRect.left + dragElRect.width / 2;
    let dragElWidth = dragElRect.width;
    let dragElHeight = dragElRect.height;
    let dragElCoreY = dragElRect.top + dragElRect.height / 2;
    let watchedCpntRect = watchedCpnt.value.getPosition();
    let watchedCpntCoreX = watchedCpntRect.left + watchedCpntRect.width / 2;
    let watchedCpntCoreY = watchedCpntRect.top + watchedCpntRect.height / 2;
    let watchedCpntWidth = watchedCpntRect.width;
    let watchedCpntHeight = watchedCpntRect.height;
    if (Math.abs(dragElCoreX - watchedCpntCoreX) < (dragElWidth / 2 + watchedCpntWidth / 2) && Math.abs(dragElCoreY - watchedCpntCoreY) < (dragElHeight / 2 + watchedCpntHeight / 2)) {
        return true;
    } else {
        return false;
    }
}

function overlapDraggingFunc() {
    let isOverlap: boolean = false;
    let overlapList: Ref<jzVueDragWatchedType>[] = [];
    for (let cpnt of props.watchComponents) {
        if (checkOverlap(cpnt)) {
            console.log('重叠啦啦啦啦啦啦');
            overlapList.push(cpnt)
        }
    }
    return [isOverlap, overlapList]
}
function overlapDraggedFunc() {
    let isOverlap = false;
    let overlapList: Ref<jzVueDragWatchedType>[] = [];
    for (let cpnt of props.watchComponents) {
        if (checkOverlap(cpnt)) {
            console.log('重叠啦啦啦啦啦啦');
            isOverlap = true;
            overlapList.push(cpnt)
        }
    }
    if (props.isReset && !isOverlap) {
        allowClickFlag = false;
        console.log("allowClickFlag:" + allowClickFlag)
        dragContainerTransition.value = '1s'
        console.log('originalTop.value:' + originalTop.value)
        console.log('originalLeft.value:' + originalLeft.value)
        offTop.value = originalTop.value
        offLeft.value = originalLeft.value
        setTimeout(() => {
            dragContainerTransition.value = '0s';
            allowClickFlag = true
            console.log("allowClickFlag:" + allowClickFlag)
        }, 1000)
    }
    return [isOverlap, overlapList]
}
</script>

<style scoped>
.drag-container {
    position: absolute;
    width: fit-content;
    height: fit-content;
    background-color: rgba(128, 128, 128, 0.5);
}
.drag-draggable-area {
    position: absolute;
    background-color: rgba(204, 232, 50, 0.5);
}
</style>