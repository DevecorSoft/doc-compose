<script setup lang="ts">
import content from "@/components/content.vue";
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
      <n-message-provider>
        <content />
      </n-message-provider>
    </n-layout-content>
  </n-layout>
</template>
