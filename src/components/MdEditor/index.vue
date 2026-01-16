<template>
  <div class="md-editor-container">
    <md-editor
      v-model="text"
      :theme="theme"
      :toolbars="toolbars"
      @onUploadImg="handleUploadImg"
      @onChange="handleChange"
      :placeholder="placeholder"
      style="height: 600px"
    />
  </div>
</template>

<script setup name="MyMdEditor">
import { ref, watch, computed } from "vue";
import { MdEditor } from "md-editor-v3";
import "md-editor-v3/lib/style.css";
import useAppStore from "@/store/modules/app";
import { uploadFile } from "@/api/common"; // 若依通用的上传接口

const props = defineProps({
  modelValue: {
    type: String,
    default: "",
  },
  placeholder: {
    type: String,
    default: "请开始你的创作...",
  },
});

const emit = defineEmits(["update:modelValue", "change"]);
const appStore = useAppStore();

// 1. 实现双向绑定
const text = ref(props.modelValue);
watch(
  () => props.modelValue,
  (val) => {
    text.value = val;
  }
);

const handleChange = (val) => {
  emit("update:modelValue", val);
  emit("change", val);
};

// 2. 自动跟随若依的暗黑模式
const theme = computed(() => {
  // 若依的布局设置中如果开启了暗黑模式，appStore 中通常有记录
  // 或者直接通过 document.documentElement.classList 判断
  return appStore.sideTheme === "theme-dark" ? "dark" : "light";
});

// 3. 图片上传逻辑：对接若依 CommonController
const handleUploadImg = async (files, callback) => {
  const res = await Promise.all(
    files.map((file) => {
      return new Promise((rev, rej) => {
        const form = new FormData();
        form.append("file", file);
        // 调用若依通用的上传接口 (后端对应 CommonController.uploadFile)
        uploadFile(form)
          .then((res) => {
            // 注意：res.url 是后端返回的图片相对路径或绝对路径
            rev(res.url);
          })
          .catch((err) => rej(err));
      });
    })
  );
  // 将上传成功的地址插入到编辑器中
  callback(res);
};

// 4. 自定义工具栏配置 (可根据喜好删减)
const toolbars = [
  "bold",
  "underline",
  "italic",
  "-",
  "title",
  "strikeThrough",
  "sub",
  "sup",
  "quote",
  "unorderedList",
  "orderedList",
  "task",
  "-",
  "codeRow",
  "code",
  "link",
  "image",
  "table",
  "mermaid",
  "katex",
  "-",
  "revoke",
  "next",
  "save",
  "=",
  "pageFullscreen",
  "fullscreen",
  "preview",
  "htmlPreview",
  "catalog",
];
</script>

<style scoped>
.md-editor-container {
  width: 100%;
  border: 1px solid var(--el-border-color);
  border-radius: 4px;
}
</style>
