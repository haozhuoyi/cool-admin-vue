<template>
	<cl-crud ref="Crud">
		<cl-row>
			<!-- 刷新按钮 -->
			<cl-refresh-btn />
			<!-- 新增按钮 -->
			<cl-add-btn />
			<!-- 删除按钮 -->
			<cl-multi-delete-btn />
			<!-- 租借按钮 -->
			<el-button type="primary" @click="openBorrowForm" :disabled="!isRowSelected">
				租借
			</el-button>
			<cl-flex1 />
			<!-- 条件搜索 -->
			<cl-search ref="Search" />
		</cl-row>

		<cl-row>
			<!-- 数据表格 -->
			<cl-table ref="Table" @selection-change="handleSelectionChange" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<!-- 分页控件 -->
			<cl-pagination />
		</cl-row>

		<!-- 新增、编辑 -->
		<cl-upsert ref="Upsert">
			<!-- 自定义设备类型选择组件插槽 -->
			<template #slot-deviceType="{ scope }">
				<cl-select v-model="scope.deviceTypeId" :options="options.deviceType" clearable />
			</template>
			<!-- 自定义项目选择组件插槽 -->
			<template #slot-project="{ scope }">
				<cl-select v-model="scope.projectId" :options="options.project" clearable />
			</template>
			<!-- 自定义用户选择组件插槽 -->
			<template #slot-user="{ scope }">
				<cl-select v-model="scope.userId" :options="options.user" clearable />
			</template>
		</cl-upsert>

		<!-- 租借表单 -->
		<cl-form ref="BorrowForm" />
	</cl-crud>
</template>

<script lang="ts" setup>
defineOptions({
	name: "xsw-device",
});

import { useCrud, useTable, useUpsert, useSearch, useForm } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { useI18n } from "vue-i18n";
import { reactive, onMounted, computed, ref } from "vue";
import { ElMessage } from "element-plus";
import { useUserStore } from "/$/base/store/user";

const { service, mitt } = useCool();
const { t } = useI18n();
const userStore = useUserStore();

// 选中的行
const selectedRows = ref<any[]>([]);
// 是否选中行
const isRowSelected = computed(() => selectedRows.value.length > 0);

// 选项列表
const options = reactive({
	deviceType: [] as any[],
	project: [] as any[],
	user: [] as any[],
	status: [
		{
			label: '空闲',
			value: 0,
			type: 'success' // 绿色
		},
		{
			label: '租借中',
			value: 1,
			type: 'warning' // 黄色
		}
	]
});

// 获取设备类型列表
async function getDeviceTypeList() {
	try {
		const res = await service.xsw.type.page({
			page: 1,
			size: 1000
		});
		
		if (res && Array.isArray(res.list)) {
			options.deviceType = res.list.map((e: any) => ({
				label: e.name,
				value: String(e.id)
			}));
			
			console.log('设备类型列表更新成功：', options.deviceType);
			return true;
		} else {
			console.warn('获取设备类型列表数据格式异常：', res);
			return false;
		}
	} catch (err) {
		console.error('获取设备类型列表失败：', err);
		return false;
	}
}

// 获取项目列表
async function getProjectList() {
	try {
		const res = await service.xsw.project.page({
			page: 1,
			size: 1000
		});
		
		if (res && Array.isArray(res.list)) {
			options.project = res.list.map((e: any) => ({
				label: e.name,
				value: String(e.id)
			}));
			
			console.log('项目列表更新成功：', options.project);
			return true;
		} else {
			console.warn('获取项目列表数据格式异常：', res);
			return false;
		}
	} catch (err) {
		console.error('获取项目列表失败：', err);
		return false;
	}
}

