<template>
  <div class="flex flex-col h-screen w-screen justify-center items-center">
    <el-upload drag action :auto-upload="false" @change="loadFile">
      <el-icon><upload-filled /></el-icon>
      <div>拖拽文件到此或 <em>单击此处上传</em></div>
      <template #tip>
        <div class="mt-2 text-center text-xs">文件大小上限为500MB</div>
      </template>
    </el-upload>
    <el-button @click="submit" type="primary">上传</el-button>
    <div class="w-full px-4">
      <el-progress
        :percentage="percentage"
        :status="percentage == 100 ? 'success' : ''"
      />
    </div>
  </div>
</template>

<script setup>
import SparkMD5 from "spark-md5";
const fileChunkList = ref([]);
const fileHash = ref("");
const percentage = ref(0);

const loadFile = async (file) => {
  const raw_file = file.raw;
  try {
    const buffer = await fileToBuffer(raw_file);
    // 将文件按固定大小（1M）进行切片，注意此处同时声明了多个常量
    const chunkSize = 51200,
      chunkList = [], // 保存所有切片的数组
      chunkListLength = Math.ceil(raw_file.size / chunkSize), // 计算总共多个切片
      suffix = /\.([0-9A-z]+)$/.exec(raw_file.name)[1]; // 文件后缀名
    // 根据文件内容生成 hash 值
    const spark = new SparkMD5.ArrayBuffer();
    spark.append(buffer);
    const hash = spark.end();
    // 生成切片，这里后端要求传递的参数为字节数据块（chunk）和每个数据块的文件名（fileName）
    let curChunk = 0;
    for (let i = 0; i < chunkListLength; i++) {
      const item = {
        chunk: raw_file.slice(curChunk, curChunk + chunkSize),
        fileName: `${hash}_${i}.${suffix}`,
      };
      curChunk += chunkSize;
      chunkList.push(item);
    }
    fileChunkList.value = chunkList;
    fileHash.value = hash;
    console.log(fileChunkList.value, fileHash.value);
  } catch (e) {
    ElMessage.error("文件转换失败");
  }
};

const fileToBuffer = (file) => {
  return new Promise((resolve, reject) => {
    const fr = new FileReader();
    fr.onload = (e) => {
      resolve(e.target.result);
    };
    fr.readAsArrayBuffer(file);
    fr.onerror = () => {
      reject(new Error("转换文件格式发生错误"));
    };
  });
};

// 模拟上传的过程
const sendFile = (chunk, index) => {
  return new Promise((resolve, reject) => {
    console.log(`处理第${index}块`, index, fileChunkList.value.length);
    percentage.value = parseInt(
      (index / (fileChunkList.value.length - 1)) * 100
    );
    console.log(percentage.value);
    setTimeout(() => {
      console.log(chunk.fileName);
      resolve(chunk.fileName);
    }, 100);
  });
};

const submit = async () => {
  let index = 0;
  try {
    for (let chunk of fileChunkList.value) {
      await sendFile(chunk, index);
      index += 1;
    }
    await submitComplete();
  } catch (e) {
    ElMessage.error(`第${index}块分片上传失败， 请重试`);
    console.log(e);
  }
};

const submitComplete = async () => {
  console.log("上传完成");
};
</script>