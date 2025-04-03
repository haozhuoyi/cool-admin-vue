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
		<cl-upsert ref="Upsert" />
	</cl-crud>
</template>

<script lang="ts" setup>
import { reactive, onMounted } from 'vue';

defineOptions({
	name: "device-info",
});

import { useCrud, useTable, useUpsert, useSearch } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { useI18n } from "vue-i18n";

const { service } = useCool();
const { t } = useI18n();

// 下拉选项
const options = reactive({
	zuhu: [] as any[],
	dingdan: [] as any[],
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

// 获取订单列表
async function getDingdanList() {
	try {
		const res = await service.dingdan.info.page({
			page: 1,
			size: 1000
		});
		
		if (res && Array.isArray(res.list)) {
			options.dingdan = res.list.map((e: any) => ({
				label: `订单${e.id}`,
				value: String(e.id)
			}));
			return true;
		} else {
			console.warn('获取订单列表数据格式异常：', res);
			return false;
		}
	} catch (err) {
		console.error('获取订单列表失败：', err);
		return false;
	}
}

// 页面加载时获取列表数据
onMounted(() => {
	getZuhuList();
	getDingdanList();
});

// cl-upsert
const Upsert = useUpsert({
	items: [
		{
			label: t("设备编号"),
			prop: "number",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("设备名称"),
			prop: "name",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("选择租户信息"),
			prop: "zuhuId",
			hook: {
				bind: [(value) => {
					if (!value) return '';
					try {
						let ids = value;
						if (typeof ids === 'string') {
							ids = JSON.parse(ids || "[]");
						}
						if (!Array.isArray(ids)) {
							ids = [];
						}
						return options.zuhu
							.filter(e => ids.includes(e.value) || ids.includes(String(e.value)))
							.map(e => e.label)
							.join("、") || "";
					} catch (e) {
						console.warn('解析zuhuId失败:', e);
						return value;
					}
				}],
				submit: [(value) => {
					return '[]';
				}]
			},
			component: { 
				name: "el-input", 
				props: { 
					clearable: true,
					disabled: true,
					placeholder: "不可手动选择"
				} 
			},
			span: 12,
		},
		{
			label: t("选择订单编号"),
			prop: "dingdanId",
			hook: {
				bind: [(value) => {
					if (!value) return '';
					try {
						let ids = value;
						if (typeof ids === 'string') {
							ids = JSON.parse(ids || "[]");
						}
						if (!Array.isArray(ids)) {
							ids = [];
						}
						return options.dingdan
							.filter(e => ids.includes(e.value) || ids.includes(String(e.value)))
							.map(e => e.label)
							.join("、") || "";
					} catch (e) {
						console.warn('解析dingdanId失败:', e);
						return value;
					}
				}],
				submit: [(value) => {
					return '[]';
				}]
			},
			component: { 
				name: "el-input", 
				props: { 
					clearable: true,
					disabled: true,
					placeholder: "不可手动选择"
				} 
			},
			span: 12,
		},
	],
	// 添加 onOpen 钩子，在表单打开时获取最新列表
	async onOpen(data?: any) {
		await Promise.all([
			getZuhuList(),
			getDingdanList()
		]);
	}
});

// cl-table
const Table = useTable({
	columns: [
		{ type: "selection" },
		{ label: t("ID"), prop: "id", minWidth: 80 },
		{ label: t("设备编号"), prop: "number", minWidth: 120 },
		{ label: t("设备名称"), prop: "name", minWidth: 120 },
		{ 
			label: t("租户信息"), 
			prop: "zuhuId", 
			minWidth: 120,
			formatter: (row) => {
				try {
					let ids = row.zuhuId;
					if (typeof ids === 'string') {
						ids = JSON.parse(ids || "[]");
					}
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
			label: t("订单编号"), 
			prop: "dingdanId", 
			minWidth: 120,
			formatter: (row) => {
				try {
					let ids = row.dingdanId;
					if (typeof ids === 'string') {
						ids = JSON.parse(ids || "[]");
					}
					if (!Array.isArray(ids)) {
						ids = [];
					}
					return options.dingdan
						.filter(e => ids.includes(e.value) || ids.includes(String(e.value)))
						.map(e => e.label)
						.join("、") || "-";
				} catch(err) {
					console.warn("格式化dingdanId失败:", err);
					return "-";
				}
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
		service: service.device.info,
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
