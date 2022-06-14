<script setup lang="ts">
import hljs from "highlight.js/lib/core";
import xml from "highlight.js/lib/languages/xml";
import { PrismEditor } from "vue-prism-editor";
import "vue-prism-editor/dist/prismeditor.min.css";
import "highlight.js/styles/base16/tomorrow.css";
import { ref } from "vue";
import content from "@/components/content.vue";

const code = ref(
  `<div style="
max-width: 960px;
background-color: #F8FAFB;
margin-left: auto;
margin-right: auto;
"
>
{{ content }}
</div>`
);
const mime_data = ref<Object>({});

hljs.registerLanguage("xml", xml);

function highlighter(code: string) {
  return hljs.highlight(code, { language: "xml" }).value;
}

function onpaste(event: ClipboardEvent) {
  const clip_items = {};
  event.clipboardData?.types.forEach((t) => {
    Object.assign(clip_items, { [t]: event.clipboardData?.getData(t) });
  });
  mime_data.value = clip_items;
  event.preventDefault();
}

function compose(template: string, mdata: string, mtype = "text/html") {
  const data_html = mdata.slice(22);
  const rich_text = template.replace(/\{\{\s*content\s*\}\}/, data_html);

  const dom_parser = new DOMParser();
  const xml_serializer = new XMLSerializer();
  const dom = dom_parser.parseFromString(
    `<div id="dc-content">${rich_text}</div>`,
    "text/html"
  );

  const svg_elements = dom.getElementsByTagName("svg");

  for (let i = 0; i < svg_elements.length; i++) {
    svg_elements.item(i)?.replaceWith(
      (() => {
        const item_xml = xml_serializer.serializeToString(
          svg_elements.item(i)!
        );
        const img_node = document.createElement("img");
        img_node.setAttribute(
          "src",
          `data:image/svg+xml;base64,${btoa(item_xml)}`
        );
        return img_node;
      })()
    );
  }

  const enhanced_text = xml_serializer.serializeToString(
    dom.getElementById("dc-content")?.firstChild!
  );

  mime_data.value = {
    [mtype]: enhanced_text,
  };

  const blob_rich = new Blob([enhanced_text], { type: mtype });
  const data = [new ClipboardItem({ [mtype]: blob_rich })];
  navigator.clipboard
    .write(data)
    .then(() => {
      window.$message?.success("Congratulation! compose finished.");
    })
    .catch((err) => {
      console.error(err);
      window.$message?.error("Please authorize.");
    });
}

function onclick() {
  compose(code.value, mime_data.value["text/html"]);
}
</script>

<template>
  <n-layout>
    <n-layout-header>
      <a href="https://github.com/DevecorSoft/doc-compose"
        ><img
          loading="lazy"
          width="149"
          height="149"
          src="https://github.blog/wp-content/uploads/2008/12/forkme_left_green_007200.png?resize=149%2C149"
          class="attachment-full size-full"
          alt="Fork me on GitHub"
          data-recalc-dims="1"
      /></a>
    </n-layout-header>
    <n-layout-content>
      <n-space justify="center">
        <n-card>
          <n-input @paste="onpaste" placeholder="⌘ + v"></n-input>
          <n-tabs type="line" animated>
            <n-tab-pane
              v-for="(mdata, mtype) in mime_data"
              :name="mtype"
              :tab="mtype"
              display-directive="show"
            >
              <n-input
                type="textarea"
                placeholder=""
                clearable
                autofocus
                readonly
                show-count
                :autosize="{
                  minRows: 3,
                }"
                style="min-width: 30rem"
                @paste="onpaste"
                :value="mdata"
              ></n-input>
              <div v-html="mdata"></div>
            </n-tab-pane>
          </n-tabs>
        </n-card>
        <n-space vertical>
          <n-card>
            <prism-editor
              class="my-editor"
              v-model="code"
              :highlight="highlighter"
              line-numbers
            ></prism-editor>
          </n-card>
          <n-button secondary type="primary" @click="onclick">compose</n-button>
        </n-space>
      </n-space>
    </n-layout-content>
  </n-layout>
  <n-message-provider>
    <content />
  </n-message-provider>
</template>

<style>
textarea.prism-editor__textarea {
  outline: 0px;
}
</style>
