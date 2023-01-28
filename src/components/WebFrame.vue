<script setup lang="ts">
import { ref, onMounted, watch } from "vue";

const emit = defineEmits<{
  (e: "insert_images", images: Array<URL>): void;
}>();
var base_url =
  import.meta.env.MODE === "development"
    ? new URL("./example/", location.href)
    : new URL("./", location.href);

var iframe = ref<HTMLIFrameElement>();

var load_page = (src: URL) => {
  import.meta.env.MODE === "development" && (src.href += "index.html");

  fetch(src)
    .then((resp) => {
      return resp.text();
    })
    .then((html) => {
      if (!iframe.value) return;
      iframe.value.srcdoc = html
        .replace("filterEl.focus();", "")
        .replace("font-size: 14px;", "font-size: 18px;")
        .replace("<style>", "<style>a{white-space:nowrap;}");
    });
};

function load_image() {
  if (!iframe.value) return;
  const iframeDoc = iframe.value.contentDocument as Document;
  var images = [...iframeDoc.querySelectorAll("a")]
    .map((ele) => new URL(ele.getAttribute("href") + "", base_url))
    .filter((link) =>
      /[.](jpg|jpeg|jfif|jfi|jif|png|bmp|gif|svg|webp)$/i.test(link + "")
    );

  emit("insert_images", images);
}

onMounted(() => {
  load_page(base_url);
});

watch(iframe, (iframe) => {
  if (!iframe) return;
  iframe.addEventListener("load", () => {
    const iframeDoc = iframe.contentDocument as Document;
    iframeDoc.addEventListener("click", (e: MouseEvent) => {
      e.preventDefault();
      const Element = iframeDoc.defaultView?.Element || window.Element;
      if (!(e.target instanceof Element)) return;
      const a = e.target.closest("a");
      const href = a?.getAttribute("href");
      const isFile = (path: string | null | undefined) => {
        if (a?.classList.contains("icon-directory")) return false; // Live Server
        if (/[/]$/.test(path + "")) return false; // rclone serve
        if (path === "..") return false;
        return true;
      };
      if (!href || isFile(href)) return;

      base_url = new URL(href, base_url);
      load_page(base_url);
    });
    load_image();
  });
});
</script>

<template>
  <iframe
    ref="iframe"
    src="about:blank"
    frameborder="0"
    style="width: 100%; height: 100%"
  ></iframe>
</template>
