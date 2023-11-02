<script setup lang="ts">
import { computed, reactive } from 'vue';
import RightPanel from './RightPanel.vue'

import type { ClassFormDataType } from './type'

const formData = reactive<ClassFormDataType>({
  colNum: 6,
  rowNum: 3,
  gapNum: 2
})
const handleClassChange = (data: ClassFormDataType) => {
  formData.colNum = data.colNum;
  formData.rowNum = data.rowNum;
  formData.gapNum = data.gapNum;
}

type ColItem = {
  /**
   * 是否有过道
   */
  segment: boolean;
  /**
   * 第几个过道
   */
  segmentIndex: number;
}

const colNum = computed(() => {
  // 分配过道的算法需要根据具体业务来实现，这里提供一个尽可能平均分配的算法
  // 实际最好是可以由用户拖拽过道实现分组
  const result: ColItem[] = Array.from({length: formData.colNum})
  result.forEach((d, index) => {
    result[index] = {segment: false, segmentIndex: 0}
  });


  let segmentSize = Math.floor(result.length / (formData.gapNum + 1))
  let segmentLeft = 0;
  for(let i = 0; i < result.length; i+=segmentSize) {
    const end = i + segmentSize;
    try {
      if (end < result.length && segmentLeft < formData.gapNum) {
        result[end - 1].segment = true;
        result[end - 1].segmentIndex = segmentLeft;
        segmentLeft += 1
      }
    } catch(e) {
      throw new Error('分配过道失败')
    }
  }

  return result;
})

const rowNum = computed(() => {
  return new Array(formData.rowNum).fill(0)
})


const showAisle = (colIndex: number) => {
  return colNum.value[colIndex].segment;
}
</script>
<template>
<div class="wrap">
  <div class="board-wrap">
    <div class="board">
      <!-- 这里不建议使用index作为key -->
      <div class="stu-col" v-for="(col, colIndex) in colNum" :key="colIndex">
        <div>
          <div class="stu-group-leader">第{{ colIndex + 1 }}组</div>
          <div v-for="(row, rowIndex) in rowNum" :key="rowIndex" class="stu"></div>
        </div>
        <div class="aisle" v-if="showAisle(colIndex)">过道 <b>{{ col.segmentIndex + 1 }}</b></div>
      </div>
    </div>
  </div>
  <div class="panel-wrap"><RightPanel @changed="handleClassChange"/></div>
</div>
</template>
<style scoped>
.wrap {
  display: flex;
  margin-top: 100px;
}
.board-wrap {
  background: white;
  width: 75%;
  flex-shrink: 0;
  overflow-x: auto;
  max-width: 80vw;
  padding-right: 24px;
  margin-right: 24px;
}
.panel-wrap {
  width: 20%;
}
.board {
  padding: 24px;
  display: flex;
}
.stu-col {
  display: flex;
  margin-left: 8px;
}
.stu {
  width: 120px;
  height: 60px;
  background: #DEE4E9;
  border-radius: 4px;
  margin-bottom: 8px;
  margin-right: 8px;
}
.stu-group-leader {
  border: 1px solid #EEEEEE;
  line-height: 24px;
  height: 24px;
  text-align: center;
  margin-bottom: 8px;
  margin-right: 8px;
  border-radius: 4px;
}
.aisle {
  width: 80px;
  background: grey;
  border-radius: 4px;
  margin: 0 24px;
  color: white;
  writing-mode: vertical-rl;
  display: flex;
  align-items: center;
  justify-content: center;
}

.aisle b {
  text-combine-upright: all;
  margin: 4px 0;
}
</style>