<script setup lang="ts">
import {
  NGlobalStyle,
  NConfigProvider,
  darkTheme,
  zhTW,
  dateZhTW,
} from "naive-ui";
import HomePage from "@/components/HomePage.vue";
import { store } from "@/store";
</script>

<template>
  <n-config-provider
    :locale="zhTW"
    :date-locale="dateZhTW"
    :theme="store.isDarkTheme ? darkTheme : null"
  >
    <HomePage />
    <n-global-style />
  </n-config-provider>
</template>

<style>
html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
#app {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
</style>
