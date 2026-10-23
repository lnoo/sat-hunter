<template>
  <div
    class="flex h-screen w-screen overflow-hidden font-sans text-gray-700 bg-gray-100 dark:bg-gray-900 dark:text-gray-300">
    <aside
      class="flex flex-col w-96 bg-white shadow-2xl z-1000 overflow-y-auto border-r border-gray-200 dark:bg-gray-800 dark:border-gray-700">
      <div
        class="p-5 border-b border-gray-100 dark:border-gray-700 flex justify-between items-center">
        <h2
          class="text-xl font-bold text-gray-800 dark:text-white flex items-center gap-2">
          <svg
            width="40"
            height="40"
            viewBox="0 0 100 100"
            fill="none"
            xmlns="http://www.w3.org/2000/svg">
            <circle
              cx="50"
              cy="50"
              r="35"
              stroke="#3B82F6"
              stroke-width="2"
              stroke-dasharray="4 4" />
            <circle cx="50" cy="50" r="22" fill="#DBEAFE" />
            <path
              d="M50 35V65M50 65L40 55M50 65L60 55"
              stroke="#1D4ED8"
              stroke-width="5"
              stroke-linecap="round"
              stroke-linejoin="round" />
            <rect x="75" y="25" width="10" height="10" rx="2" fill="#1D4ED8">
              <animateTransform
                attributeName="transform"
                type="rotate"
                from="0 50 50"
                to="360 50 50"
                dur="10s"
                repeatCount="indefinite" />
            </rect>
          </svg>
          卫星底图下载器
          <span
            class="text-xs bg-blue-100 text-blue-700 px-2 py-0.5 rounded-full dark:bg-blue-900 dark:text-blue-300">
            Pro
          </span>
        </h2>
        <label class="switch">
          <input type="checkbox" :checked="isDark" @change="toggleDark" />
          <span class="slider"></span>
        </label>
      </div>

      <div class="flex-1 p-5 flex flex-col gap-6">
        <div
          class="text-xs h-8 font-bold text-center py-2 px-3 rounded-md border transition-colors duration-300"
          :class="[
            currentMode === 'polygon'
              ? 'bg-green-50 text-green-700 border-green-200 dark:bg-green-900/30 dark:text-green-400 dark:border-green-800'
              : currentMode === 'rect'
                ? 'bg-gray-100 text-gray-600 border-gray-200 dark:bg-gray-700 dark:text-gray-300 dark:border-gray-600'
                : 'bg-yellow-50 text-yellow-600 border-yellow-100 dark:bg-yellow-900/30 dark:text-yellow-400 dark:border-yellow-800',
          ]">
          {{ modeText }}
        </div>

        <div
          class="bg-white rounded-xl border border-gray-100 shadow-sm p-4 hover:shadow-md transition-shadow dark:bg-gray-800 dark:border-gray-700">
          <div
            class="text-sm font-bold text-gray-800 mb-3 flex items-center gap-2 dark:text-gray-200">
            <span class="text-lg">📍</span>
            按区域轮廓下载
          </div>
          <Select
            :modelValue="searchQuery"
            :options="regionOptions"
            @update:modelValue="selectRegion"
            placeholder="选择或输入区域关键字" />
        </div>

        <div
          class="bg-white rounded-xl border border-gray-100 shadow-sm p-4 dark:bg-gray-800 dark:border-gray-700">
          <div
            class="text-sm font-bold text-gray-800 mb-2 flex items-center gap-2 dark:text-gray-200">
            <span class="text-lg">📐</span>
            按矩形框选下载
          </div>
          <p class="text-xs text-gray-500 leading-relaxed dark:text-gray-400">
            点击地图左侧工具栏的
            <strong
              class="text-gray-900 bg-gray-200 px-1 rounded dark:text-gray-100 dark:bg-gray-700">
              ⬛ 矩形
            </strong>
            图标画框。
            <br />
            <span class="text-blue-500 dark:text-blue-400">
              提示：画完后可以拖动角点微调范围。
            </span>
          </p>
        </div>

        <div
          class="bg-white rounded-xl border border-gray-100 shadow-sm p-4 dark:bg-gray-800 dark:border-gray-700">
          <div
            class="text-sm font-bold text-gray-800 mb-3 flex items-center gap-2 dark:text-gray-200">
            <span class="text-lg">⚙️</span>
            设置
          </div>

          <div class="mb-2 flex justify-between items-center">
            <label class="text-xs font-medium text-gray-600 dark:text-gray-400">
              缩放级别 (Zoom)
            </label>
            <span
              class="text-xs font-bold text-blue-600 bg-blue-50 px-2 py-0.5 rounded dark:bg-blue-900/50 dark:text-blue-400">
              {{ zoomLevel }}
            </span>
          </div>

          <input
            type="range"
            min="1"
            max="18"
            v-model.number="zoomLevel"
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-600 dark:bg-gray-700" />

          <div
            class="flex justify-between mt-2 text-[10px] text-gray-400 dark:text-gray-500">
            <span>全球 (1-3)</span>
            <span>国家级 (6)</span>
            <span>省市级 (10)</span>
            <span>街道级 (18)</span>
          </div>

          <div
            v-if="tileCount > 1000"
            class="mt-3 text-xs bg-amber-50 text-amber-700 p-2 rounded border border-amber-100 flex items-start gap-1 dark:bg-amber-900/30 dark:text-amber-400 dark:border-amber-800">
            ⚠️
            <span>
              瓦片量
              <strong>{{ tileCount }}</strong>
              张，可能导致内存不足。
            </span>
          </div>

          <div class="mt-4">
            <div class="mb-2 flex justify-between items-center">
              <label
                class="text-xs font-medium text-gray-600 dark:text-gray-400">
                底图
              </label>
            </div>
            <div class="grid grid-cols-3 gap-2">
              <button
                v-for="option in baseMapOptions"
                :key="option.id"
                class="flex-1 py-1.5 px-1 text-xs rounded-lg border transition-colors cursor-pointer truncate"
                :class="
                  option.id === currentBaseMap
                    ? 'bg-blue-600 text-white border-blue-600 dark:bg-blue-600 dark:border-blue-600'
                    : 'bg-white text-gray-600 border-gray-200 hover:bg-gray-50 dark:bg-gray-700 dark:text-gray-300 dark:border-gray-600 dark:hover:bg-gray-600'
                "
                @click="setBaseMap(option.id)">
                {{ option.name }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="p-5 mt-auto">
        <button
          class="w-full py-3 px-4 cursor-pointer bg-blue-600 hover:bg-blue-700 text-white text-sm font-bold rounded-xl shadow-lg shadow-blue-200 disabled:bg-gray-300 disabled:text-gray-500 disabled:shadow-none disabled:cursor-not-allowed transition-all transform active:scale-[0.98] dark:shadow-blue-900/30 dark:disabled:bg-gray-700 dark:disabled:text-gray-500"
          :disabled="!currentMode || isDownloading"
          @click="startDownload">
          {{ isDownloading ? `处理中 ${downloadProgress}%` : "开始下载" }}
        </button>

        <div
          v-if="isDownloading"
          class="mt-3 h-2 w-full bg-gray-200 rounded-full overflow-hidden dark:bg-gray-700">
          <div
            class="h-full bg-green-500 transition-all duration-300 ease-out"
            :style="{ width: downloadProgress + '%' }"></div>
        </div>

        <div class="text-center mt-3 text-xs text-gray-400 font-medium min-h-4">
          {{ statusText }}
        </div>
      </div>
    </aside>

    <div
      id="map"
      ref="mapContainer"
      class="flex-1 h-full z-0 bg-gray-900"></div>
  </div>
</template>

<script setup lang="ts">
  import { onMounted, ref, shallowRef, computed, watch } from "vue";
  import Select from "./components/Select.vue";
  import L from "leaflet";
  import "leaflet/dist/leaflet.css";
  import "@geoman-io/leaflet-geoman-free";
  import "@geoman-io/leaflet-geoman-free/dist/leaflet-geoman.css";

  type DownloadMode = "rect" | "polygon" | null;
  type BaseMapId =
    | "esri_light_gray"
    | "esri_dark_gray"
    | "esri_hill_shade"
    | "esri_imagery_firefly"
    | "esri_imagery"
    | "gaode_vector"
    | "google_satellite";

  interface TileCoord {
    x: number;
    y: number;
  }

  interface Point {
    x: number;
    y: number;
  }

  interface GeoJSONResult {
    type: "Polygon" | "MultiPolygon";
    coordinates: any[];
  }

  interface RegionItem {
    adcode: number | string;
    name: string;
    center: [number, number];
    centroid: [number, number];
    level: string;
    bbox: [number, number, number, number];
    childrenNum?: number;
    parent?: {
      adcode: number | null;
    };
    subFeatureIndex?: number;
    acroutes?: number[];
    pinyin?: string[];
  }

  interface BaseMapOption {
    id: BaseMapId;
    name: string;
    url: string;
    labelUrl?: string;
    labelOptions?: L.TileLayerOptions;
  }

  // 内置"全世界"选项：adcode 使用字符串哨兵值，避免与真实行政区划冲突
  const WORLD_ADCODE = "world";

  const WORLD_REGION: RegionItem = {
    adcode: WORLD_ADCODE,
    name: "全世界 (World)",
    center: [20, 0],
    centroid: [20, 0],
    level: "世界",
    bbox: [-180, -85, 180, 85],
  };

  // 全球范围 GeoJSON（留出极区空白，Web 墨卡托在 ±90° 无法投影）
  const WORLD_GEOJSON: GeoJSONResult = {
    type: "Polygon",
    coordinates: [
      [
        [-180, -85],
        [180, -85],
        [180, 85],
        [-180, 85],
        [-180, -85],
      ],
    ],
  };

  const mapContainer = ref<HTMLElement | null>(null);
  const map = shallowRef<L.Map | null>(null);
  const cityLayer = shallowRef<L.Layer | null>(null);
  const rectLayer = shallowRef<L.Layer | null>(null);
  const baseMapLayer = shallowRef<L.TileLayer | null>(null);
  const labelLayer = shallowRef<L.TileLayer | null>(null);

  const searchQuery = ref<number | string>();
  const allRegions = ref<RegionItem[]>([]);
  const isRegionLoading = ref(false);
  const regionLoadError = ref("");
  const isSearching = ref(false);
  const searchError = ref("");
  const zoomLevel = ref(13);
  const currentMode = ref<DownloadMode>(null);
  const cityGeoJSON = ref<GeoJSONResult | null>(null);
  const tileCount = ref(0);
  const isDownloading = ref(false);
  const downloadProgress = ref(0);
  const statusText = ref("请搜索城市或绘制矩形...");
  const currentBaseMap = ref<BaseMapId>("gaode_vector");
  const isDark = ref(false);

  const updateThemeState = (dark: boolean) => {
    isDark.value = dark;
    if (dark) {
      document.documentElement.classList.add("dark");
    } else {
      document.documentElement.classList.remove("dark");
    }
  };

  const toggleDark = () => {
    const newState = !isDark.value;
    updateThemeState(newState);
    localStorage.setItem("theme", newState ? "dark" : "light");
  };

  const baseMapOptions: BaseMapOption[] = [
    {
      id: "gaode_vector",
      name: "高德 卫星影像",
      url: "https://webst01.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}",
    },
    {
      id: "esri_light_gray",
      name: "浅色灰底图",
      url: "https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_dark_gray",
      name: "暗色灰底图",
      url: "https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Dark_Gray_Base/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_imagery",
      name: "ESRI 卫星影像",
      url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_hill_shade",
      name: "地形图",
      url: "https://server.arcgisonline.com/arcgis/rest/services/Elevation/World_Hillshade/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "esri_imagery_firefly",
      name: "萤火虫影像混合图",
      url: "https://fly.maptiles.arcgis.com/arcgis/rest/services/World_Imagery_Firefly/MapServer/tile/{z}/{y}/{x}",
    },
    {
      id: "google_satellite",
      name: "Google 卫星影像",
      url: "http://mt0.google.com/vt/lyrs=s&hl=en&x={x}&y={y}&z={z}",
    },
  ];

  const regionOptions = computed<RegionItem[]>(() => [
    WORLD_REGION,
    ...allRegions.value,
  ]);

  const selectedData = computed<RegionItem | undefined>(() => {
    if (searchQuery.value === WORLD_ADCODE) return WORLD_REGION;
    return allRegions.value.find((op) => op.adcode === searchQuery.value);
  });

  const modeText = computed(() => {
    if (currentMode.value === "rect") return "当前模式: 矩形框选";
    if (currentMode.value === "polygon")
      return `当前模式: 区域轮廓 (${selectedData.value?.name})`;
    return "当前模式: 等待操作";
  });

  watch(zoomLevel, () => updateEstimate());

  const setBaseMap = (id: BaseMapId) => {
    if (!map.value) return;
    if (currentBaseMap.value === id && baseMapLayer.value) return;
    currentBaseMap.value = id;
    if (baseMapLayer.value) {
      map.value.removeLayer(baseMapLayer.value);
      baseMapLayer.value = null;
    }
    if (labelLayer.value) {
      map.value.removeLayer(labelLayer.value);
      labelLayer.value = null;
    }
    const config = baseMapOptions.find((item) => item.id === id);
    if (!config) return;
    const layer = L.tileLayer(config.url, {
      maxZoom: 19,
      crossOrigin: "anonymous",
    });
    layer.addTo(map.value);
    baseMapLayer.value = layer;
  };

  const loadAddressData = async () => {
    isRegionLoading.value = true;
    regionLoadError.value = "";
    try {
      const res = await fetch(
        "https://geo.datav.aliyun.com/areas_v3/bound/all.json",
      );
      if (!res.ok) {
        throw new Error(`Failed to load address data: ${res.status}`);
      }
      const ct = res.headers.get("content-type") || "";
      if (!ct.includes("application/json")) {
        throw new Error("Invalid content type for address data");
      }
      const data = await res.json();
      if (Array.isArray(data)) {
        allRegions.value = data as RegionItem[];
      } else {
        allRegions.value = [];
        regionLoadError.value = "区域数据格式不正确";
      }
    } catch (e) {
      regionLoadError.value = "区域数据加载失败";
      console.error(e);
    } finally {
      isRegionLoading.value = false;
    }
  };

  onMounted(() => {
    const theme = localStorage.getItem("theme");
    const mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");

    if (theme === "dark") {
      updateThemeState(true);
    } else if (theme === "light") {
      updateThemeState(false);
    } else {
      updateThemeState(mediaQuery.matches);
    }

    mediaQuery.addEventListener("change", (e) => {
      updateThemeState(e.matches);
      localStorage.removeItem("theme");
    });

    if (!mapContainer.value) return;

    // 1. 创建 Map
    const mapInstance = L.map(mapContainer.value).setView(
      [39.9042, 116.4074],
      10,
    );
    map.value = mapInstance;

    // 2. 底图
    setBaseMap(currentBaseMap.value);

    mapInstance.pm.addControls({
      position: "topleft",
      drawCircleMarker: false,
      drawMarker: false,
      drawCircle: false,
      drawPolyline: false,
      drawPolygon: false, // 如果需要手绘多边形可设为 true
      drawText: false,
      drawRectangle: true, // 核心功能

      editMode: true, // 允许编辑顶点
      dragMode: false, // 禁止整体拖拽(防止误操作)
      cutPolygon: false,
      removalMode: true,
      rotateMode: false,
    });

    // 设置绘制样式
    mapInstance.pm.setPathOptions({
      color: "#3b82f6", // Tailwind Blue-500
      fillColor: "#3b82f6",
      fillOpacity: 0.2,
      weight: 2,
    });

    // 设置全局语言
    mapInstance.pm.setLang("zh");

    // 监听创建 (画完框)
    mapInstance.on("pm:create", (e: any) => {
      // 1. 清除旧的
      clearAllLayers();

      const layer = e.layer;
      rectLayer.value = layer;
      setMode("rect");

      // 2. 监听该层的编辑事件 (拖拽角点)
      layer.on("pm:edit", () => {
        updateEstimate();
      });
    });

    // 监听删除 (点击删除按钮)
    mapInstance.on("pm:remove", (e: any) => {
      if (e.layer === rectLayer.value) {
        rectLayer.value = null;
        setMode(null);
      }
    });

    loadAddressData();
  });

  const clearAllLayers = (resetMode = true) => {
    // 移除城市轮廓
    if (cityLayer.value && map.value) {
      map.value.removeLayer(cityLayer.value);
      cityLayer.value = null;
    }

    // 移除矩形框 (Geoman 已经把它加到地图上了，所以需要手动 remove)
    if (rectLayer.value && map.value) {
      map.value.removeLayer(rectLayer.value);
      rectLayer.value = null;
    }

    cityGeoJSON.value = null;
    if (resetMode) {
      setMode(null);
    }
  };

  const setMode = (mode: DownloadMode) => {
    currentMode.value = mode;
    updateEstimate();
  };

  const selectRegion = (adcode: number | string) => {
    searchQuery.value = adcode;
    searchError.value = "";

    // 内置"全世界"：无需请求轮廓数据，直接使用全球范围
    if (adcode === WORLD_ADCODE) {
      zoomLevel.value = 4;
      clearAllLayers(false);
      setMode("polygon");
      cityGeoJSON.value = WORLD_GEOJSON;
      if (map.value) {
        const layer = L.geoJSON(WORLD_GEOJSON as any, {
          style: { color: "#ef4444", weight: 2, fillOpacity: 0.05 },
          interactive: false,
        }).addTo(map.value);
        cityLayer.value = layer;
        map.value.fitBounds(layer.getBounds());
        updateEstimate();
        statusText.value = "已选中: 全世界";
      }
      return;
    }

    const level = { country: 6, province: 10, city: 10, district: 14 }?.[
      selectedData.value?.level ?? ""
    ];
    if (level) {
      zoomLevel.value = level;
    }

    searchLocation(adcode);
  };

  const searchLocation = async (adcode: number | string) => {
    // 搜索时清除之前的框
    clearAllLayers(false);
    setMode("polygon");

    statusText.value = "正在获取轮廓...";

    try {
      const endpoint = `https://geo.datav.aliyun.com/areas_v3/bound/${adcode}.json`;

      const res = await fetch(endpoint);
      if (!res.ok) {
        throw new Error(`Failed to fetch boundary: ${res.status}`);
      }
      const ct = res.headers.get("content-type") || "";
      if (!ct.includes("application/json")) {
        throw new Error("Invalid content type for boundary data");
      }
      const json = await res.json();

      let geometry: GeoJSONResult | null = null;
      if (
        json &&
        json.type === "FeatureCollection" &&
        Array.isArray(json.features)
      ) {
        const polys: any[] = [];
        json.features.forEach((f: any) => {
          if (f && f.geometry) {
            if (f.geometry.type === "Polygon") {
              polys.push(f.geometry.coordinates);
            } else if (f.geometry.type === "MultiPolygon") {
              polys.push(...f.geometry.coordinates);
            }
          }
        });
        if (polys.length > 0) {
          geometry = { type: "MultiPolygon", coordinates: polys };
        }
      } else if (
        json &&
        (json.type === "Polygon" || json.type === "MultiPolygon")
      ) {
        geometry = { type: json.type, coordinates: json.coordinates };
      }

      if (!geometry) {
        searchError.value = "该地点没有详细的轮廓数据";
        setMode(null);
        return;
      }

      cityGeoJSON.value = geometry;
      if (map.value && cityGeoJSON.value) {
        const layer = L.geoJSON(cityGeoJSON.value as any, {
          style: { color: "#ef4444", weight: 2, fillOpacity: 0.1 },
        }).addTo(map.value);
        cityLayer.value = layer;
        map.value.fitBounds(layer.getBounds());
        // setMode("polygon"); // 已在开始时设置
        updateEstimate();
        statusText.value = `已选中: ${selectedData.value?.name}`;
      }
    } catch (e) {
      searchError.value = "网络请求失败";
      console.error(e);
      setMode(null);
    } finally {
      isSearching.value = false;
    }
  };

  // 判断当前是否为"全世界"范围
  const isWorldRange = () =>
    currentMode.value === "polygon" && searchQuery.value === WORLD_ADCODE;

  const updateEstimate = () => {
    let bounds: L.LatLngBounds | null = null;

    if (currentMode.value === "rect" && rectLayer.value) {
      // @ts-ignore
      bounds = rectLayer.value.getBounds();
    } else if (currentMode.value === "polygon" && cityLayer.value) {
      // @ts-ignore
      bounds = cityLayer.value.getBounds();
    }

    if (!bounds) {
      tileCount.value = 0;
      statusText.value = "请操作左侧面板...";
      return;
    }

    const z = zoomLevel.value;

    // 全世界：横向瓦片数固定为 2^z，纵向取实际范围
    if (isWorldRange()) {
      const se = bounds.getSouthEast();
      const tMaxY = latLonToTile(se.lat, se.lng, z).y;
      const maxY = Math.min(
        Math.max(tMaxY, 0),
        Math.pow(2, z) - 1,
      );
      tileCount.value = Math.pow(2, z) * (maxY + 1);
      statusText.value = `预计瓦片: ${tileCount.value} 张 (全世界)`;
      return;
    }

    const nw = bounds.getNorthWest();
    const se = bounds.getSouthEast();
    const tMin = latLonToTile(nw.lat, nw.lng, z);
    const tMax = latLonToTile(se.lat, se.lng, z);

    tileCount.value =
      (Math.abs(tMax.x - tMin.x) + 1) * (Math.abs(tMax.y - tMin.y) + 1);
    statusText.value = `预计瓦片: ${tileCount.value} 张`;
  };

  const startDownload = async () => {
    let bounds: L.LatLngBounds | undefined;
    if (currentMode.value === "rect" && rectLayer.value) {
      // @ts-ignore
      bounds = rectLayer.value.getBounds();
    } else if (currentMode.value === "polygon" && cityLayer.value) {
      // @ts-ignore
      bounds = cityLayer.value.getBounds();
    }

    if (!bounds) return;
    if (tileCount.value > 1500) {
      alert("瓦片数量过多，请缩小范围或降低 Zoom 级别");
      return;
    }

    isDownloading.value = true;
    downloadProgress.value = 0;
    statusText.value = "准备下载...";

    const z = zoomLevel.value;
    const nw = bounds.getNorthWest();
    const se = bounds.getSouthEast();

    const tMin = latLonToTile(nw.lat, nw.lng, z);
    const tMax = latLonToTile(se.lat, se.lng, z);

    const tileLimit = Math.pow(2, z);
    const minX = Math.max(0, Math.min(tMin.x, tMax.x));
    const maxX = Math.min(tileLimit - 1, Math.max(tMin.x, tMax.x));
    const minY = Math.max(0, Math.min(tMin.y, tMax.y));
    const maxY = Math.min(tileLimit - 1, Math.max(tMin.y, tMax.y));

    const tilesX = maxX - minX + 1;
    const tilesY = maxY - minY + 1;
    const totalTiles = tilesX * tilesY;

    const canvas = document.createElement("canvas");
    canvas.width = tilesX * 256;
    canvas.height = tilesY * 256;
    const ctx = canvas.getContext("2d");

    if (!ctx) {
      isDownloading.value = false;
      return;
    }

    let loaded = 0;
    const activeBaseMap =
      baseMapOptions.find((item) => item.id === currentBaseMap.value) ||
      baseMapOptions[0];
    if (!activeBaseMap) {
      isDownloading.value = false;
      return;
    }
    const tileBaseUrl = activeBaseMap.url;

    // 待下载瓦片队列
    const pendingTiles: TileCoord[] = [];
    for (let x = minX; x <= maxX; x++) {
      for (let y = minY; y <= maxY; y++) {
        pendingTiles.push({ x, y });
      }
    }

    // 限制并发数，避免大范围（如全世界）一次性发起过多请求
    const CONCURRENCY = 16;
    const loadTile = (t: TileCoord) =>
      new Promise<void>((resolve) => {
        const img = new Image();
        img.crossOrigin = "Anonymous";
        img.src = tileBaseUrl
          .replace("{z}", z.toString())
          .replace("{y}", t.y.toString())
          .replace("{x}", t.x.toString());

        img.onload = () => {
          ctx.drawImage(img, (t.x - minX) * 256, (t.y - minY) * 256);
          loaded++;
          downloadProgress.value = Math.floor((loaded / totalTiles) * 100);
          resolve();
        };
        img.onerror = () => resolve();
      });

    await Promise.all(
      Array.from(
        { length: Math.min(CONCURRENCY, pendingTiles.length) },
        async () => {
          while (pendingTiles.length > 0) {
            const t = pendingTiles.shift();
            if (!t) break;
            await loadTile(t);
          }
        },
      ),
    );

    // 城市轮廓裁剪（全世界范围本身就是标准瓦片范围，无需裁剪）
    if (
      currentMode.value === "polygon" &&
      cityGeoJSON.value &&
      !isWorldRange()
    ) {
      statusText.value = "应用轮廓裁剪...";
      ctx.globalCompositeOperation = "destination-in";
      ctx.beginPath();

      const originWorldPixelX = minX * 256;
      const originWorldPixelY = minY * 256;

      const getCanvasPixel = (lat: number, lng: number): Point => {
        const p = project(lat, lng, z);
        return {
          x: p.x - originWorldPixelX,
          y: p.y - originWorldPixelY,
        };
      };

      drawGeoJSONPath(ctx, cityGeoJSON.value, getCanvasPixel);
      ctx.fill();
      ctx.globalCompositeOperation = "source-over";
    }

    statusText.value = "导出图片...";

    const pNW = project(nw.lat, nw.lng, z);
    const pSE = project(se.lat, se.lng, z);
    const originX = minX * 256;
    const originY = minY * 256;

    let cropX = pNW.x - originX;
    let cropY = pNW.y - originY;
    let cropW = pSE.x - pNW.x;
    let cropH = pSE.y - pNW.y;

    cropX = Math.max(0, cropX);
    cropY = Math.max(0, cropY);
    if (cropX + cropW > canvas.width) cropW = canvas.width - cropX;
    if (cropY + cropH > canvas.height) cropH = canvas.height - cropY;

    const finalCanvas = document.createElement("canvas");
    finalCanvas.width = cropW;
    finalCanvas.height = cropH;
    const finalCtx = finalCanvas.getContext("2d");

    if (finalCtx) {
      finalCtx.drawImage(
        canvas,
        cropX,
        cropY,
        cropW,
        cropH,
        0,
        0,
        cropW,
        cropH,
      );

      finalCanvas.toBlob((blob) => {
        if (blob) {
          const url = URL.createObjectURL(blob);
          const a = document.createElement("a");
          a.href = url;
          const name =
            currentMode.value === "polygon"
              ? selectedData.value?.name ?? "region"
              : "terra_snap_map";
          a.download = `${name}_z${z}.png`;
          document.body.appendChild(a);
          a.click();
          document.body.removeChild(a);
          URL.revokeObjectURL(url);
        }
        isDownloading.value = false;
        statusText.value = "下载完成";
      }, "image/png");
    }
  };

  const latLonToTile = (lat: number, lon: number, zoom: number): TileCoord => {
    const x = Math.floor(((lon + 180) / 360) * Math.pow(2, zoom));
    const y = Math.floor(
      ((1 -
        Math.log(
          Math.tan((lat * Math.PI) / 180) + 1 / Math.cos((lat * Math.PI) / 180),
        ) /
          Math.PI) /
        2) *
        Math.pow(2, zoom),
    );
    return { x, y };
  };

  const project = (lat: number, lng: number, zoom: number): Point => {
    let siny = Math.sin((lat * Math.PI) / 180);
    siny = Math.min(Math.max(siny, -0.9999), 0.9999);
    return {
      x: (256 * Math.pow(2, zoom) * (lng + 180)) / 360,
      y:
        256 *
        Math.pow(2, zoom) *
        (0.5 - Math.log((1 + siny) / (1 - siny)) / (4 * Math.PI)),
    };
  };

  const drawGeoJSONPath = (
    ctx: CanvasRenderingContext2D,
    geometry: GeoJSONResult,
    converter: (lat: number, lng: number) => Point,
  ) => {
    if (geometry.type === "Polygon") {
      drawPolygonCoordinates(ctx, geometry.coordinates, converter);
    } else if (geometry.type === "MultiPolygon") {
      geometry.coordinates.forEach((polyCoords: any[]) => {
        drawPolygonCoordinates(ctx, polyCoords, converter);
      });
    }
  };

  const drawPolygonCoordinates = (
    ctx: CanvasRenderingContext2D,
    coordinates: any[],
    converter: (lat: number, lng: number) => Point,
  ) => {
    coordinates.forEach((ring: any[]) => {
      ring.forEach((coord, index) => {
        const p = converter(coord[1], coord[0]);
        if (index === 0) ctx.moveTo(p.x, p.y);
        else ctx.lineTo(p.x, p.y);
      });
      ctx.closePath();
    });
  };
</script>

<style scoped>
  .switch {
    font-size: 12px;
    position: relative;
    display: inline-block;
    width: 3.5em;
    height: 2em;
  }

  .switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #e5e7eb;
    transition: 0.4s;
    border-radius: 30px;
    overflow: hidden;
  }

  .slider:before {
    position: absolute;
    content: "";
    height: 1.4em;
    width: 1.4em;
    border-radius: 20px;
    left: 0.3em;
    bottom: 0.3em;
    transition: 0.4s;
    transition-timing-function: cubic-bezier(0.81, -0.04, 0.38, 1.5);
    background-color: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .switch input:checked + .slider {
    background-color: #3d3d3d;
  }

  .switch input:checked + .slider:before {
    transform: translateX(1.5em);
    background-color: transparent;
    box-shadow: inset 8px -4px 0px 0px #fff;
  }

  .slider::after {
    content: "☀️";
    position: absolute;
    right: 0.5em;
    top: 50%;
    transform: translateY(-50%);
    font-size: 0.8em;
    opacity: 1;
    transition: opacity 0.3s;
  }

  .switch input:checked + .slider::after {
    opacity: 0;
  }

  :deep(.leaflet-bottom) {
    display: none;
  }
</style>
