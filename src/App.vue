<template>
  <div class="container">
    <jzVueDragLimitedAreaVue :dragLimitedRreaW="1000" :dragLimitedRreaH="600" ref="test">
      <jzVueDragWatched
        ref="de1"
        :zIndex="2"
        checkOverlapArea="innerArea"
        innerAreaW="50%"
        innerAreaH="50%"
        innerAreaT="10%"
        innerAreaL="10%"
      >
        <div class="test2"></div>
      </jzVueDragWatched>
      <jzVueDragWatched ref="de2" :zIndex="2" class="iiii">
        <div class="test2"></div>
      </jzVueDragWatched>
      <jzVueDragEl
        :watchComponents="watchComponents"
        :isReset="true"
        class="lalalal"
        @dragging="dragging"
        @dragstart="dragstart"
      >
        <div class="test3"></div>
      </jzVueDragEl>
      <jzVueDragEl
        :watchComponents="watchComponents"
        :isReset="true"
        class="tttttt"
        :draggableAreaW="'50%'"
        :draggableAreaH="'50%'"
        :draggableAreaT="'20%'"
        :draggableAreaL="'20%'"
        :checkOverlapArea="'draggableArea'"
      >
        <div class="test3"></div>
      </jzVueDragEl>
    </jzVueDragLimitedAreaVue>
  </div>
</template>

<script setup lang="ts">
import { Ref, ref, onMounted } from 'vue'
import jzVueDragEl from './components/jzVueDragEl.vue';
import jzVueDragLimitedAreaVue from './components/jzVueDragLimitedArea.vue';
import jzVueDragWatched from './components/jzVueDragWatched.vue'
const test = ref()
const de1 = ref()
const de2 = ref()
const watchComponents = [de1, de2]

function dragging(event) {
  // console.log(event)
}
function dragstart(event) {
  console.log(event)
}
</script>

<style scoped>
.container {
  position: relative;
  width: 900px;
  height: 600px;
}
.test2 {
  width: 200px;
  height: 200px;
}
.test3 {
  width: 100px;
  height: 100px;
}
.lalalal {
  top: 250px;
  left: 250px;
}
.tttttt {
  top: 250px;
  left: 400px;
}
.iiii {
  top: 300px;
}
</style>