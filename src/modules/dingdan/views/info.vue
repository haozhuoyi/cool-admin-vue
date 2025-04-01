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
defineOptions({
	name: "dingdan-info",
});

import { useCrud, useTable, useUpsert, useSearch } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { useI18n } from "vue-i18n";

const { service } = useCool();
const { t } = useI18n();

// cl-upsert
const Upsert = useUpsert({
	items: [
		{
			label: t("选择租户"),
			prop: "zuhuId",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("选择设备"),
			prop: "deviceId",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("订单时间"),
			prop: "datetime",
			component: { name: "el-input", props: { clearable: true } },
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
			label: t("已还金额"),
			prop: "yihuanJine",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("待还金额"),
			prop: "daihuanJine",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("总金额"),
			prop: "zongJine",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("还款日期"),
			prop: "hkDatetime",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("分期数量"),
			prop: "fenqiCount",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("订单状态"),
			prop: "status",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
	],
});

// cl-table
const Table = useTable({
	columns: [
		{ type: "selection" },
		{ label: t("租户ID"), prop: "zuhuId", minWidth: 120 },
		{ label: t("设备ID"), prop: "deviceId", minWidth: 120 },
		{ label: t("订单时间"), prop: "datetime", minWidth: 120 },
		{ label: t("订单来源"), prop: "ori", minWidth: 120 },
		{ label: t("套餐类型"), prop: "taocanType", minWidth: 120 },
		{ label: t("已还金额"), prop: "yihuanJine", minWidth: 120 },
		{ label: t("待还金额"), prop: "daihuanJine", minWidth: 120 },
		{ label: t("总金额"), prop: "zongJine", minWidth: 120 },
		{ label: t("还款日期"), prop: "hkDatetime", minWidth: 120 },
		{ label: t("分期数量"), prop: "fenqiCount", minWidth: 120 },
		{ label: t("订单状态"), prop: "status", minWidth: 120 },
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
