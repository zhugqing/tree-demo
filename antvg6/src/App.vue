<template>
  <!-- <Tree /> -->
  <!-- <div id="container">sssss</div> -->
  <Tree1 />
  <!-- <Tree2 /> -->
  <!-- <Tree3 /> -->
</template>

<script setup lang="ts">
// import Tree from "./tree-huo.vue";
import Tree1 from "./tree1.vue";
// import Tree2 from "./tree2.vue";
// import Tree3 from "./tree3.vue";
</script>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
