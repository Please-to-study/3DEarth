<template>
  <mars-dialog :visible="true" right="10" top="10" width="330">
    <graphic-layer-state :defaultCount="100" :interaction="false" :openEditor="true" />
  </mars-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref, toRaw, onMounted } from "vue"
import GraphicLayerState from "@mars/components/mars-sample/graphic-layer-state.vue"
import * as mapWork from "./map.js"
</script>
