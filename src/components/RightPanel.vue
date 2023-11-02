<script setup lang="ts">
import { Form, FormItem, InputNumber } from 'ant-design-vue';
import { onMounted, reactive } from 'vue';

const emits = defineEmits<{(event: 'changed', data: typeof formData): void}>()

const formData = reactive({
  colNum: 6,
  rowNum: 3,
  gapNum: 2,
}) // 这个表单数据应该放在pinia或vuex里

onMounted(() => {
  emits('changed', formData)
})
</script>
<template>
  <Form :model="formData">
    <FormItem label="列数" name="colNum"><InputNumber v-model:value="formData.colNum" @change="emits('changed', formData)"></InputNumber> </FormItem>
    <FormItem label="每列人数" name="rowNum"><InputNumber v-model:value="formData.rowNum" @change="emits('changed', formData)" ></InputNumber></FormItem>
    <FormItem label="过道数" name="gapNum"><InputNumber v-model:value="formData.gapNum" :max="formData.colNum - 1" @change="emits('changed', formData)"></InputNumber></FormItem>
  </Form>
</template>
<style scoped>

</style>