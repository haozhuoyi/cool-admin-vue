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
			<!-- 自定义渠道选择组件插槽 -->
			<template #slot-qudao="{ scope }">
				<cl-select v-model="scope.qudaoIds" :options="options.qudao" multiple clearable />
			</template>
		</cl-upsert>
	</cl-crud>
</template>

<script lang="ts" setup>
defineOptions({
	name: "xiaoshou-info",
});

import { useCrud, useTable, useUpsert, useSearch } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { useI18n } from "vue-i18n";
import { reactive, onMounted, computed } from "vue";

const { service } = useCool();
const { t } = useI18n();

// 渠道列表
const options = reactive({
	qudao: [] as any[]
});

// 获取渠道列表
async function getQudaoList() {
	try {
		const res = await service.xiaoshou.qudao.page({
			page: 1,
			size: 1000
		});
		
		if (res && Array.isArray(res.list)) {
			// 确保value是字符串类型
			options.qudao = res.list.map((e: any) => ({
				label: e.name,
				value: String(e.id)  // 转为字符串
			}));
			
			console.log('渠道列表更新成功：', options.qudao);
			return true;
		} else {
			console.warn('获取渠道列表数据格式异常：', res);
			return false;
		}
	} catch (err) {
		console.error('获取渠道列表失败：', err);
		return false;
	}
}

// 页面加载时获取渠道列表
onMounted(() => {
	getQudaoList();
});

// 创建一个计算属性，用于包装渠道选项
const qudaoOptions = computed(() => options.qudao);

// cl-upsert
const Upsert = useUpsert({
	items: [
		{
			label: t("名称"),
			prop: "name",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("选择关联渠道"),
			prop: "qudaoIds",
			hook: {
				bind: [(value) => {
					// 如果值已经是数组就直接使用，否则尝试解析JSON
					if (Array.isArray(value)) {
						// 确保数组中的值是字符串
						return value.map(id => String(id));
					} else if (typeof value === 'string') {
						try {
							const parsed = JSON.parse(value);
							// 确保解析后的数组中值是字符串
							return Array.isArray(parsed) ? parsed.map(id => String(id)) : [];
						} catch (e) {
							console.warn('解析qudaoIds失败:', e);
							return [];
						}
					}
					return value ? [String(value)] : [];
				}],
				submit: [(value) => {
					// 确保提交的始终是数组格式
					return Array.isArray(value) ? value : [];
				}]
			},
			component: { 
				name: "slot-qudao"
			},
			span: 12,
		},
		{
			label: t("选择名下租户"),
			prop: "zuhuIds",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("联系方式"),
			prop: "phone",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("备注"),
			prop: "note",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
	],
	// 添加 onOpen 钩子，在表单打开时获取最新的渠道列表
	async onOpen(data?: any) {
		await getQudaoList();
		console.log('表单打开，数据:', data);
		// 打印渠道数据，帮助调试
		if (data && data.qudaoIds) {
			console.log('渠道ID:', data.qudaoIds, '类型:', typeof data.qudaoIds);
		}
	}
});

// cl-table
const Table = useTable({
	columns: [
		{ type: "selection" },
		{ label: t("名称"), prop: "name", minWidth: 120 },
		{ 
			label: t("关联渠道"), 
			prop: "qudaoIds",
			minWidth: 120,
			formatter: (row) => {
				try {
					// 处理可能是数组或字符串的情况
					let ids = row.qudaoIds;
					
					// 如果是字符串，尝试解析
					if (typeof ids === 'string') {
						ids = JSON.parse(ids || "[]");
					}
					
					// 确保是数组
					if (!Array.isArray(ids)) {
						ids = [];
					}
					
					return options.qudao
						.filter(e => ids.includes(e.value) || ids.includes(String(e.value)))
						.map(e => e.label)
						.join("、") || "-";
				} catch(err) {
					console.warn("格式化qudaoIds失败:", err);
					return "-";
				}
			}
		},
		{ label: t("名下租户"), prop: "zuhuIds", minWidth: 120 },
		{ label: t("联系方式"), prop: "phone", minWidth: 120 },
		{ label: t("备注"), prop: "note", minWidth: 120 },
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
		service: service.xiaoshou.info,
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
