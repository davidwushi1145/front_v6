<script setup lang="ts">
import type { Ref } from 'vue';
import { ref } from 'vue';
import { UploadFilled } from '@element-plus/icons-vue'
import Header from "@/components/Header.vue";
import axios from "axios";
import type { AxiosProgressEvent } from 'axios';

let src: Ref<string> = ref('');
let progress: Ref<number> = ref(0); // 定义一个响应式引用
let algorithmName: Ref<string> = ref('alpha'); // 定义一个响应式引用

const handleSuccess = (response: Blob) => {
  const blob = new Blob([response], { type: 'image/png' });
  src.value = URL.createObjectURL(blob);
};

const beforeUpload = (file: File) => {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('algorithm_name', algorithmName.value); // 添加挖掘算法到表单数据

  axios.post('http://localhost:1145/upload', formData, {
    headers: {
      'Content-Type': 'multipart/form-data'
    },
    responseType: 'blob',
    onUploadProgress: (progressEvent: AxiosProgressEvent) => {
      if (progressEvent.total) {
        progress.value = Math.round((progressEvent.loaded * 50) / progressEvent.total); // 更新进度条的值，上传过程占50%
      }
    },
    onDownloadProgress: (progressEvent: AxiosProgressEvent) => {
      if (progressEvent.total) {
        progress.value = 50 + Math.round((progressEvent.loaded * 50) / progressEvent.total); // 更新进度条的值，下载过程占50%
      }
    }
  }).then(response => {
    handleSuccess(response.data);
    progress.value = 100; // 当文件处理完成并返回结果时，将进度条的值设置为100%
  });

  return false;
};
</script>

<template>
  <div class="common-layout">
    <el-container>
      <el-header style="flex: auto"><Header/></el-header>
      <el-main>
        <el-select v-model="algorithmName" placeholder="请选择挖掘算法" class="select"> <!-- 添加下拉菜单 -->
          <el-option label="alpha" value="alpha"></el-option>
          <el-option label="heuristic" value="heuristic"></el-option>
        </el-select>
        <el-upload
            class="upload-demo"
            drag
            action="http://localhost:1145/upload"
            multiple
            :on-success="handleSuccess"
            :before-upload="beforeUpload"
        >
          <el-icon class="el-icon--upload"><upload-filled /></el-icon>
          <div class="el-upload__text">
            Drop file here or <em>click to upload</em>
          </div>
          <template #tip>
            <div class="el-upload__tip">
              挖掘需要时间，请耐心等待
            </div>
          </template>
        </el-upload>
        <el-progress :percentage="progress"></el-progress> <!-- 添加进度条 -->
        <div class="demo-image__placeholder">
          <div class="block">
            <span class="demonstration">挖掘结果</span>
            <el-image :src="src" slot="等待上传">
              <template #error>
                <div class="image-slot">Loading<span class="dot">...</span></div>
              </template>
            </el-image>
          </div>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<style>
.demo-image__error .image-slot {
  font-size: 30px;
}
.common-layout {
  display: flex;        /* 使用Flexbox */
  justify-content: center; /* 水平居中 */
  align-items: center;     /* 垂直居中 */
  text-align: center;  /* 文本居中 */
  width: 89vw;
}
.demo-image__placeholder .demonstration {
  display: block;
  color: var(--el-text-color-secondary);
  font-size: 80px;
  margin-bottom: 20px;
}
.demo-image__error .image-slot .el-icon {
  font-size: 30px;
}
.demo-image__error .el-image {
  width: 100%;
  height: 200px;
}
.select {
  width: 200px;
  margin-bottom: 20px;
}
</style>
