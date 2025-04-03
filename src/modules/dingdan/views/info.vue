<template>
	<cl-crud ref="Crud">
		<cl-row>
			<!-- 刷新按钮 -->
			<cl-refresh-btn />
			<!-- 新增按钮 -->
			<cl-add-btn />
			<!-- 删除按钮 -->
			<cl-multi-delete-btn />
			<cl-flex1 />
			<!-- 条件搜索 -->
			<cl-search ref="Search" />
		</cl-row>

		<cl-row>
			<!-- 数据表格 -->
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<!-- 分页控件 -->
			<cl-pagination />
		</cl-row>

		<!-- 新增、编辑 -->
		<cl-upsert ref="Upsert">
			<!-- 租户选择插槽 -->
			<template #slot-zuhu="{ scope }">
				<cl-select v-model="scope.zuhuId" :options="options.zuhu" clearable />
			</template>
			<!-- 设备选择插槽 -->
			<template #slot-device="{ scope }">
				<cl-select v-model="scope.deviceId" :options="options.device" clearable />
			</template>
		</cl-upsert>
	</cl-crud>
</template>

<script lang="ts" setup>
defineOptions({
	name: "dingdan-info",
});

import { useCrud, useTable, useUpsert, useSearch } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { useI18n } from "vue-i18n";
import { reactive, onMounted } from 'vue';

const { service } = useCool();
const { t } = useI18n();

// 下拉选项
const options = reactive({
	zuhu: [] as any[],
	device: [] as any[],
	status: [
		{ label: '还款中', value: 0 },
		{ label: '已逾期', value: 1 },
		{ label: '已还清', value: 2 }
	]
});

// 获取租户列表
async function getZuhuList() {
	try {
		const res = await service.zuhu.info.page({
			page: 1,
			size: 1000
		});
		
		if (res && Array.isArray(res.list)) {
			options.zuhu = res.list.map((e: any) => ({
				label: e.name,
				value: String(e.id)
			}));
			return true;
		} else {
			console.warn('获取租户列表数据格式异常：', res);
			return false;
		}
	} catch (err) {
		console.error('获取租户列表失败：', err);
		return false;
	}
}

// 获取设备列表
async function getDeviceList() {
	try {
		const res = await service.device.info.page({
			page: 1,
			size: 1000
		});
		
		if (res && Array.isArray(res.list)) {
			options.device = res.list.map((e: any) => ({
				label: e.name || e.number,  // 优先使用设备名称，如果没有则使用设备编号
				value: String(e.id)
			}));
			return true;
		} else {
			console.warn('获取设备列表数据格式异常：', res);
			return false;
		}
	} catch (err) {
		console.error('获取设备列表失败：', err);
		return false;
	}
}

// 页面加载时获取列表数据
onMounted(() => {
	getZuhuList();
	getDeviceList();
});

// cl-upsert
const Upsert = useUpsert({
	items: [
		{
			label: t("选择租户"),
			prop: "zuhuId",
			hook: {
				bind: [(value) => {
					if (!value) return undefined;
					try {
						// 如果是字符串，尝试解析 JSON
						if (typeof value === 'string') {
							const parsed = JSON.parse(value);
							return Array.isArray(parsed) ? parsed[0] : parsed;
						}
						// 如果是数组，取第一个元素
						if (Array.isArray(value)) {
							return value[0];
						}
						// 如果是对象，直接返回
						return value;
					} catch (e) {
						console.warn('解析zuhuId失败:', e);
						return value;
					}
				}],
				submit: [(value) => {
					if (!value) return '[]';
					const arr = Array.isArray(value) ? value : [value];
					return arr;
				}]
			},
			component: { 
				name: "slot-zuhu"
			},
			required: true,
			span: 12,
		},
		{
			label: t("选择设备"),
			prop: "deviceId",
			hook: {
				bind: [(value) => {
					if (!value) return undefined;
					try {
						// 如果是字符串，尝试解析 JSON
						if (typeof value === 'string') {
							const parsed = JSON.parse(value);
							return Array.isArray(parsed) ? parsed[0] : parsed;
						}
						// 如果是数组，取第一个元素
						if (Array.isArray(value)) {
							return value[0];
						}
						// 如果是对象，直接返回
						return value;
					} catch (e) {
						console.warn('解析deviceId失败:', e);
						return value;
					}
				}],
				submit: [(value) => {
					if (!value) return '[]';
					const arr = Array.isArray(value) ? value : [value];
					return arr;
				}]
			},
			component: { 
				name: "slot-device"
			},
			required: true,
			span: 12,
		},
		{
			label: t("订单时间"),
			prop: "datetime",
			component: { 
				name: "el-date-picker", 
				props: { 
					type: "datetime",
					valueFormat: "YYYY-MM-DD HH:mm:ss",
					placeholder: "请选择订单时间",
					clearable: true,
					style: {
						width: "100%"
					}
				} 
			},
			required: true,
			span: 12,
		},
		{
			label: t("订单来源"),
			prop: "ori",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("套餐类型"),
			prop: "taocanType",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("总金额"),
			prop: "zongJine",
			rules: [
				{ required: true, message: "请输入总金额" },
				{ pattern: /^\d+(\.\d{1,2})?$/, message: "请输入有效的数字，最多保留两位小数" },
				{
					validator: (rule: any, value: any, callback: any) => {
						const yihuanJine = Upsert.value?.form?.yihuanJine;
						if (Number(value) < Number(yihuanJine || 0)) {
							callback(new Error("总金额必须大于等于已还金额"));
						} else {
							callback();
						}
					},
					trigger: "blur"
				}
			],
			component: { 
				name: "el-input", 
				props: { 
					clearable: true,
					onChange(val: string) {
						const form = Upsert.value?.form;
						if (form) {
							const zongJine = Number(val) || 0;
							const yihuanJine = Number(form.yihuanJine) || 0;
							form.daihuanJine = (zongJine - yihuanJine).toString();
						}
					}
				} 
			},
			span: 12,
		},
		{
			label: t("已还金额"),
			prop: "yihuanJine",
			rules: [
				{ required: true, message: "请输入已还金额" },
				{ pattern: /^\d+(\.\d{1,2})?$/, message: "请输入有效的数字，最多保留两位小数" },
				{
					validator: (rule: any, value: any, callback: any) => {
						const zongJine = Upsert.value?.form?.zongJine;
						if (Number(value) > Number(zongJine || 0)) {
							callback(new Error("已还金额不能大于总金额"));
						} else {
							callback();
						}
					},
					trigger: "blur"
				}
			],
			component: { 
				name: "el-input", 
				props: { 
					clearable: true,
					onChange(val: string) {
						const form = Upsert.value?.form;
						if (form) {
							const zongJine = Number(form.zongJine) || 0;
							const yihuanJine = Number(val) || 0;
							form.daihuanJine = (zongJine - yihuanJine).toString();
						}
					}
				} 
			},
			span: 12,
		},
		{
			label: t("待还金额"),
			prop: "daihuanJine",
			component: { 
				name: "el-input", 
				props: { 
					clearable: true,
					disabled: true,
					placeholder: "总金额-已还金额（自动计算）"
				} 
			},
			span: 12,
		},
		{
			label: t("还款日期(每个月几号)"),
			prop: "hkDatetime",
			rules: [
				{ required: true, message: "请输入还款日期" },
				{ pattern: /^([1-9]|[12]\d|28)$/, message: "请输入1-28之间的数字" }
			],
			component: { 
				name: "el-input", 
				props: { 
					clearable: true,
					type: "number",
					min: 1,
					max: 28,
					placeholder: "请输入1-28之间的数字"
				} 
			},
			span: 12,
		},
		{
			label: t("分期数量"),
			prop: "fenqiCount",
			required: true,
			component: { 
				name: "el-input", 
				props: { 
					clearable: true,
					type: "number",
					min: 1,
					placeholder: "请输入分期数量"
				} 
			},
			span: 12,
		},
		{
			label: t("订单状态"),
			prop: "status",
			required: true,
			component: { 
				name: "cl-select",
				props: { 
					clearable: true,
					options: options.status
				}
			},
			span: 12,
		},
	],
	// 添加 onOpen 钩子，在表单打开时获取最新列表
	async onOpen(data?: any) {
		await Promise.all([
			getZuhuList(),
			getDeviceList()
		]);
	}
});

