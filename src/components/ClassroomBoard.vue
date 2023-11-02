<script setup lang="ts">
import { reactive, ref, watchEffect } from 'vue';
import { type UseDraggableReturn, VueDraggable } from 'vue-draggable-plus'
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

type RowItem = {
  name: string;
}

type ColItem = {
  /**
   * 是否为过道
   */
  segment: boolean;
  /**
   * 第几个过道
   */
  segmentIndex: number;
  /**
   * 第几组 
   */
  groupIndex: number;
  rowList: RowItem[];
  id: number;
}

const colList = ref([] as ColItem[])

watchEffect(() => {
  // 分配过道的算法需要根据具体业务来实现，这里提供一个尽可能平均分配的算法
  // 实际最好是可以由用户拖拽过道实现分组
  const result: ColItem[] = Array.from({ length: formData.colNum })
  result.forEach((d, index) => {
    const rowList = Array.from({ length: formData.rowNum }) as RowItem[]
    rowList.fill({ name: '' })
    rowList.forEach((r, rIndex) => {
      // 实际业务中学生应该是一个对象，有自己的id，这里直接用名字当作id使用
      rowList[rIndex] = { name: `张${index}${rIndex}` }
    })
    result[index] = { segment: false, segmentIndex: 0, id: index, groupIndex: index, rowList }
  });


  let segmentSize = Math.floor(result.length / (formData.gapNum + 1))
  let segmentLeft = 0;
  for (let i = 0; i < result.length; i += segmentSize) {
    const end = i + segmentSize;
    try {
      if (end < result.length && segmentLeft < formData.gapNum) {
        // 添加过道
        result.splice(end + segmentLeft, 0, { segment: true, segmentIndex: segmentLeft, id: result.length + 1, groupIndex: end, rowList: [] })
        segmentLeft += 1
      }
    } catch (e) {
      throw new Error('分配过道失败')
    }
  }

  colList.value = result;
});

const el = ref<UseDraggableReturn>();

const handleUpdate = () => {
  // 重新排序
  colList.value.filter((col) =>
    col.segment
  ).forEach((s, sIndex) => {
    s.segmentIndex = sIndex;
  })

  colList.value.filter((col) => !col.segment).forEach((g, gIndex) => {
    g.groupIndex = gIndex;
  })
}
</script>
<template>
  <div class="wrap">
    <div class="board-wrap">
      <VueDraggable v-model="colList" ref="el" class="board" handle=".col-handle" @update="handleUpdate" :animation="150"
        group="col">
        <div class="stu-col list" v-for="col in colList" :key="col.id">
          <div v-show="!col.segment">
            <div class="stu-group-leader col-handle">第{{ col.groupIndex + 1 }}组</div>
            <VueDraggable v-model="col.rowList" :animation="150" group="row" class="col-content col-handle">
              <div v-for="row in col.rowList" :key="row.name" class="stu">{{ row.name }}</div>
            </VueDraggable>
          </div>
          <div class="aisle col-handle" v-show="col.segment">过道 <b>{{ col.segmentIndex + 1 }}</b></div>
        </div>
      </VueDraggable>
    </div>
    <div class="panel-wrap">
      <RightPanel @changed="handleClassChange" />
    </div>
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
  display: flex;
  justify-content: center;
  align-items: center;
  background: #DEE4E9;
  border-radius: 4px;
  margin-bottom: 8px;
  cursor: move;
}

.col-handle {
  cursor: move;
}

.col-content {
  min-height: 400px;
  border: 1px solid #eee;
  border-radius: 4px;
  padding: 8px;
}

.stu-group-leader {
  min-width: 138px;
  border: 1px solid #EEEEEE;
  line-height: 24px;
  text-align: center;
  margin-bottom: 8px;
  border-radius: 4px;
  padding: 8px;
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