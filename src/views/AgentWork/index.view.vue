<script lang="ts" setup>
import AddProjectDialog from './component/addProject.dialog.vue';
import MenuComp from './component/menu.comp.vue';
</script>

<template>
  <div class="grid h-screen grid-cols-[280px_minmax(0,1fr)] overflow-hidden bg-[#f6f6f4] text-slate-900">
    <MenuComp />

    <main class="h-full overflow-hidden bg-[#f7f7f5] p-3">
      <RouterView />
    </main>

    <AddProjectDialog />
  </div>
</template>

<style lang="scss" scoped></style>
