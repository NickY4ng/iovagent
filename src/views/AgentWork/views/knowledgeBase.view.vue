<script lang="ts" setup>
import { nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue';
import { useRouter } from 'vue-router';

import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

import { Icon } from '@packages/icon';

import { agentWorkData } from '@/pinia/agentWork';

import { agentWorkRouteName } from '../useAgentWorkNav';
import { strokeIconPaths } from '../strokeIconPaths';


interface PlatformVehicle {
  id: string;
  plate: string;
  lineCount: number;
  usageCount: number;
  lat: number;
  lng: number;
}

interface PlatformLine {
  id: string;
  from: string;
  to: string;
  fromLat: number;
  fromLng: number;
  toLat: number;
  toLng: number;
  waybillCount: number;
  carrierVehicleCount: number;
}

interface PlatformPoi {
  id: string;
  name: string;
  type: 'shipper' | 'consignee';
  industry: string;
  count: number;
  lat: number;
  lng: number;
}


const props = withDefaults(defineProps<{ emptyMode?: boolean }>(), { emptyMode: false });

const assetDetailTab = ref<'vehicle' | 'line' | 'poi'>('vehicle');


const platformAssets = {
  vehicleCount: 1200,
  waybillTotal: 130000,
  poiCount: 340,
  platformVehicleCount: 7300000,
  platformPoiCount: 10000000,
  platformMonthlyWaybillCount: 120000000,
};

const operationAnalysis = {
  highRiskWaybillCount: 23,
  lowRiskWaybillCount: 156,
  abnormalEvents: [
    { type: '超速', count: 12 },
    { type: '疲劳驾驶', count: 5 },
    { type: '异常停车', count: 8 },
  ],
};

const platformVehicles = ref<PlatformVehicle[]>([
  { id: 'V003', plate: '浙B34567', lineCount: 6, usageCount: 112, lat: 29.8683, lng: 121.544 },
  { id: 'V006', plate: '粤B24680', lineCount: 4, usageCount: 103, lat: 22.5431, lng: 114.0579 },
  { id: 'V005', plate: '鲁Q98765', lineCount: 5, usageCount: 95, lat: 34.3416, lng: 118.3255 },
  { id: 'V001', plate: '沪A12345', lineCount: 5, usageCount: 86, lat: 31.2304, lng: 121.4737 },
  { id: 'V008', plate: '豫N24680', lineCount: 5, usageCount: 81, lat: 34.4156, lng: 115.65 },
  { id: 'V002', plate: '苏E56789', lineCount: 4, usageCount: 74, lat: 31.5497, lng: 120.2953 },
  { id: 'V007', plate: '京A13579', lineCount: 3, usageCount: 67, lat: 39.9042, lng: 116.4074 },
  { id: 'V004', plate: '皖K55821', lineCount: 3, usageCount: 58, lat: 31.8206, lng: 117.2272 },
  { id: 'V009', plate: '苏A88888', lineCount: 4, usageCount: 72, lat: 32.0603, lng: 118.7969 },
  { id: 'V010', plate: '浙C66666', lineCount: 3, usageCount: 55, lat: 28.0005, lng: 120.7023 },
]);

const platformLines = ref<PlatformLine[]>([
  { id: 'L006', from: '深圳', to: '广州', fromLat: 22.5431, fromLng: 114.0579, toLat: 23.1291, toLng: 113.2644, waybillCount: 486, carrierVehicleCount: 38 },
  { id: 'L001', from: '上海', to: '杭州', fromLat: 31.2304, fromLng: 121.4737, toLat: 30.2741, toLng: 120.1551, waybillCount: 442, carrierVehicleCount: 35 },
  { id: 'L002', from: '苏州', to: '南京', fromLat: 31.2989, fromLng: 120.5853, toLat: 32.0603, toLng: 118.7969, waybillCount: 388, carrierVehicleCount: 31 },
  { id: 'L007', from: '北京', to: '天津', fromLat: 39.9042, fromLng: 116.4074, toLat: 39.0842, toLng: 117.2009, waybillCount: 297, carrierVehicleCount: 26 },
  { id: 'L004', from: '合肥', to: '南京', fromLat: 31.8206, fromLng: 117.2272, toLat: 32.0603, toLng: 118.7969, waybillCount: 251, carrierVehicleCount: 22 },
  { id: 'L005', from: '徐州', to: '济南', fromLat: 34.3416, fromLng: 117.1848, toLat: 36.6512, toLng: 117.1201, waybillCount: 208, carrierVehicleCount: 18 },
  { id: 'L008', from: '郑州', to: '武汉', fromLat: 34.7466, fromLng: 113.6253, toLat: 30.5928, toLng: 114.3055, waybillCount: 196, carrierVehicleCount: 17 },
  { id: 'L003', from: '宁波', to: '上海', fromLat: 29.8683, fromLng: 121.544, toLat: 31.2304, toLng: 121.4737, waybillCount: 334, carrierVehicleCount: 28 },
  { id: 'L009', from: '广州', to: '深圳', fromLat: 23.1291, fromLng: 113.2644, toLat: 22.5431, toLng: 114.0579, waybillCount: 273, carrierVehicleCount: 24 },
  { id: 'L010', from: '杭州', to: '宁波', fromLat: 30.2741, fromLng: 120.1551, toLat: 29.8683, toLng: 121.544, waybillCount: 243, carrierVehicleCount: 21 },
]);

const platformPois = ref<PlatformPoi[]>([
  { id: 'P003', name: '苏州工业园区集散点', type: 'consignee', industry: '制造业', count: 74, lat: 31.316, lng: 120.7488 },
  { id: 'P001', name: '上海外高桥物流中心', type: 'shipper', industry: '仓储物流', count: 86, lat: 31.3226, lng: 121.601 },
  { id: 'P004', name: '南京江宁物流园', type: 'consignee', industry: '仓储物流', count: 58, lat: 31.9533, lng: 118.7975 },
  { id: 'P008', name: '深圳盐田物流园', type: 'consignee', industry: '仓储物流', count: 52, lat: 22.5944, lng: 114.2363 },
  { id: 'P005', name: '宁波北仑港', type: 'shipper', industry: '港口航运', count: 45, lat: 29.9194, lng: 121.8458 },
  { id: 'P002', name: '杭州钱塘智造园', type: 'shipper', industry: '制造业', count: 62, lat: 30.2828, lng: 120.3748 },
  { id: 'P006', name: '合肥经开产业区', type: 'shipper', industry: '制造业', count: 38, lat: 31.7988, lng: 117.2108 },
  { id: 'P007', name: '广州白云物流园', type: 'shipper', industry: '仓储物流', count: 35, lat: 23.1291, lng: 113.2644 },
  { id: 'P009', name: '天津港保税区', type: 'consignee', industry: '港口航运', count: 32, lat: 39.0842, lng: 117.2009 },
  { id: 'P010', name: '成都双流物流园', type: 'consignee', industry: '仓储物流', count: 30, lat: 30.5728, lng: 104.0668 },
]);


const store = agentWorkData();
const router = useRouter();
const activeFolderId = ref('platformAssets');

const highlightedAssetId = ref<string>('');





function selectAssetTab(tab: 'vehicle' | 'line' | 'poi') {
  assetDetailTab.value = tab;
}

const platformMapRef = ref<HTMLDivElement | null>(null);
let platformMapInstance: L.Map | null = null;

const assetLayers: Record<string, L.Layer> = {};

function clearAssetLayers() {
  Object.values(assetLayers).forEach((layer) => {
    if (platformMapInstance) platformMapInstance.removeLayer(layer);
  });
  Object.keys(assetLayers).forEach((key) => delete assetLayers[key]);
}

function createArrowIcon(color: string) {
  const svg = encodeURIComponent(
    `<svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 12 12"><path d="M1 6h8M6 2l5 4-5 4" fill="none" stroke="${color}" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`,
  );
  return L.divIcon({
    className: 'leaflet-arrow-icon',
    html: `<img src="data:image/svg+xml,${svg}" style="width:12px;height:12px;transform:rotate(0deg);" />`,
    iconSize: [12, 12],
    iconAnchor: [6, 6],
  });
}

function buildLineTooltipHtml(line: PlatformLine) {
  return `<div class="line-tooltip-content">
    <div class="line-tooltip-title">${line.from} → ${line.to}</div>
    <div class="line-tooltip-row"><span>运单数</span><b>${line.waybillCount} 单</b></div>
    <div class="line-tooltip-row"><span>承运车辆数</span><b>${line.carrierVehicleCount} 辆</b></div>
  </div>`;
}

const LINE_TIER_STYLES: Record<'low' | 'mid' | 'high', { color: string; weight: number }> = {
  low: { color: '#60a5fa', weight: 2 },
  mid: { color: '#8b5cf6', weight: 4 },
  high: { color: '#b91c1c', weight: 6 },
};

function getLineTierStyles() {
  const counts = platformLines.value.map((line) => line.waybillCount);
  const min = Math.min(...counts);
  const max = Math.max(...counts);
  const step = (max - min) / 3;
  const styles: Record<string, { color: string; weight: number }> = {};
  platformLines.value.forEach((line) => {
    let tier: 'low' | 'mid' | 'high' = 'mid';
    if (max > min) {
      if (line.waybillCount >= min + 2 * step) tier = 'high';
      else if (line.waybillCount >= min + step) tier = 'mid';
      else tier = 'low';
    }
    styles[line.id] = { ...LINE_TIER_STYLES[tier] };
  });
  return styles;
}

function buildArcPoints(fromLat: number, fromLng: number, toLat: number, toLng: number, segments = 32): L.LatLngExpression[] {
  const midLat = (fromLat + toLat) / 2;
  const midLng = (fromLng + toLng) / 2;
  const dx = toLng - fromLng;
  const dy = toLat - fromLat;
  const dist = Math.sqrt(dx * dx + dy * dy) || 1;
  const bulge = dist * 0.22;
  const ctrlLat = midLat + (-dy / dist) * bulge;
  const ctrlLng = midLng + (dx / dist) * bulge;
  const points: L.LatLngExpression[] = [];
  for (let i = 0; i <= segments; i += 1) {
    const t = i / segments;
    const a = (1 - t) ** 2;
    const b = 2 * (1 - t) * t;
    const c = t * t;
    points.push([a * fromLat + b * ctrlLat + c * toLat, a * fromLng + b * ctrlLng + c * toLng]);
  }
  return points;
}

function fitBoundsToLines() {
  if (!platformMapInstance || platformLines.value.length === 0) return;
  if (platformLines.value.length === 1) {
    const only = platformLines.value[0]!;
    platformMapInstance.setView([(only.fromLat + only.toLat) / 2, (only.fromLng + only.toLng) / 2], 8);
    return;
  }
  const bounds = L.latLngBounds(
    platformLines.value.flatMap((line) => [
      [line.fromLat, line.fromLng] as [number, number],
      [line.toLat, line.toLng] as [number, number],
    ]),
  );
  platformMapInstance.fitBounds(bounds, { padding: [48, 48] });
}

function drawPlatformMapLayers() {
  if (!platformMapInstance) return;
  clearAssetLayers();

  const tierStyles = getLineTierStyles();
    platformLines.value.forEach((line) => {
      const tierStyle = tierStyles[line.id] ?? LINE_TIER_STYLES.mid;
      const isHighlighted = highlightedAssetId.value === line.id;
      const arcPoints = buildArcPoints(line.fromLat, line.fromLng, line.toLat, line.toLng);
      const polyline = L.polyline(arcPoints, {
        color: tierStyle.color,
        weight: isHighlighted ? tierStyle.weight + 2 : tierStyle.weight,
        opacity: isHighlighted ? 1 : 0.8,
      }).addTo(platformMapInstance!);

      polyline.bindTooltip(buildLineTooltipHtml(line), {
        direction: 'top',
        sticky: true,
        opacity: 1,
        className: 'line-tooltip',
        offset: [0, -8],
      });
      polyline.on('mouseover', () => {
        polyline.setStyle({ weight: tierStyle.weight + 2, opacity: 1 });
        polyline.bringToFront();
      });
      polyline.on('mouseout', () => {
        if (highlightedAssetId.value === line.id) return;
        polyline.setStyle({ weight: tierStyle.weight, opacity: 0.8 });
      });
      assetLayers[line.id] = polyline;

      // 方向箭头：位于飞线中点，指向终点（取曲线切向）
      const midIdx = Math.floor(arcPoints.length / 2);
      const midPoint = arcPoints[midIdx] as [number, number];
      const prevPoint = arcPoints[midIdx - 1] as [number, number];
      const angle = (Math.atan2(midPoint[1] - prevPoint[1], midPoint[0] - prevPoint[0]) * 180) / Math.PI;
      const arrowMarker = L.marker(midPoint, {
        icon: createArrowIcon('#ffffff'),
        interactive: false,
      }).addTo(platformMapInstance!);
      const img = (arrowMarker.getElement()?.querySelector('img') as HTMLImageElement | null);
      if (img) img.style.transform = `rotate(${-angle}deg)`;
      assetLayers[`${line.id}-arrow`] = arrowMarker;

      const fromMarker = L.circleMarker([line.fromLat, line.fromLng], {
        radius: highlightedAssetId.value === line.id ? 6 : 4,
        color: '#475569',
        fillColor: '#ffffff',
        fillOpacity: 1,
        weight: highlightedAssetId.value === line.id ? 2.5 : 1.5,
      }).addTo(platformMapInstance!);
      assetLayers[`${line.id}-from`] = fromMarker;

      const toMarker = L.circleMarker([line.toLat, line.toLng], {
        radius: highlightedAssetId.value === line.id ? 6 : 4,
        color: '#475569',
        fillColor: '#ffffff',
        fillOpacity: 1,
        weight: highlightedAssetId.value === line.id ? 2.5 : 1.5,
      }).addTo(platformMapInstance!);
      assetLayers[`${line.id}-to`] = toMarker;
    });
}

function initPlatformMap() {
  if (!platformMapRef.value || platformMapInstance) return;
  platformMapInstance = L.map(platformMapRef.value, {
    attributionControl: false,
    center: [32.5, 118.8],
    zoom: 5,
    zoomControl: false,
  });
  L.control.zoom({ position: 'bottomright' }).addTo(platformMapInstance);
  // 原型阶段使用高德栅格底图（GCJ-02 坐标系），与 WGS-84 业务数据存在偏移，仅作演示
  L.tileLayer('https://webrd0{s}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}', {
    maxZoom: 19,
    subdomains: '1234',
  }).addTo(platformMapInstance);
  drawPlatformMapLayers();
  setTimeout(() => {
    platformMapInstance?.invalidateSize();
    fitBoundsToLines();
  }, 80);
}

function clearPlatformMap() {
  platformMapInstance?.remove();
  platformMapInstance = null;
}

watch(
  () => activeFolderId.value,
  (folderId) => {
    if (folderId === 'platformAssets') {
      nextTick(() => {
        if (!platformMapInstance) initPlatformMap();
        else setTimeout(() => {
          platformMapInstance?.invalidateSize();
          fitBoundsToLines();
        }, 80);
      });
    }
  },
  { immediate: true },
);

function formatNumber(num: number) {
  if (num >= 100000000) return `${(num / 100000000).toFixed(1)}亿`;
  if (num >= 10000) return `${(num / 10000).toFixed(0)}万`;
  return num.toLocaleString();
}

function maskPlate(plate: string) {
  if (plate.length <= 4) return plate;
  return `${plate.slice(0, 3)}${'*'.repeat(Math.max(1, plate.length - 4))}${plate.slice(-1)}`;
}

function highlightAsset(id: string) {
  highlightedAssetId.value = id;
  drawPlatformMapLayers();
}

function clearHighlightAsset() {
  highlightedAssetId.value = '';
  drawPlatformMapLayers();
}

function goToAgentConversation() {
  store.startNewConversation();
  router.push({ name: agentWorkRouteName.agent });
}






onMounted(() => {
  if (activeFolderId.value === 'platformAssets') {
    nextTick(() => initPlatformMap());
  }
});
onBeforeUnmount(() => {
  clearPlatformMap();
});
</script>

<template>
  <div
    class="relative flex h-full flex-col overflow-hidden bg-[#fcfcfc]"
  >
    <div class="flex h-12 items-center justify-between gap-4 border-b border-[#eeeeec] bg-[#fcfcfc] px-4">
      <div class="flex items-center gap-2.5">
        <div class="flex h-7 w-7 items-center justify-center rounded-md bg-[#f2f2ef] text-slate-700">
          <Icon :svg="strokeIconPaths.book" :size="16" />
        </div>
        <h1 class="text-sm font-semibold leading-5 text-slate-950">我的知识库</h1>
      </div>
    </div>

    <div class="flex min-h-0 flex-1">

      <main class="flex min-h-0 flex-1 flex-col overflow-hidden bg-[#fcfcfc]">
        <div class="flex min-h-0 flex-1 flex-col overflow-y-auto p-4">
          <!-- 平台数据资产 -->
          <template v-if="activeFolderId === 'platformAssets'">
            <!-- 标题区 -->
            <div class="mb-4">
              <div class="flex items-center gap-2">
                <span class="flex h-7 w-7 items-center justify-center rounded-md bg-blue-50 text-blue-600">
                  <Icon :svg="strokeIconPaths.map" :size="15" />
                </span>
                <h2 class="text-base font-semibold text-slate-900">平台数据资产</h2>
              </div>
              <p class="mt-1 pl-9 text-xs text-slate-500">
                来自你在大卡数字人中的业务调用，已自动沉淀为可检索知识
              </p>
            </div>

            <!-- KPI 卡片（位于地图上方） -->
            <div class="grid grid-cols-3 gap-4" :class="props.emptyMode ? 'mb-6' : 'mb-4'">
              <div class="rounded-xl border border-[#e3e3df] bg-white p-4">
                <div class="flex items-center gap-2">
                  <span class="flex h-8 w-8 items-center justify-center rounded-lg bg-blue-50 text-blue-600">
                    <Icon :svg="strokeIconPaths.truck" :size="15" />
                  </span>
                  <span class="text-xs text-slate-500">车辆使用</span>
                </div>
                <p class="mt-2 text-3xl font-bold tracking-tight text-slate-900">{{ props.emptyMode ? 0 : formatNumber(platformAssets.vehicleCount) }}</p>
                <p class="mt-0.5 text-[11px] text-slate-400">累计使用车辆数</p>
              </div>
              <div class="rounded-xl border border-[#e3e3df] bg-white p-4">
                <div class="flex items-center gap-2">
                  <span class="flex h-8 w-8 items-center justify-center rounded-lg bg-emerald-50 text-emerald-600">
                    <Icon :svg="strokeIconPaths.fileText" :size="15" />
                  </span>
                  <span class="text-xs text-slate-500">运单量</span>
                </div>
                <p class="mt-2 text-3xl font-bold tracking-tight text-slate-900">{{ props.emptyMode ? 0 : formatNumber(platformAssets.waybillTotal) }}</p>
                <p class="mt-0.5 text-[11px] text-slate-400">累计运单数</p>
              </div>
              <div class="rounded-xl border border-[#e3e3df] bg-white p-4">
                <div class="flex items-center gap-2">
                  <span class="flex h-8 w-8 items-center justify-center rounded-lg bg-amber-50 text-amber-600">
                    <Icon :svg="strokeIconPaths.locate" :size="15" />
                  </span>
                  <span class="text-xs text-slate-500">运单起终点企业</span>
                </div>
                <p class="mt-2 text-3xl font-bold tracking-tight text-slate-900">{{ props.emptyMode ? 0 : formatNumber(platformAssets.poiCount) }}</p>
                <p class="mt-0.5 text-[11px] text-slate-400">累计装货 / 卸货企业点位</p>
              </div>
            </div>

            <!-- 地图（放大展示） -->
            <div v-if="!props.emptyMode" class="mb-5 relative">
              <div ref="platformMapRef" class="h-[420px] w-full overflow-hidden rounded-xl border border-[#deded9] bg-[#f7f7f5]"></div>
            </div>
            <div v-else class="mb-6 flex h-[300px] flex-col items-center justify-center gap-4 rounded-xl border border-dashed border-[#d5d5d0] bg-[#fafaf8]">
              <span class="flex h-11 w-11 items-center justify-center rounded-full bg-[#f0f0ed] text-slate-400">
                <Icon :svg="strokeIconPaths.map" :size="20" />
              </span>
              <p class="text-sm font-medium text-slate-500">暂无可视化数据</p>
              <p class="text-xs text-slate-400">产生业务调用后，车辆、路线与起终点企业将在此呈现</p>
            </div>

            <!-- 在途运营分析 -->
            <div v-if="!props.emptyMode" class="mb-5 rounded-xl border border-[#e3e3df] bg-white p-5">
              <div class="mb-4 flex items-center justify-between">
                <div>
                  <h3 class="text-sm font-semibold text-slate-900">在途运营分析</h3>
                  <p class="mt-0.5 text-xs text-slate-500">基于已采购服务生成的在途监控与运营指标</p>
                </div>
                <button
                  type="button"
                  class="inline-flex shrink-0 items-center gap-1.5 rounded-md bg-slate-900 px-3.5 py-2 text-xs font-medium text-white transition hover:bg-slate-700"
                  @click="goToAgentConversation()"
                >
                  <Icon :svg="strokeIconPaths.msg" :size="13" />
                  在对话中分析
                </button>
              </div>
              <div class="grid grid-cols-3 gap-4">
                <div class="rounded-lg border border-[#e3e3df] bg-[#f7f7f5] p-4">
                  <p class="text-xs text-slate-500">高风险运单数</p>
                  <p class="mt-1 text-xl font-semibold text-red-500">{{ operationAnalysis.highRiskWaybillCount }}</p>
                </div>
                <div class="rounded-lg border border-[#e3e3df] bg-[#f7f7f5] p-4">
                  <p class="text-xs text-slate-500">低风险运单数</p>
                  <p class="mt-1 text-xl font-semibold text-emerald-600">{{ operationAnalysis.lowRiskWaybillCount }}</p>
                </div>
                <div class="rounded-lg border border-[#e3e3df] bg-[#f7f7f5] p-4">
                  <p class="text-xs text-slate-500">在途异常事件</p>
                  <p class="mt-1 text-xl font-semibold text-amber-600">
                    {{ operationAnalysis.abnormalEvents.reduce((sum, item) => sum + item.count, 0) }} 次
                  </p>
                </div>
              </div>
            </div>


            <!-- 客户常用资产明细 -->
            <div class="mb-3 flex items-center justify-between">
              <div>
                <h3 class="text-sm font-semibold text-slate-900">
                  {{ assetDetailTab === 'vehicle' ? '车辆使用 · Top 10' : assetDetailTab === 'line' ? '路线覆盖 · Top 10' : '运单起终点企业' }}
                </h3>
                <p class="mt-0.5 text-xs text-slate-500">{{ props.emptyMode ? '数据沉淀后将按运单覆盖排序' : '按运单覆盖排序' }}</p>
              </div>
              <div class="flex items-center gap-1 rounded-lg border border-[#e3e3df] bg-white p-1">
                <button
                  v-for="tab in [
                    { id: 'vehicle', label: '车辆' },
                    { id: 'line', label: '路线' },
                    { id: 'poi', label: 'POI' },
                  ]"
                  :key="tab.id"
                  type="button"
                  class="rounded-md px-3 py-1 text-xs font-medium transition"
                  :class="assetDetailTab === tab.id ? 'bg-slate-900 text-white' : 'text-slate-600 hover:bg-[#f7f7f5] hover:text-slate-900'"
                  @click="selectAssetTab(tab.id as 'vehicle' | 'line' | 'poi')"
                >
                  {{ tab.label }}
                </button>
              </div>
            </div>

            <div v-if="!props.emptyMode" class="mb-4 space-y-2">
              <div
                v-if="assetDetailTab === 'vehicle'"
                v-for="vehicle in platformVehicles"
                :key="vehicle.id"
                class="asset-list-item flex cursor-pointer items-center justify-between rounded-lg border border-[#deded9] bg-white px-4 py-3 transition hover:border-blue-300 hover:bg-blue-50/30"
                :class="highlightedAssetId === vehicle.id ? 'border-blue-400 bg-blue-50/60' : ''"
                @mouseenter="highlightAsset(vehicle.id)"
                @mouseleave="clearHighlightAsset"
              >
                <div class="flex items-center gap-3">
                  <span class="flex h-9 w-9 items-center justify-center rounded-md bg-blue-50 text-blue-600">
                    <Icon :svg="strokeIconPaths.truck" :size="16" />
                  </span>
                  <div>
                    <p class="text-sm font-medium text-slate-900">{{ maskPlate(vehicle.plate) }}</p>
                  </div>
                </div>
                <div class="flex gap-6 text-xs text-slate-500">
                  <span>车辆使用次数 {{ vehicle.usageCount }} 次</span>
                </div>
              </div>

              <div
                v-if="assetDetailTab === 'line'"
                v-for="line in platformLines"
                :key="line.id"
                class="asset-list-item flex cursor-pointer items-center justify-between rounded-lg border border-[#deded9] bg-white px-4 py-3 transition hover:border-emerald-300 hover:bg-emerald-50/30"
                :class="highlightedAssetId === line.id ? 'border-emerald-400 bg-emerald-50/60' : ''"
                @mouseenter="highlightAsset(line.id)"
                @mouseleave="clearHighlightAsset"
              >
                <div class="flex items-center gap-3">
                  <span class="flex h-9 w-9 items-center justify-center rounded-md bg-emerald-50 text-emerald-600">
                    <Icon :svg="strokeIconPaths.route" :size="16" />
                  </span>
                  <div>
                    <p class="text-sm font-medium text-slate-900">{{ line.from }} → {{ line.to }}</p>
                  </div>
                </div>
                <div class="flex gap-6 text-xs text-slate-500">
                  <span>运单覆盖 {{ line.waybillCount }} 单</span>
                </div>
              </div>

              <template v-if="assetDetailTab === 'poi'">
                <div>
                  <p class="mb-2 flex items-center gap-2 text-xs font-medium text-slate-500">
                    <span class="h-2 w-2 rounded-full bg-emerald-500"></span>
                    装货地 POI · Top 5
                  </p>
                  <div class="space-y-2">
                    <div
                      v-for="poi in platformPois.filter((item) => item.type === 'shipper').slice(0, 5)"
                      :key="poi.id"
                      class="asset-list-item flex cursor-pointer items-center justify-between rounded-lg border border-[#deded9] bg-white px-4 py-3 transition hover:border-emerald-300 hover:bg-emerald-50/30"
                      :class="highlightedAssetId === poi.id ? 'border-emerald-400 bg-emerald-50/60' : ''"
                      @mouseenter="highlightAsset(poi.id)"
                      @mouseleave="clearHighlightAsset"
                    >
                      <div class="flex items-center gap-3">
                        <span class="flex h-9 w-9 items-center justify-center rounded-md bg-emerald-50 text-emerald-600">
                          <Icon :svg="strokeIconPaths.locate" :size="16" />
                        </span>
                        <div>
                          <p class="text-sm font-medium text-slate-900">{{ poi.name }}</p>
                        </div>
                      </div>
                      <span class="text-xs text-slate-500">累计运单覆盖 {{ poi.count }} 次</span>
                    </div>
                  </div>
                </div>

                <div class="pt-2">
                  <p class="mb-2 flex items-center gap-2 text-xs font-medium text-slate-500">
                    <span class="h-2 w-2 rounded-full bg-orange-500"></span>
                    卸货地 POI · Top 5
                  </p>
                  <div class="space-y-2">
                    <div
                      v-for="poi in platformPois.filter((item) => item.type === 'consignee').slice(0, 5)"
                      :key="poi.id"
                      class="asset-list-item flex cursor-pointer items-center justify-between rounded-lg border border-[#deded9] bg-white px-4 py-3 transition hover:border-orange-300 hover:bg-orange-50/30"
                      :class="highlightedAssetId === poi.id ? 'border-orange-400 bg-orange-50/60' : ''"
                      @mouseenter="highlightAsset(poi.id)"
                      @mouseleave="clearHighlightAsset"
                    >
                      <div class="flex items-center gap-3">
                        <span class="flex h-9 w-9 items-center justify-center rounded-md bg-orange-50 text-orange-600">
                          <Icon :svg="strokeIconPaths.locate" :size="16" />
                        </span>
                        <div>
                          <p class="text-sm font-medium text-slate-900">{{ poi.name }}</p>
                        </div>
                      </div>
                      <span class="text-xs text-slate-500">累计运单覆盖 {{ poi.count }} 次</span>
                    </div>
                  </div>
                </div>
              </template>
            </div>

            <div v-else class="mb-4 flex flex-col items-center justify-center gap-3 rounded-xl border border-dashed border-[#d5d5d0] bg-[#fafaf8] py-16">
              <span class="flex h-10 w-10 items-center justify-center rounded-full bg-[#f0f0ed] text-slate-400">
                <Icon :svg="assetDetailTab === 'vehicle' ? strokeIconPaths.truck : assetDetailTab === 'line' ? strokeIconPaths.route : strokeIconPaths.locate" :size="18" />
              </span>
              <p class="text-sm font-medium text-slate-500">
                {{ assetDetailTab === 'vehicle' ? '暂无使用车辆' : assetDetailTab === 'line' ? '暂无覆盖路线' : '暂无起终点企业' }}
              </p>
              <p class="text-xs text-slate-400">产生业务调用后，数据将自动沉淀至此</p>
            </div>

            <!-- 平台可扩展数据能力（需采购） -->
            <div class="rounded-xl border border-blue-200 bg-blue-50/50 p-5">
              <div class="mb-3 flex items-center gap-2">
                <span class="flex h-6 w-6 items-center justify-center rounded-full bg-blue-100 text-blue-600">
                  <Icon :svg="strokeIconPaths.info" :size="14" />
                </span>
                <h3 class="text-sm font-semibold text-slate-900">更多数据能力</h3>
              </div>

              <div class="mb-4 rounded-lg bg-gradient-to-r from-slate-800 to-slate-900 p-4 text-white shadow-sm">
                <p class="text-xs font-medium text-slate-200">大卡数字人数据底座</p>
                <p class="mt-2 text-sm">
                  {{ formatNumber(platformAssets.platformVehicleCount) }} 辆在网重卡 ·
                  {{ formatNumber(platformAssets.platformPoiCount) }} 个行业 POI ·
                  月均 {{ formatNumber(platformAssets.platformMonthlyWaybillCount) }} 单虚拟运单
                </p>
              </div>

              <p class="mb-3 text-xs text-slate-600">
                以下数据分析能力可基于平台扩展数据提供，如需开通请联系销售经理。
              </p>
              <div class="grid grid-cols-2 gap-x-6 gap-y-2 text-xs text-slate-600">
                <div class="flex items-center gap-1.5">
                  <span class="h-1 w-1 rounded-full bg-blue-400"></span>
                  车辆静态档案信息
                </div>
                <div class="flex items-center gap-1.5">
                  <span class="h-1 w-1 rounded-full bg-blue-400"></span>
                  路线其他发货方、收货方、运量、用车情况
                </div>
                <div class="flex items-center gap-1.5">
                  <span class="h-1 w-1 rounded-full bg-blue-400"></span>
                  同路线其他承运商及承运车辆信息
                </div>
                <div class="flex items-center gap-1.5">
                  <span class="h-1 w-1 rounded-full bg-blue-400"></span>
                  POI 档案信息（名称、经营范围、地点、围栏）
                </div>
                <div class="flex items-center gap-1.5">
                  <span class="h-1 w-1 rounded-full bg-blue-400"></span>
                  常跑路线行业洞察（运量、货类、承运商分布）
                </div>
              </div>
            </div>

            <div
              v-if="!props.emptyMode"
              class="mt-4 flex cursor-pointer items-center justify-between rounded-lg border border-[#deded9] bg-white px-4 py-3 transition hover:border-slate-400 hover:bg-[#f7f7f5]"
              @click="goToAgentConversation()"
            >
              <div class="flex items-center gap-3">
                <span class="flex h-9 w-9 items-center justify-center rounded-full bg-slate-900 text-white">
                  <Icon :svg="strokeIconPaths.msg" :size="15" />
                </span>
                <div>
                  <p class="text-sm font-medium text-slate-900">在对话中分析</p>
                  <p class="text-xs text-slate-500">基于当前平台数据资产继续提问</p>
                </div>
              </div>
              <Icon :svg="strokeIconPaths.chevron" :size="14" svg-class="text-slate-400" />
            </div>
          </template>
        </div>
      </main>

    </div>


  </div>
</template>

<style lang="scss" scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>

<style lang="scss">
.line-tooltip {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  box-shadow: 0 8px 24px rgba(15, 23, 42, 0.16);
  padding: 10px 12px;
  color: #0f172a;

  &::before {
    display: none;
  }
}

.line-tooltip-content {
  min-width: 168px;
}

.line-tooltip-title {
  font-size: 12px;
  font-weight: 600;
  color: #0f172a;
  margin-bottom: 6px;
  padding-bottom: 6px;
  border-bottom: 1px solid #f1f5f9;
}

.line-tooltip-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  font-size: 11px;
  line-height: 20px;

  span {
    color: #64748b;
  }

  b {
    font-weight: 600;
    color: #0f172a;
  }
}
</style>