// 获取用户列表
async function getUserList() {
	try {
		const res = await service.base.sys.user.page({
			page: 1,
			size: 1000
		});
		
		if (res && Array.isArray(res.list)) {
			options.user = res.list.map((e: any) => ({
				label: e.name || e.nickName || e.username || '未命名用户',
				value: String(e.id)
			}));
			
			console.log('用户列表更新成功：', options.user);
			return true;
		} else {
			console.warn('获取用户列表数据格式异常：', res);
			return false;
		}
	} catch (err) {
		console.error('获取用户列表失败：', err);
		return false;
	}
}

// 表格选择变化
function handleSelectionChange(rows: any[]) {
	selectedRows.value = rows;
}

// 获取当前登录用户
function getCurrentUser() {
	// 从pinia store获取当前用户信息
	const userInfo = userStore.info;
	
	if (!userInfo) {
		return {
			id: '',
			name: '未知用户'
		};
	}
	
	return {
		id: userInfo.id,
		name: userInfo.name || userInfo.nickName || userInfo.username || '未命名用户'
	};
}

// 租借表单
const BorrowForm = useForm();

// 打开租借表单
function openBorrowForm() {
	if (selectedRows.value.length === 0) {
		ElMessage.warning('请至少选择一个设备');
		return;
	}

	if (selectedRows.value.length > 1) {
		ElMessage.warning('只能选择一个设备进行租借');
		return;
	}

	// 获取选中的设备
	const selectedDevice = selectedRows.value[0];

	// 如果设备已经是租借中状态，提示无法再次租借
	if (selectedDevice.status === 1) {
		ElMessage.warning('该设备已处于租借中状态，无法再次租借');
		return;
	}

	// 获取当前登录用户
	const currentUser = getCurrentUser();
	
	if (!currentUser.id) {
		ElMessage.warning('无法获取当前用户信息，请重新登录');
		return;
	}

	// 打开表单
	BorrowForm.value?.open({
		title: '租借设备',
		width: '550px',
		items: [
			{
				label: t("设备名称"),
				prop: "deviceName",
				component: {
					name: "el-input",
					props: {
						disabled: true
					}
				},
				value: selectedDevice.name,
				span: 24,
			},
			{
				label: t("选择项目"),
				prop: "projectId",
				component: {
					name: "cl-select",
					props: {
						options: options.project,
						clearable: true
					}
				},
				span: 24,
				required: true,
			},
			{
				label: t("位置"),
				prop: "location",
				component: {
					name: "el-input",
					props: {
						type: "textarea",
						rows: 2
					}
				},
				span: 24,
			},
			{
				label: t("租借人"),
				prop: "userId",
				component: {
					name: "el-input",
					props: {
						disabled: true
					}
				},
				value: currentUser.name,
				span: 24,
			},
		],
		on: {
			submit: async (data, { done, close }) => {
				try {
					// 更新设备状态为租借中
					await service.xsw.device.update({
						id: selectedDevice.id,
						status: 1, // 租借中
						projectId: data.projectId ? [data.projectId] : null, // 设置当前所处项目
						location: data.location, // 设置当前位置
						userId: [currentUser.id], // 设置当前借取人
					});

					ElMessage.success('租借成功');
					close();
					refresh();
				} catch (err) {
					console.error('租借失败:', err);
					ElMessage.error('租借失败');
					done();
				}
			}
		}
	});
}

// 页面加载时获取列表数据
onMounted(() => {
	getDeviceTypeList();
	getProjectList();
	getUserList();
});

// 创建计算属性
const deviceTypeOptions = computed(() => options.deviceType);
const projectOptions = computed(() => options.project);
const userOptions = computed(() => options.user);

