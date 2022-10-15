<script setup lang="ts">
import MMPSearch from "./components/MMPSearch.vue";
</script>

<template>
  <div class="wrapper" id="single-wrapper">
    <div class="container" id="content" tabindex="-1">
      <div class="row">
        <div class="col-md-12 content-area" id="primary">
          <main class="site-main" id="main">
            <MMPSearch />
          </main>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.foo {
  margin: 0;
}
</style>
