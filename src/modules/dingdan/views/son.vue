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
import { reactive } from 'vue';
import { useCrud, useTable, useUpsert, useSearch } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { useI18n } from "vue-i18n";

const { service } = useCool();
const { t } = useI18n();

// 状态选项
const options = reactive({
	status: [
		{ label: '未还款', value: 0 },
		{ label: '已逾期', value: 1 },
		{ label: '已还款', value: 2 }
	]
});

// cl-upsert
const Upsert = useUpsert({
	items: [
		{
			label: t("选择订单"),
			prop: "dingdanId",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("金额"),
			prop: "jine",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("还款日期"),
			prop: "datetime",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("订单状态"),
			prop: "status",
			component: { 
				name: "cl-select", 
				props: { 
					clearable: true,
					options: options.status
				}
			},
			span: 12
		},
	],
});

// cl-table
const Table = useTable({
	columns: [
		{ type: "selection" },
		{ label: t("订单ID"), prop: "dingdanId", minWidth: 120 },
		{ label: t("金额"), prop: "jine", minWidth: 120 },
		{ label: t("还款日期"), prop: "datetime", minWidth: 120 },
		{ 
			label: t("订单状态"), 
			prop: "status", 
			minWidth: 120,
			formatter: (row) => {
				const item = options.status.find(e => e.value === row.status);
				return item ? item.label : '-';
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
		service: service.dingdan.son,
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