// cl-table
const Table = useTable({
	columns: [
		{ type: "selection" },
		{ label: t("ID"), prop: "id", minWidth: 80 },
		{ 
			label: t("租户"), 
			prop: "zuhuId", 
			minWidth: 120,
			formatter: (row) => {
				try {
					// 处理可能是数组或字符串的情况
					let ids = row.zuhuId;
					
					// 如果是字符串，尝试解析
					if (typeof ids === 'string') {
						ids = JSON.parse(ids || "[]");
					}
					
					// 确保是数组
					if (!Array.isArray(ids)) {
						ids = [];
					}
					
					return options.zuhu
						.filter(e => ids.includes(e.value) || ids.includes(String(e.value)))
						.map(e => e.label)
						.join("、") || "-";
				} catch(err) {
					console.warn("格式化zuhuId失败:", err);
					return "-";
				}
			}
		},
		{ 
			label: t("设备"), 
			prop: "deviceId", 
			minWidth: 120,
			formatter: (row) => {
				try {
					// 处理可能是数组或字符串的情况
					let ids = row.deviceId;
					
					// 如果是字符串，尝试解析
					if (typeof ids === 'string') {
						ids = JSON.parse(ids || "[]");
					}
					
					// 确保是数组
					if (!Array.isArray(ids)) {
						ids = [];
					}
					
					return options.device
						.filter(e => ids.includes(e.value) || ids.includes(String(e.value)))
						.map(e => e.label)
						.join("、") || "-";
				} catch(err) {
					console.warn("格式化deviceId失败:", err);
					return "-";
				}
			}
		},
		{ label: t("订单时间"), prop: "datetime", minWidth: 120 },
		{ label: t("订单来源"), prop: "ori", minWidth: 120 },
		{ label: t("套餐类型"), prop: "taocanType", minWidth: 120 },
		{ label: t("总金额"), prop: "zongJine", minWidth: 120 },
		{ label: t("已还金额"), prop: "yihuanJine", minWidth: 120 },
		{ label: t("待还金额"), prop: "daihuanJine", minWidth: 120 },
		{ label: t("还款日期(每个月几号)"), prop: "hkDatetime", minWidth: 120 },
		{ label: t("分期数量"), prop: "fenqiCount", minWidth: 120 },
		{ 
			label: t("订单状态"), 
			prop: "status", 
			minWidth: 120,
			formatter: (row) => {
				const status = options.status.find(e => e.value === row.status);
				return status ? status.label : '-';
			}
		},
		{
			label: t("创建时间"),
			prop: "createTime",
			minWidth: 170,
			sortable: "desc",
			component: { name: "cl-date-text" },
		},
		{
			label: t("更新时间"),
			prop: "updateTime",
			minWidth: 170,
			sortable: "custom",
			component: { name: "cl-date-text" },
		},
		{ type: "op", buttons: ["edit", "delete"] },
	],
});

// cl-search
const Search = useSearch();

// cl-crud
const Crud = useCrud(
	{
		service: service.dingdan.info,
	},
	(app) => {
		app.refresh();
	},
);

// 刷新
function refresh(params?: any) {
	Crud.value?.refresh(params);
}
</script>
