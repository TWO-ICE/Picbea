<template>
  <div class="body" @paste="handlePaste">
    <h1>截图美化 ctrl+v</h1>
    <div class="box">
      <div v-if="base64Images.length > 0" ref="downloadDiv">
        <div v-for="(base64Image, index) in base64Images" :key="index" class="wrap">
          <img :src="base64Image" alt="Pasted Image">
        </div>
      </div>
    </div>
    <button @click="downloadImage" v-show="result">保存图片</button>
  </div>
</template>

<script>
import html2canvas from 'html2canvas';
export default {
  data() {
    return {
      base64Images: [],
      result: false,
    };
  },
  methods: {
    handlePaste(event) {
      const items = (event.clipboardData || event.originalEvent.clipboardData).items;
      const imageItems = Array.from(items).filter(item => item.type.indexOf('image') === 0);

      if (imageItems.length === 0) {
        return;
      }

      this.base64Images = [];

      const loadImage = (blob) => {
        const reader = new FileReader();

        reader.onload = () => {
          this.base64Images.push(reader.result);
          if (this.base64Images.length === imageItems.length) {
            this.$forceUpdate(); // 强制刷新视图
          }
        };

        reader.readAsDataURL(blob);
      };

      imageItems.forEach(item => {
        const blob = item.getAsFile();
        if (blob) {
          loadImage(blob);
        }
      });

      this.result = true;
    },
    async downloadImage() {
      await this.$nextTick(); // 确保组件渲染完成

      const div = this.$refs.downloadDiv; // 获取要下载的<div>的引用
      const canvas = await html2canvas(div);

      // 将Canvas转换为DataURL
      const imageDataURL = canvas.toDataURL('image/png');

      // 创建一个下载链接并模拟点击
      const a = document.createElement('a');
      a.href = imageDataURL;
      a.download = 'downloaded_image.png';
      a.click();
    }
  }
};
</script>
