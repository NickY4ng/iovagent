<script lang="ts" setup>
import type { PageId } from '../interface';

import { useRoute, useRouter } from 'vue-router';
import { computed, ref } from 'vue';

import { Icon } from '@packages/icon';
import { storeToRefs } from 'pinia';

import { agentWorkData } from '@/pinia/agentWork';
import { removeToken } from '@/utils/auth';

import { strokeIconPaths } from '../strokeIconPaths';
import { agentWorkRouteName } from '../useAgentWorkNav';
import { badgeToneClass, projectStatusTone } from '../utils';

const store = agentWorkData();
const { projects } = storeToRefs(store);
const route = useRoute();
const router = useRouter();
const isUserMenuOpen = ref(false);
const isProjectCreateActive = computed(() => route.name === agentWorkRouteName.projectCreate);
const currentUserName = computed(() => localStorage.getItem('iovagent_login_user') || import.meta.env.VITE_APP_DEFAULT_LOGIN_USERNAME || '演示用户');

const navs: { icon: string; id: PageId; label: string }[] = [
  { id: 'agent', label: '智能体工作台', icon: strokeIconPaths.bot },
  { id: 'orders', label: '运单列表', icon: strokeIconPaths.list },
  { id: 'risk', label: '异常运单列表', icon: strokeIconPaths.shield },
  { id: 'detail', label: '运单详情与地图', icon: strokeIconPaths.map },
  { id: 'analytics', label: '统计归因', icon: strokeIconPaths.gauge },
  { id: 'projects', label: '项目管理', icon: strokeIconPaths.settings },
  { id: 'downloads', label: '下载任务', icon: strokeIconPaths.download },
];

function goNav(page: PageId) {
  if (page === 'projectCreate') {
    if (isProjectCreateActive.value) return;
    router.push({ name: agentWorkRouteName[page], query: { from: route.fullPath } });
    return;
  }
  router.push({ name: agentWorkRouteName[page] });
}

function isNavActive(page: PageId) {
  if (page === 'projects' && route.name === agentWorkRouteName.projectCreate) return true;
  return route.name === agentWorkRouteName[page];
}

function logout() {
  removeToken();
  localStorage.removeItem('token');
  localStorage.removeItem('iovagent_login_user');
  isUserMenuOpen.value = false;
  router.replace('/login');
}
</script>

<template>
  <aside class="flex h-full flex-col overflow-y-hidden border-r border-[#e9e9e7] bg-[#f7f7f6]">
    <div class="border-b border-[#e9e9e7] bg-[#f7f7f6] px-4 py-3">
      <div class="flex items-center gap-2.5">
        <div class="flex h-8 w-8 items-center justify-center rounded-md border border-[#deded9] bg-white text-slate-700">
          <Icon :svg="strokeIconPaths.truck" :size="18" />
        </div>
        <div class="min-w-0">
          <div class="truncate text-sm font-semibold text-slate-950">在途物流智能体</div>
        </div>
      </div>
      <div class="mt-3 flex items-center justify-between gap-2 text-xs text-slate-500">
        <span
          class="inline-flex items-center rounded-md border px-2 py-0.5 font-medium"
          :class="badgeToneClass(projectStatusTone(store.currentProject.status))"
        >
          {{ store.currentProject.status }}
        </span>
        <span class="truncate">{{ store.currentProject.name }}</span>
      </div>
    </div>

    <div class="p-3">
      <button
        type="button"
        class="flex w-full items-center justify-center gap-2 rounded-md bg-slate-900 px-3 py-2 text-sm font-medium text-white transition hover:bg-slate-800"
        @click="goNav('projectCreate')"
      >
        <Icon :svg="strokeIconPaths.plus" :size="16" /> 新建项目
      </button>
    </div>

    <div class="flex h-0 flex-1 flex-col px-3 pb-3">
      <div class="mb-2 px-1 text-xs font-medium text-slate-500">项目</div>
      <div class="min-h-0 flex-1 divide-y divide-[#deded9] overflow-y-auto">
        <button
          v-for="p in projects"
          :key="p.id"
          type="button"
          class="w-full rounded-md px-2.5 py-2.5 text-left transition"
          :class="store.currentProject.id === p.id ? 'bg-white' : 'hover:bg-white/75'"
          @click="store.switchProject(p)"
        >
          <div class="flex items-center justify-between gap-2">
            <div class="text-sm font-medium text-slate-900">
              {{ p.name }}
            </div>
            <Icon :svg="strokeIconPaths.chevron" :size="15" svg-class="text-slate-400" />
          </div>
          <div class="mt-1.5 flex items-center justify-between gap-2">
            <div class="truncate text-[11px] leading-5 text-slate-500">今日 {{ p.total }} 单 · {{ p.risk }} 异常</div>
            <span
              class="shrink-0 rounded-md border px-1.5 py-0.5 text-[11px] font-medium leading-4"
              :class="badgeToneClass(projectStatusTone(p.status))"
            >
              {{ p.status }}
            </span>
          </div>
        </button>
      </div>
    </div>

    <nav class="border-t border-[#e9e9e7] px-3 py-3">
      <div class="mb-2 px-1 text-xs font-medium text-slate-500">菜单</div>
      <div class="space-y-1">
        <button
          v-for="n in navs"
          :key="n.id"
          type="button"
          class="flex w-full items-center gap-3 rounded-md px-3 py-2 text-sm transition"
          :class="isNavActive(n.id) ? 'bg-white font-medium text-slate-950' : 'text-slate-600 hover:bg-white/75'"
          @click="goNav(n.id)"
        >
          <Icon :svg="n.icon" :size="17" /> {{ n.label }}
        </button>
      </div>
    </nav>

    <div class="relative border-t border-[#e9e9e7] p-3">
      <div
        v-if="isUserMenuOpen"
        class="absolute right-3 bottom-full left-3 z-20 mb-2 overflow-hidden rounded-md border border-[#deded9] bg-white shadow-lg"
      >
        <button type="button" class="flex w-full items-center gap-2 px-3 py-2 text-left text-sm text-slate-700 hover:bg-[#f7f7f5]" @click="logout">
          <Icon :svg="strokeIconPaths.x" :size="15" />
          退出登录
        </button>
      </div>
      <button
        type="button"
        class="flex w-full items-center justify-between gap-2 rounded-md px-2.5 py-2 text-left text-sm text-slate-700 hover:bg-white"
        @click="isUserMenuOpen = !isUserMenuOpen"
      >
        <span class="flex min-w-0 items-center gap-2">
          <span class="flex h-7 w-7 shrink-0 items-center justify-center rounded-md border border-[#deded9] bg-white text-slate-600">
            <Icon :svg="strokeIconPaths.user" :size="15" />
          </span>
          <span class="min-w-0">
            <span class="block text-[11px] leading-4 text-slate-400">当前用户</span>
            <span class="block truncate text-xs font-medium leading-4 text-slate-800">{{ currentUserName }}</span>
          </span>
        </span>
        <Icon :svg="strokeIconPaths.chevron" :size="14" :svg-class="isUserMenuOpen ? '-rotate-90 text-slate-500' : 'rotate-90 text-slate-400'" />
      </button>
    </div>
  </aside>
</template>

<style lang="scss"></style>
