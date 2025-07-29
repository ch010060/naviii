<script setup lang="ts">
import { ref } from "vue";
import {
  NDrawerContent,
  NTabs,
  NTabPane,
  NInput,
  NInputGroup,
  NInputGroupLabel,
  NButton,
  NSpace,
  NSelect,
  NUpload,
  useMessage,
} from "naive-ui";
import type { UploadFileInfo } from "naive-ui";
import { oneDark } from "@codemirror/theme-one-dark";
import { store } from "@/store";
import { LocalDB } from "@/assets/ts/db";

const message = useMessage();
const db = new LocalDB();
let inputIconAPI = ref(store.iconAPI);
let inputBookMarks = ref(JSON.stringify(store.bookMarks, null, 2));
let searchSelectValue = ref(store.searchJump);
let searchSelectOptions = [
  {
    label: "在目前頁面打開搜索頁",
    value: "_self",
  },
  {
    label: "在新頁面打開搜索頁",
    value: "_blank",
  },
];
let openSelectValue = ref(store.openBookMarkJump);
let openSelectOptions = [
  {
    label: "在目前頁面打開書籤頁",
    value: "_self",
  },
  {
    label: "在新頁面打開書籤頁",
    value: "_blank",
  },
];

const saveData = () => {
  try {
    store.setSearchJump(searchSelectValue.value);
    store.setOpenBookMarkJump(openSelectValue.value);
    if (inputIconAPI.value[inputIconAPI.value.length - 1] != "/") {
      inputIconAPI.value += "/";
    }
    store.setIconAPI(inputIconAPI.value);
    if (inputIconAPI.value == "") {
      inputIconAPI.value = store.iconAPI;
    }
    try {
      store.setBookMarks(inputBookMarks.value);
    } catch (e) {
      const msg = "儲存失敗！書籤Json數據格式錯誤：" + e;
      console.log(msg);
      message.error(msg, {
        duration: 8000,
      });
      return;
    }
    if (inputBookMarks.value == "") {
      inputBookMarks.value = JSON.stringify(store.bookMarks, null, 2);
    }
    message.success("儲存成功！");
  } catch (e) {
    const msg = "儲存失敗！" + e;
    console.log(msg);
    message.error(msg, {
      duration: 8000,
    });
  }
};

const clearData = () => {
  db.clear();
  message.success("數據已清除！即將重新整理頁面~");
  setTimeout(() => {
    location.reload();
  }, 3000);
};

const exportData = () => {
  db.exportToJson();
};

const importData = (fileList: UploadFileInfo[]) => {
  if (fileList.length && fileList[0].file) {
    const file = fileList[0].file;
    const reader = new FileReader();
    reader.readAsText(file, "UTF-8");
    reader.onload = () => {
      if (typeof reader.result == "string") {
        try {
          let data = JSON.parse(reader.result);
          for (let key in data) {
            if (db.keys.includes(key)) {
              db.setItem(key, data[key]);
            }
          }
          message.success("數據匯入成功！即將重新整理頁面~");
          setTimeout(() => {
            location.reload();
          }, 3000);
        } catch (e) {
          const msg = "數據匯入失敗！" + e;
          console.log(msg);
          message.error(msg, {
            duration: 8000,
          });
        }
      }
    };
  }
};
</script>

<template>
  <n-drawer-content :native-scrollbar="false">
    <template #header> 設定 </template>
    <n-tabs type="segment">
      <n-tab-pane name="Base" tab="基礎">
        <n-space vertical>
          <n-input-group>
            <n-input-group-label>iconAPI</n-input-group-label>
            <n-input v-model:value="inputIconAPI" />
          </n-input-group>
          <n-select
            v-model:value="searchSelectValue"
            :options="searchSelectOptions"
          />
          <n-select
            v-model:value="openSelectValue"
            :options="openSelectOptions"
          />
          <n-upload
            class="base-item"
            :trigger-style="{ width: '100%' }"
            :show-file-list="false"
            @update:file-list="importData"
          >
            <n-button class="base-item" strong secondary type="info">
              匯入數據
            </n-button>
          </n-upload>
          <n-button
            class="base-item"
            strong
            secondary
            type="info"
            @click="exportData"
          >
            導出數據
          </n-button>
          <n-button
            class="base-item"
            strong
            secondary
            type="error"
            @click="clearData"
          >
            清除數據
          </n-button>
        </n-space>
      </n-tab-pane>
      <n-tab-pane name="BookMarks" tab="書籤">
        <VueCodemirror
          v-model="inputBookMarks"
          :extensions="store.isDarkTheme ? [oneDark] : []"
        />
      </n-tab-pane>
    </n-tabs>
    <template #footer>
      <n-button
        class="base-item"
        strong
        secondary
        type="primary"
        @click="saveData"
      >
        儲存
      </n-button>
    </template>
  </n-drawer-content>
</template>

<style scoped>
.base-item {
  width: 100%;
}
</style>