// cl-upsert
const Upsert = useUpsert({
	items: [
		{
			label: t("名称"),
			prop: "name",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
			required: true,
		},
		{
			label: t("编号"),
			prop: "number",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
			required: true,
		},
		{
			label: t("选择当前所处项目"),
			prop: "projectId",
			hook: {
				bind: [(value) => {
					return value ? String(value) : undefined;
				}],
				submit: [(value) => {
					return value ? [value] : null;
				}]
			},
			component: { 
				name: "slot-project"
			},
			span: 12,
		},
		{
			label: t("当前状态"),
			prop: "status",
			value: 0, // 默认值为"空闲"
			component: { 
				name: "cl-select", 
				props: { 
					options: options.status,
					clearable: true 
				} 
			},
			span: 12,
			required: true,
		},
		{
			label: t("选择设备类型"),
			prop: "deviceTypeId",
			hook: {
				bind: [(value) => {
					return value ? String(value) : undefined;
				}],
				submit: [(value) => {
					return value ? [value] : null;
				}]
			},
			component: { 
				name: "slot-deviceType"
			},
			span: 12,
		},
		{
			label: t("当前位置"),
			prop: "location",
			component: { name: "el-input", props: { clearable: true } },
			span: 12,
		},
		{
			label: t("选择当前借取人"),
			prop: "userId",
			hook: {
				bind: [(value) => {
					return value ? String(value) : undefined;
				}],
				submit: [(value) => {
					return value ? [value] : null;
				}]
			},
			component: { 
				name: "slot-user"
			},
			span: 12,
		},
	],
	// 添加 onOpen 钩子，在表单打开时获取最新列表
	async onOpen(data?: any) {
		await Promise.all([getDeviceTypeList(), getProjectList(), getUserList()]);
		console.log('表单打开，数据:', data);
		if (data) {
			if (data.deviceTypeId) {
				console.log('设备类型ID:', data.deviceTypeId, '类型:', typeof data.deviceTypeId);
			}
			if (data.projectId) {
				console.log('项目ID:', data.projectId, '类型:', typeof data.projectId);
			}
			if (data.userId) {
				console.log('用户ID:', data.userId, '类型:', typeof data.userId);
			}
		}
	}
});

// cl-table
const Table = useTable({
	columns: [
		{ type: "selection" },
		{ label: t("名称"), prop: "name", minWidth: 120 },
		{ label: t("编号"), prop: "number", minWidth: 120 },
		{ 
			label: t("当前所处项目"), 
			prop: "projectId", 
			minWidth: 120,
			formatter: (row) => {
				try {
					let id = row.projectId;
					if (Array.isArray(id)) {
						id = id[0];
					}
					const project = options.project.find(e => e.value === String(id));
					return project ? project.label : "-";
				} catch(err) {
					console.warn("格式化projectId失败:", err);
					return "-";
				}
			}
		},
		{ 
			label: t("当前状态"), 
			prop: "status", 
			minWidth: 120,
			dict: options.status, // 使用状态选项字典
			dictColor: true, // 显示不同颜色
		},
		{ 
			label: t("设备类型"), 
			prop: "deviceTypeId",
			minWidth: 120,
			formatter: (row) => {
				try {
					let id = row.deviceTypeId;
					if (Array.isArray(id)) {
						id = id[0];
					}
					const type = options.deviceType.find(e => e.value === String(id));
					return type ? type.label : "-";
				} catch(err) {
					console.warn("格式化deviceTypeId失败:", err);
					return "-";
				}
			}
		},
		{ label: t("当前位置"), prop: "location", minWidth: 120 },
		{ 
			label: t("当前借取人"), 
			prop: "userId", 
			minWidth: 120,
			formatter: (row) => {
				try {
					let id = row.userId;
					if (Array.isArray(id)) {
						id = id[0];
					}
					const user = options.user.find(e => e.value === String(id));
					return user ? user.label : "-";
				} catch(err) {
					console.warn("格式化userId失败:", err);
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
const Search = useSearch({
	items: [
		{
			label: t("设备状态"),
			prop: "status",
			component: {
				name: "cl-select",
				props: {
					options: options.status,
					clearable: true
				}
			}
		},
		{
			label: t("名称"),
			prop: "name",
			component: {
				name: "el-input",
				props: {
					clearable: true
				}
			}
		}
	]
});

// cl-crud
const Crud = useCrud(
	{
		service: service.xsw.device,
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
