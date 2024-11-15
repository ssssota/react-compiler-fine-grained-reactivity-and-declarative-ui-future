<template>
  <div v-html="svg"></div>
</template>

<script setup lang="ts">
import { exportToSvg } from "@excalidraw/excalidraw/dist/excalidraw.production.min.js";
import { onMounted, ref } from "vue";
const svg = ref<string>("");

interface AppState {
  exportWithDarkMode: boolean;
  exportBackground: boolean;
}
interface Props {
  drawFilePath: string;
}

const props = defineProps<Props>();

onMounted(() => {
  loadJsonAndExport(props);
});

const loadJsonAndExport = async ({ drawFilePath: path }: Props) => {
  try {
    const url = new URL(path, window.location.origin + import.meta.env.BASE_URL)
      .href;
    const json = await (await fetch(url)).json();

    const svgElement = await exportToSvg(json);
    svgElement.style.maxWidth = "100%";
    svgElement.style.height = "auto";

    svg.value = omitFontFamily(svgElement.outerHTML);
  } catch (error) {
    console.error("Failed to load JSON or export to SVG", error);
  }
};

function omitFontFamily(svg: string) {
  return svg.replace(/font-family="[^"]+?"/g, "");
}
</script>

<style scoped>
/* custom */
@font-face {
  font-family: "Excalidraw";
  src: url("./Excalifont-Regular.woff2") format("woff2");
}
div {
  font-family: "Excalidraw";
}
</style>
