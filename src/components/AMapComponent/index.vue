<template>
    <div
        :style="props.style || { width: '100%', height: '100%' }"
        :class="props.class"
    >
        <el-amap v-if="amapKey" :zooms="[3, 20]" @init="initMap" ref="mapRef">
            <template v-if="isOpenUi">
                <template v-if="uiLoading">
                    <slot></slot>
                </template>
            </template>
            <template v-else><slot></slot></template>
        </el-amap>
        <JEmpty v-else description="请配置高德地图key" style="padding: 20%" />
    </div>
</template>

<script lang="ts" setup>
import { CSSProperties, PropType } from 'vue';
import AMap, { initAMapApiLoader } from '@vuemap/vue-amap';
import '@vuemap/vue-amap/dist/style.css';
import { getAMapUiPromise } from './utils';
import { useSystem } from '@/store/system';

const system = useSystem();
interface AMapProps {
    style?: CSSProperties;
    class?: string;
    AMapUI?: string | boolean;
}
const amapKey = system.$state.configInfo.amap?.apiKey;

initAMapApiLoader({
    key: amapKey || '',
});

const props = defineProps({
    style: Object as PropType<AMapProps['style']>,
    class: String as PropType<AMapProps['class']>,
    AMapUI: [String, Boolean],
    center: Array,
});

const mapRef = ref();

const uiLoading = ref<boolean>(false);

const map = ref<any>(null);

const isOpenUi = computed(() => {
    return 'AMapUI' in props || props.AMapUI;
});

const getAMapUI = () => {
    const version = typeof props.AMapUI === 'string' ? props.AMapUI : '1.1';
    getAMapUiPromise(version).then(() => {
        uiLoading.value = true;
    });
};

const marker = ref<any[]>([]);

const initMap = (e: any) => {
    map.value = e;
    if (isOpenUi.value) {
        getAMapUI();
    }
};
</script>

<style lang="less" scoped>
</style>