<template>
  <div class="on-scan">
    <el-input
      type="password"
      v-model="selected"
      ref="ele"
      @blur="state.focus = false"
      @keyup.enter="actions.blur"
    ></el-input>
    <el-input
      :class="{ focus: state.focus }"
      v-model="selected"
      ref="eleInput"
      @focus="actions.putInFocus"
      readonly
    ></el-input>
  </div>
</template>
<script lang="ts" setup>
import type { ElInput } from 'element-plus'
import { reactive, computed, onMounted, onUnmounted, ref, watch } from 'vue'
import onScan from 'onscan.js'
const ele = ref<InstanceType<typeof ElInput>>()
const eleInput = ref<InstanceType<typeof ElInput>>()
let bindEle = ref(null)
let $emits = defineEmits(['update:modelValue', 'blur'])
const props = defineProps({
  modelValue: {
    type: [String, Number],
    default: '',
  },
  autofocus: {
    type: Boolean,
    default: false,
  },
  filter: {
    type: Boolean,
    default: false,
  },
})
const selected = computed({
  get: () => props.modelValue,
  set: (val) => $emits('update:modelValue', val),
})
const state = reactive({
  focus: false,
})
watch(
  () => props.autofocus,
  (newValue, oldValue) => {
    if (newValue) {
      actions.putInFocus()
    }
  }
)
onMounted(() => {
  if (props.autofocus) {
    actions.putInFocus()
  }
  bindEle.value = ele.value?.ref
  let scanOptions = {
    onScan: function (sCode: any, iQty: any) {
      console.log('on scan components:', sCode, iQty)
      selected.value = sCode
      ele.value?.select()
      // actions.blur({})
    },
    onScanError() {},
  }
  try {
    onScan.attachTo(bindEle.value, scanOptions)
  } catch (e) {
    onScan.setOptions(bindEle.value, scanOptions)
  }
})
onUnmounted(() => {
  onScan.detachFrom(bindEle.value)
})
const keyupLastTime = ref('')
const realBarcode = ref('')
const barcode = ref('')
const actions = {
  blur(row: any) {
    if (/\S/.test(selected.value)) {
      $emits('blur', selected.value, { from: 'singleScan' })
    }
  },
  focus() {
    ele.value?.focus()
  },
  putInFocus() {
    actions.focus()
    state.focus = true
  },
}
defineExpose({
  actions: actions,
})
</script>

<style lang="scss" scoped>
.on-scan {
  position: relative;
  .el-input:last-child {
    position: absolute;
    left: 0;
    top: 1px;
  }
  .focus {
    :deep(.el-input__wrapper) {
      box-shadow: 0 0 0 1px var(--el-input-focus-border-color) inset;
      background-color: var(--el-color-warning-light-8);
    }
  }
}
</style>
