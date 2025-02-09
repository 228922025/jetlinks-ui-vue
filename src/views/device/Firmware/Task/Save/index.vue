<template lang="">
    <j-modal
        :title="data.id ? '查看' : '新增' + '任务'"
        ok-text="确认"
        cancel-text="取消"
        :visible="true"
        width="700px"
        :confirm-loading="loading"
        @cancel="handleCancel"
        @ok="handleOk"
    >
        <j-form
            class="form"
            layout="vertical"
            :model="formData"
            name="basic"
            autocomplete="off"
            ref="formRef"
            :rules="rules"
        >
            <j-row :gutter="[24, 0]">
                <j-col :span="24">
                    <j-form-item label="任务名称" name="name">
                        <j-input
                            placeholder="请输入任务名称"
                            v-model:value="formData.name"
                            :disabled="view"
                    /></j-form-item>
                </j-col>
                <j-col :span="24"
                    ><j-form-item label="推送方式" name="mode">
                        <j-select
                            v-model:value="formData.mode"
                            :options="[
                                { label: '平台推送', value: 'push' },
                                { label: '设备拉取', value: 'pull' },
                            ]"
                            placeholder="请选择推送方式"
                            allowClear
                            show-search
                            :filter-option="filterOption"
                            @change="changeMode"
                            :disabled="view"
                        /> </j-form-item
                ></j-col>
                <j-col :span="12" v-if="formData.mode === 'push'"
                    ><j-form-item
                        label="响应超时时间"
                        name="responseTimeoutSeconds"
                    >
                        <j-input-number
                            placeholder="请输入响应超时时间(秒)"
                            style="width: 100%"
                            :min="1"
                            :max="99999"
                            :disabled="view"
                            v-model:value="
                                formData.responseTimeoutSeconds
                            " /></j-form-item
                ></j-col>
                <j-col
                    :span="formData.mode === 'push' ? 12 : 24"
                    v-if="formData.mode === 'push' || formData.mode === 'pull'"
                    ><j-form-item label="升级超时时间" name="timeoutSeconds">
                        <j-input-number
                            placeholder="请输入升级超时时间(秒)"
                            style="width: 100%"
                            :min="1"
                            :max="99999"
                            :disabled="view"
                            v-model:value="
                                formData.timeoutSeconds
                            " /></j-form-item
                ></j-col>
                <j-col :span="12" v-if="!!formData.mode"
                    ><j-form-item label="升级设备" name="releaseType">
                        <j-radio-group
                            v-model:value="formData.releaseType"
                            button-style="solid"
                            @change="changeShareCluster"
                            :disabled="view"
                        >
                            <j-radio value="all">所有设备</j-radio>
                            <j-radio value="part">选择设备</j-radio>
                        </j-radio-group>
                    </j-form-item>
                </j-col>
                <j-col :span="12" v-if="formData.releaseType === 'part'">
                    <j-form-item label="选择设备" name="deviceId">
                        <SelectDevices
                            v-model:modelValue="formData.deviceId"
                            :data="data"
                        ></SelectDevices> </j-form-item
                ></j-col>
                <j-col :span="24">
                    <j-form-item label="说明" name="description">
                        <j-textarea
                            placeholder="请输入说明"
                            v-model:value="formData.description"
                            :maxlength="200"
                            :rows="3"
                            showCount
                            :disabled="view"
                        /> </j-form-item
                ></j-col>
            </j-row>
        </j-form>
    </j-modal>
</template>
<script lang="ts" setup name="TaskPage">
import { queryProduct, saveTask } from '@/api/device/firmware';
import type { FormInstance } from 'ant-design-vue';
import SelectDevices from './SelectDevices.vue';

const props = defineProps({
    data: {
        type: Object,
        default: () => {},
    },
});

const formRef = ref<FormInstance>();

const route = useRoute();
const loading = ref(false);
const productOptions = ref([]);
const emit = defineEmits(['change']);

const firmwareId = route.query.id;
const productId = route.query.productId;
const view = props.data.view;

const formData: any = ref({
    name: '',
    mode: undefined,
    responseTimeoutSeconds: '',
    timeoutSeconds: '',
    releaseType: 'all',
    deviceId: undefined,
    description: '',
});

const rules = {
    name: [
        { required: true, message: '请输入任务名称' },
        { max: 64, message: '最多可输入64个字符' },
    ],
    mode: [{ required: true, message: '请选择推送方式' }],
    responseTimeoutSeconds: [{ required: true, message: '请输入响应超时时间' }],
    timeoutSeconds: [{ required: true, message: '请输入升级超时时间' }],
    releaseType: [{ required: true }],
    deviceId: [{ required: true, message: '请选择设备' }],
    description: [{ max: 200, message: '最多可输入200个字符' }],
};

const filterOption = (input: string, option: any) => {
    return option.label.toLowerCase().indexOf(input.toLowerCase()) >= 0;
};

const onSubmit = async () => {
    const params = await formRef.value?.validate();
    loading.value = true;
    const resp = await saveTask({
        ...params,
        firmwareId,
        productId,
    });
    loading.value = false;
    resp.success && emit('change', true);
};

const handleOk = () => {
    return view ? emit('change', false) : onSubmit();
};
const handleCancel = () => {
    emit('change', false);
};

const changeShareCluster = () => {
    formData.value.deviceId = undefined;
};

onMounted(() => {
    queryProduct({
        paging: false,
        terms: [{ column: 'state', value: 1 }],
        sorts: [{ name: 'createTime', order: 'desc' }],
    }).then((resp: any) => {
        productOptions.value = resp.result.map((item: any) => ({
            value: item.id,
            label: item.name,
        }));
    });
});
watch(
    () => props.data,
    (value) => {
        if (value.id) {
            formData.value = {
                ...value,
                mode: value.mode.value,
                releaseType: value?.deviceId ? 'part' : 'all',
            };
        }
    },
    { immediate: true, deep: true },
);
</script>

<style lang="less" scoped></style>
