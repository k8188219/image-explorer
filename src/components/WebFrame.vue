<script setup lang="ts">
import { ref, onMounted, watch } from "vue";

const emit = defineEmits<{
  (e: "insert_images", images: Array<URL>): void;
}>();
var base_url =
  import.meta.env.MODE === "development"
    ? new URL("./example/", location.href)
    : new URL("./", location.href);

var iframeRef = ref<HTMLIFrameElement>();

var load_page = (src: URL) => {
  import.meta.env.MODE === "development" && (src.href += "index.html");
  const iframe = iframeRef.value;
  if (!iframe) return;

  fetch(src)
    .then((resp) => resp.text())
    .then((html) =>
      html
        .replace("font-size: 14px;", "font-size: 18px;")
        .replace("<style>", "<style>a{white-space:nowrap;}")
    )
    .then((html) => {
      const t = document.createElement("template");
      t.innerHTML = html;
      return t.content;
    })
    .then(async (content) => {
      iframe.contentWindow?.location.reload();
      await new Promise((resolve) => (iframe.onload = resolve));
      iframe.contentDocument?.body.append(content);
    });
};

function load_image() {
  const iframe = iframeRef.value;
  if (!iframe) return;
  const iframeDoc = iframe.contentDocument as Document;
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

watch(iframeRef, (iframe) => {
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
    :ref="(el) => (iframeRef = el as HTMLIFrameElement)"
    src="about:blank"
    frameborder="0"
    style="width: 100%; height: 100%"
  ></iframe>
</template>
