<script lang="ts" setup>
import { computed, reactive, ref } from 'vue';
import { useRouter } from 'vue-router';

import { Icon } from '@packages/icon';
import { ElMessage } from 'element-plus';

import { strokeIconPaths } from './AgentWork/strokeIconPaths';

type ConfigTab = 'dataset' | 'employees';
type LoginType = '短信验证码' | '手机扫码' | '图形验证码' | '无验证';

interface DataEmployee {
  description: string;
  id: string;
  loginType: LoginType;
  loginUrl: string;
  name: string;
  skillContent: string;
  skillFileName: string;
  skillUpdated: string;
  skillVersion: string;
}

interface WaybillField {
  example: string;
  name: string;
  semantic: string;
}

interface ValidationResult {
  checkedAt: string;
  entity: Record<string, string>;
  fieldNames: string[];
  message: string;
  success: boolean;
}

const router = useRouter();
const activeTab = ref<ConfigTab>('employees');
const isCreateEmployeeModalOpen = ref(false);
const isValidationModalOpen = ref(false);
const editingEmployeeId = ref('');
const validatingEmployee = ref<DataEmployee | null>(null);
const loginTypes: LoginType[] = ['无验证', '图形验证码', '短信验证码', '手机扫码'];
const newEmployeeForm = reactive({
  description: '',
  loginType: '无验证' as LoginType,
  loginUrl: '',
  name: '',
  skillContent: '',
  skillFileName: '',
});
const validationForm = reactive({
  graphicCode: '',
  password: '',
  smsCode: '',
  username: '',
});
const validationResult = ref<ValidationResult | null>(null);
const dataEmployees = ref<DataEmployee[]>([
  {
    id: 'jinyu-cement-tms',
    name: '金隅水泥TMS',
    description: '面向金隅水泥运输业务的TMS抓取数据员工。',
    loginUrl: 'https://tms.jinyu.demo/login',
    loginType: '图形验证码',
    skillVersion: 'v1.3',
    skillUpdated: '今天 09:40',
    skillFileName: 'jinyu-waybill-mapping.skill.md',
    skillContent: `# 金隅水泥TMS 运单映射 Skill

目标：进入“运输管理 / 在途运单”页面，抓取今日在途运单明细。

页面导航：
1. 登录后进入【运输管理】。
2. 打开【运单查询】并筛选状态=在途。
3. 展开列表字段：运单号、车牌、承运商、起运地、目的地、发车时间、预计到达时间。

语义映射：
- 运单编号 -> waybill_no
- 车牌号码 -> vehicle_plate
- 承运单位 -> carrier_name
- 起运工厂 -> origin_name
- 收货仓库 -> destination_name
- 运输状态 -> order_status`,
  },
  {
    id: 'zhilian-shunda-tms',
    name: '智链顺达TMS',
    description: '负责从智链顺达调度中心抓取执行中运输任务。',
    loginUrl: 'https://tms.zhilian-shunda.demo/login',
    loginType: '短信验证码',
    skillVersion: 'v1.1',
    skillUpdated: '昨天 18:20',
    skillFileName: 'zhilian-waybill-mapping.skill.md',
    skillContent: `# 智链顺达TMS 运单映射 Skill

目标：从“调度中心 / 执行中任务”抓取执行中运单。

页面导航：
1. 使用账号和短信验证码登录。
2. 进入【调度中心】。
3. 打开【执行中任务】，按更新时间倒序抓取。

语义映射：
- 任务单号 -> waybill_no
- 司机车辆 -> vehicle_plate
- 物流商 -> carrier_name
- 装货点 -> origin_name
- 卸货点 -> destination_name
- 最新定位 -> current_location`,
  },
  {
    id: 'jinmailang-logistics',
    name: '今麦郎物流管理',
    description: '面向今麦郎发运看板和运单列表的数据接入员工。',
    loginUrl: 'https://logistics.jinmailang.demo/login',
    loginType: '无验证',
    skillVersion: 'v1.0',
    skillUpdated: '06-24 15:12',
    skillFileName: 'jinmailang-waybill-mapping.skill.md',
    skillContent: `# 今麦郎物流管理 运单映射 Skill

目标：从“发运看板 / 运单列表”抓取发运和在途数据。

页面导航：
1. 登录后进入【发运看板】。
2. 切换到【运单列表】。
3. 抓取列表和详情弹窗中的线路、货品、状态、异常标记。

语义映射：
- 发运单号 -> waybill_no
- 线路名称 -> route_name
- 货品名称 -> cargo_name
- 当前节点 -> order_status
- 异常标签 -> abnormal_type`,
  },
  {
    id: 'spreadsheet-waybill',
    name: '表格运单',
    description: '用于上传表格运单并映射为标准运单数据集。',
    loginUrl: '本地表格导入',
    loginType: '无验证',
    skillVersion: 'v1.2',
    skillUpdated: '06-23 11:08',
    skillFileName: 'spreadsheet-waybill-mapping.skill.md',
    skillContent: `# 表格运单映射 Skill

目标：将客户上传的 Excel / CSV 运单表映射为标准运单数据集。

读取规则：
1. 第一行默认为表头。
2. 自动识别运单号、车牌、司机、承运商、线路、起止点、时间字段。
3. 若存在多个候选字段，优先选择包含“运单”“车牌”“起运”“目的”“状态”的中文表头。

语义映射：
- 运单号 / 单号 / 任务号 -> waybill_no
- 车牌 / 车辆 -> vehicle_plate
- 司机 / 驾驶员 -> driver_name
- 承运商 / 物流商 -> carrier_name`,
  },
  {
    id: 'scan-login-tms',
    name: '扫码登录TMS',
    description: '用于演示手机扫码登录场景的数据员工，抓取在途运单列表。',
    loginUrl: 'https://tms.scan-login.demo/login',
    loginType: '手机扫码',
    skillVersion: 'v1.0',
    skillUpdated: '刚刚',
    skillFileName: 'scan-login-waybill-mapping.skill.md',
    skillContent: `# 扫码登录TMS 运单映射 Skill

目标：使用手机扫码登录目标 TMS，进入在途运单页面并抓取运单明细。

页面导航：
1. 打开登录页，等待二维码渲染完成。
2. 用户使用手机端扫码确认登录。
3. 登录成功后进入【在途监控 / 运单列表】。
4. 抓取第一屏运单字段并进入详情页补充轨迹和状态字段。

语义映射：
- 运单号 -> waybill_no
- 车牌 -> vehicle_plate
- 司机 -> driver_name
- 承运商 -> carrier_name
- 当前位置 -> current_location
- 运单状态 -> order_status`,
  },
]);
const selectedEmployeeId = ref(dataEmployees.value[0]!.id);

const menuItems: { desc: string; icon: string; id: ConfigTab; label: string }[] = [
  { id: 'employees', label: '数据员工配置', desc: '抓取账号、登录方式、映射 Skill', icon: strokeIconPaths.bot },
  { id: 'dataset', label: '标准数据集', desc: '运单字段、语义、数据示例', icon: strokeIconPaths.list },
];

const waybillFields: WaybillField[] = [
  { name: 'waybill_no', semantic: '运单唯一编号，用于跨系统识别同一票运输任务。', example: 'WB202606250018' },
  { name: 'source_system', semantic: '数据来源系统或导入渠道，便于追踪抓取来源。', example: '金隅水泥TMS' },
  { name: 'project_name', semantic: '归属项目或客户项目名称。', example: '华东干线在途监控' },
  { name: 'carrier_name', semantic: '承运商、物流商或实际运输服务商名称。', example: '安捷物流' },
  { name: 'vehicle_plate', semantic: '执行运输任务的车辆车牌号。', example: '沪A12345' },
  { name: 'driver_name', semantic: '当前运单绑定司机姓名。', example: '张师傅' },
  { name: 'driver_phone', semantic: '司机联系方式，用于人工复核和异常联系。', example: '138****6821' },
  { name: 'route_name', semantic: '线路名称或起止点组合后的标准线路。', example: '上海工厂 → 广州仓' },
  { name: 'origin_name', semantic: '装货地、发货工厂或起运仓名称。', example: '上海一厂' },
  { name: 'origin_address', semantic: '装货地详细地址或围栏地址。', example: '上海市嘉定区胜辛南路88号' },
  { name: 'destination_name', semantic: '卸货地、收货仓或目的地名称。', example: '广州仓' },
  { name: 'destination_address', semantic: '卸货地详细地址或目的地围栏地址。', example: '广州市黄埔区开创大道168号' },
  { name: 'cargo_name', semantic: '货品、物料或运输品类名称。', example: '袋装水泥 P.O42.5' },
  { name: 'cargo_weight', semantic: '货物重量，统一保留数值和单位。', example: '31.5 吨' },
  { name: 'order_status', semantic: '运单当前执行状态。', example: '在途' },
  { name: 'plan_depart_time', semantic: '计划发车或计划出库时间。', example: '2026-06-25 08:00' },
  { name: 'actual_depart_time', semantic: '实际发车或离开发货地时间。', example: '2026-06-25 08:23' },
  { name: 'plan_arrival_time', semantic: '计划到达目的地时间。', example: '2026-06-26 02:30' },
  { name: 'actual_arrival_time', semantic: '实际到达目的地时间，未到达时为空。', example: '-' },
  { name: 'current_location', semantic: '最近一次定位解析出的当前位置。', example: 'G60沪昆高速嘉兴段' },
  { name: 'gps_time', semantic: '最近一次有效GPS定位时间。', example: '2026-06-25 14:16:32' },
  { name: 'risk_level', semantic: '智能体归一后的风险等级。', example: '高风险' },
  { name: 'abnormal_type', semantic: '异常类型，可承接规则预警、GPS疑似造假、长时间停车等。', example: '非目的地物流园长停' },
  { name: 'raw_payload_ref', semantic: '原始抓取数据引用，用于问题追溯和重新映射。', example: 'crawl://20260625/jinyu/018' },
];

const selectedEmployee = computed(() => dataEmployees.value.find((employee) => employee.id === selectedEmployeeId.value) ?? dataEmployees.value[0]!);
const currentValidationLoginType = computed(() => validatingEmployee.value?.loginType ?? '无验证');
const isEditingEmployee = computed(() => editingEmployeeId.value.length > 0);
const employeeFormTitle = computed(() => (isEditingEmployee.value ? '编辑数据员工（TMS）' : '增加数据员工（TMS）'));
const employeeFormConfirmText = computed(() => (isEditingEmployee.value ? '保存' : '确认'));

function bumpVersion(version: string) {
  const versionNumber = Number(version.replace('v', ''));
  return Number.isFinite(versionNumber) ? `v${(versionNumber + 0.1).toFixed(1)}` : 'v1.0';
}

function loginTypeClass(loginType: LoginType) {
  if (loginType === '短信验证码') return 'border-amber-200 bg-amber-50 text-amber-700';
  if (loginType === '图形验证码') return 'border-sky-200 bg-sky-50 text-sky-700';
  if (loginType === '手机扫码') return 'border-violet-200 bg-violet-50 text-violet-700';
  return 'border-emerald-200 bg-emerald-50 text-emerald-700';
}

function showSkill(employee: DataEmployee) {
  selectedEmployeeId.value = employee.id;
}

function resetValidationForm() {
  validationForm.username = '';
  validationForm.password = '';
  validationForm.graphicCode = '';
  validationForm.smsCode = '';
  validationResult.value = null;
}

function openValidationModal(employee: DataEmployee) {
  showSkill(employee);
  validatingEmployee.value = employee;
  resetValidationForm();
  isValidationModalOpen.value = true;
}

function closeValidationModal() {
  isValidationModalOpen.value = false;
  validatingEmployee.value = null;
  resetValidationForm();
}

function sendSmsCode() {
  if (!validationForm.username.trim()) {
    ElMessage.warning('请先输入账号');
    return;
  }
  validationForm.smsCode = '246810';
  ElMessage.success('短信验证码已发送');
}

function resetNewEmployeeForm() {
  editingEmployeeId.value = '';
  newEmployeeForm.name = '';
  newEmployeeForm.description = '';
  newEmployeeForm.loginUrl = '';
  newEmployeeForm.loginType = '无验证';
  newEmployeeForm.skillContent = '';
  newEmployeeForm.skillFileName = '';
}

function openCreateEmployeeModal() {
  resetNewEmployeeForm();
  isCreateEmployeeModalOpen.value = true;
}

function openEditEmployeeModal(employee: DataEmployee) {
  showSkill(employee);
  editingEmployeeId.value = employee.id;
  newEmployeeForm.name = employee.name;
  newEmployeeForm.description = employee.description;
  newEmployeeForm.loginUrl = employee.loginUrl;
  newEmployeeForm.loginType = employee.loginType;
  newEmployeeForm.skillContent = employee.skillContent;
  newEmployeeForm.skillFileName = employee.skillFileName;
  isCreateEmployeeModalOpen.value = true;
}

function closeCreateEmployeeModal() {
  isCreateEmployeeModalOpen.value = false;
  resetNewEmployeeForm();
}

async function uploadNewEmployeeSkill(event: Event) {
  const input = event.target as HTMLInputElement;
  const file = input.files?.[0];
  if (!file) return;
  newEmployeeForm.skillContent = await file.text();
  newEmployeeForm.skillFileName = file.name;
  input.value = '';
}

function confirmCreateEmployee() {
  const name = newEmployeeForm.name.trim();
  const description = newEmployeeForm.description.trim();
  const loginUrl = newEmployeeForm.loginUrl.trim();
  if (!name) {
    ElMessage.warning('请输入数据员工名称');
    return;
  }
  if (!description) {
    ElMessage.warning('请输入数据员工描述');
    return;
  }
  if (!loginUrl) {
    ElMessage.warning('请输入接入地址');
    return;
  }
  if (!newEmployeeForm.skillFileName || !newEmployeeForm.skillContent) {
    ElMessage.warning('请上传数据获取映射 skill 文件');
    return;
  }

  if (isEditingEmployee.value) {
    const employee = dataEmployees.value.find((item) => item.id === editingEmployeeId.value);
    if (!employee) {
      ElMessage.warning('未找到需要编辑的数据员工');
      return;
    }
    const isSkillChanged = newEmployeeForm.skillFileName !== employee.skillFileName || newEmployeeForm.skillContent !== employee.skillContent;
    dataEmployees.value = dataEmployees.value.map((item) =>
      item.id === employee.id
        ? {
            ...item,
            name,
            description,
            loginUrl,
            loginType: newEmployeeForm.loginType,
            skillContent: newEmployeeForm.skillContent,
            skillFileName: newEmployeeForm.skillFileName,
            skillUpdated: isSkillChanged ? '刚刚' : item.skillUpdated,
            skillVersion: isSkillChanged ? bumpVersion(item.skillVersion) : item.skillVersion,
          }
        : item,
    );
    selectedEmployeeId.value = employee.id;
    isCreateEmployeeModalOpen.value = false;
    resetNewEmployeeForm();
    ElMessage.success('数据员工已保存');
    return;
  }

  const employee: DataEmployee = {
    id: `custom-tms-${Date.now()}`,
    name,
    description,
    loginUrl,
    loginType: newEmployeeForm.loginType,
    skillVersion: 'v1.0',
    skillUpdated: '刚刚',
    skillFileName: newEmployeeForm.skillFileName,
    skillContent: newEmployeeForm.skillContent,
  };
  dataEmployees.value = [employee, ...dataEmployees.value];
  selectedEmployeeId.value = employee.id;
  isCreateEmployeeModalOpen.value = false;
  resetNewEmployeeForm();
  ElMessage.success('数据员工已增加');
}

function buildValidationEntity(employee: DataEmployee) {
  return {
    waybill_no: `WB${new Date().toISOString().slice(0, 10).replaceAll('-', '')}001`,
    source_system: employee.name,
    carrier_name: '安捷物流',
    vehicle_plate: '沪A12345',
    driver_name: '张师傅',
    route_name: '上海工厂 → 广州仓',
    origin_name: '上海一厂',
    destination_name: '广州仓',
    order_status: '在途',
    current_location: 'G60沪昆高速嘉兴段',
    gps_time: '2026-06-25 14:16:32',
    risk_level: '低风险',
  };
}

function validateEmployee() {
  if (!validatingEmployee.value) return;
  if (currentValidationLoginType.value === '手机扫码') {
    const entity = buildValidationEntity(validatingEmployee.value);
    validationResult.value = {
      checkedAt: '刚刚',
      entity,
      fieldNames: Object.keys(entity),
      message: `${validatingEmployee.value.name} 扫码登录成功，已通过数据映射 skill 获取 1 条运单样例。`,
      success: true,
    };
    ElMessage.success('验证完成');
    return;
  }
  if (!validationForm.username.trim()) {
    ElMessage.warning('请输入账号');
    return;
  }
  if (!validationForm.password.trim()) {
    ElMessage.warning('请输入密码');
    return;
  }
  if (currentValidationLoginType.value === '图形验证码' && !validationForm.graphicCode.trim()) {
    ElMessage.warning('请输入图形验证码');
    return;
  }
  if (currentValidationLoginType.value === '短信验证码' && !validationForm.smsCode.trim()) {
    ElMessage.warning('请输入短信验证码');
    return;
  }
  const entity = buildValidationEntity(validatingEmployee.value);
  validationResult.value = {
    checkedAt: '刚刚',
    entity,
    fieldNames: Object.keys(entity),
    message: `${validatingEmployee.value.name} 登录成功，已通过数据映射 skill 获取 1 条运单样例。`,
    success: true,
  };
  ElMessage.success('验证完成');
}

async function uploadSkill(employee: DataEmployee, event: Event) {
  const input = event.target as HTMLInputElement;
  const file = input.files?.[0];
  if (!file) return;
  const content = await file.text();
  dataEmployees.value = dataEmployees.value.map((item) =>
    item.id === employee.id
      ? {
          ...item,
          skillContent: content || item.skillContent,
          skillFileName: file.name,
          skillUpdated: '刚刚',
          skillVersion: bumpVersion(item.skillVersion),
        }
      : item,
  );
  selectedEmployeeId.value = employee.id;
  input.value = '';
  ElMessage.success(`${employee.name} 的数据映射 skill 已更新`);
}
</script>

<template>
  <div class="flex h-screen flex-col overflow-hidden bg-[#f7f7f5] text-slate-900">
    <header class="flex h-14 shrink-0 items-center justify-between border-b border-[#deded9] bg-white px-5">
      <div class="flex min-w-0 items-center gap-3">
        <div class="flex h-8 w-8 shrink-0 items-center justify-center rounded-md border border-[#deded9] bg-[#f7f7f5] text-slate-700">
          <Icon :svg="strokeIconPaths.bot" :size="18" />
        </div>
        <div class="min-w-0">
          <h1 class="truncate text-sm font-semibold leading-5 text-slate-950">智能体运营配置</h1>
          <p class="truncate text-xs leading-4 text-slate-500">Playwright 抓取指引、数据映射 Skill 与标准数据集管理</p>
        </div>
      </div>
      <button type="button" class="rounded-md border border-[#deded9] px-3 py-1.5 text-xs text-slate-600 hover:bg-[#f7f7f5]" @click="router.push('/index')">
        返回工作台
      </button>
    </header>

    <main class="grid min-h-0 flex-1 grid-cols-[230px_minmax(0,1fr)] gap-3 p-4">
      <aside class="flex min-h-0 flex-col overflow-hidden rounded-md border border-[#deded9] bg-white">
        <div class="border-b border-[#e2e2dc] px-4 py-3">
          <h2 class="text-sm font-semibold leading-5 text-slate-950">运营菜单</h2>
          <p class="mt-1 text-xs leading-5 text-slate-500">配置数据员工抓取和标准数据集。</p>
        </div>
        <nav class="flex-1 space-y-1 p-3">
          <button
            v-for="item in menuItems"
            :key="item.id"
            type="button"
            class="flex w-full items-start gap-2.5 rounded-md px-3 py-2.5 text-left transition"
            :class="activeTab === item.id ? 'bg-slate-900 text-white' : 'text-slate-600 hover:bg-[#f7f7f5] hover:text-slate-950'"
            @click="activeTab = item.id"
          >
            <span
              class="mt-0.5 flex h-7 w-7 shrink-0 items-center justify-center rounded-md"
              :class="activeTab === item.id ? 'bg-white/12 text-white' : 'bg-[#f2f2ef] text-slate-600'"
            >
              <Icon :svg="item.icon" :size="15" />
            </span>
            <span class="min-w-0">
              <span class="block text-sm font-medium leading-5">{{ item.label }}</span>
              <span class="mt-0.5 block text-xs leading-4" :class="activeTab === item.id ? 'text-white/70' : 'text-slate-400'">
                {{ item.desc }}
              </span>
            </span>
          </button>
        </nav>
        <div class="border-t border-[#e2e2dc] px-4 py-3 text-xs leading-5 text-slate-500">
          当前仅开放运单标准数据集，后续可扩展车辆、司机、费用等表。
        </div>
      </aside>

      <div class="min-h-0 overflow-hidden">
        <section v-if="activeTab === 'employees'" class="grid h-full min-h-0 grid-cols-[minmax(0,1.45fr)_minmax(360px,0.75fr)] gap-3">
        <div class="flex min-h-0 flex-col overflow-hidden rounded-md border border-[#deded9] bg-white">
          <div class="flex h-11 shrink-0 items-center justify-between border-b border-[#e2e2dc] px-4">
            <h2 class="text-sm font-semibold leading-5 text-slate-950">数据员工列表</h2>
            <div class="flex items-center gap-2">
              <span class="text-xs text-slate-500">{{ dataEmployees.length }} 个数据员工</span>
              <button type="button" class="rounded-md bg-slate-900 px-3 py-1.5 text-xs font-medium text-white hover:bg-slate-800" @click="openCreateEmployeeModal">
                增加数据员工（TMS）
              </button>
            </div>
          </div>
          <div class="min-h-0 flex-1 overflow-auto">
            <table class="w-full border-collapse text-left text-sm">
              <thead class="sticky top-0 z-10 bg-[#f7f7f5]">
                <tr class="text-xs font-semibold text-slate-500">
                  <th class="px-4 py-3">数据员工名称</th>
                  <th class="px-4 py-3">登录地址</th>
                  <th class="w-[130px] px-4 py-3">登录方式</th>
                  <th class="w-[230px] px-4 py-3">操作</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-[#ededea]">
                <tr
                  v-for="employee in dataEmployees"
                  :key="employee.id"
                  class="cursor-pointer hover:bg-[#f7f7f5]"
                  :class="selectedEmployee.id === employee.id ? 'bg-[#f7f7f5]' : 'bg-white'"
                  @click="showSkill(employee)"
                >
                  <td class="px-4 py-4 align-middle">
                    <div class="font-medium text-slate-950">{{ employee.name }}</div>
                    <div class="mt-1 max-w-[300px] truncate text-xs text-slate-500">{{ employee.description || '暂无描述' }}</div>
                    <div class="mt-1 text-xs text-slate-400">{{ employee.skillFileName }} · {{ employee.skillVersion }}</div>
                  </td>
                  <td class="px-4 py-4 align-middle">
                    <div class="max-w-[340px] truncate font-mono text-xs text-slate-600">{{ employee.loginUrl }}</div>
                    <div class="mt-1 text-xs text-slate-400">更新：{{ employee.skillUpdated }}</div>
                  </td>
                  <td class="px-4 py-4 align-middle">
                    <span class="inline-flex rounded-md border px-2 py-0.5 text-xs font-medium" :class="loginTypeClass(employee.loginType)">
                      {{ employee.loginType }}
                    </span>
                  </td>
                  <td class="px-4 py-4 align-middle">
                    <div class="flex flex-wrap items-center gap-2">
                      <button type="button" class="rounded-md border border-[#deded9] px-2 py-1 text-xs hover:bg-white" @click.stop="openEditEmployeeModal(employee)">
                        编辑
                      </button>
                      <button type="button" class="rounded-md border border-[#deded9] px-2 py-1 text-xs hover:bg-white" @click.stop="openValidationModal(employee)">
                        验证数据员工
                      </button>
                      <label class="cursor-pointer rounded-md bg-slate-900 px-2 py-1 text-xs font-medium text-white hover:bg-slate-800" @click.stop>
                        上传更新skill
                        <input class="hidden" type="file" accept=".md,.txt,.yaml,.yml" @change.stop="uploadSkill(employee, $event)" />
                      </label>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <aside class="flex min-h-0 flex-col overflow-hidden rounded-md border border-[#deded9] bg-white">
          <div class="flex h-11 shrink-0 items-center justify-between border-b border-[#e2e2dc] px-4">
            <h2 class="text-sm font-semibold leading-5 text-slate-950">数据映射 Skill</h2>
            <span class="rounded-md border border-[#deded9] bg-[#f7f7f5] px-2 py-0.5 text-xs text-slate-500">{{ selectedEmployee.skillVersion }}</span>
          </div>
          <div class="space-y-3 border-b border-[#e2e2dc] px-4 py-3 text-xs text-slate-500">
            <div class="flex items-center justify-between gap-3">
              <span>当前数据员工</span>
              <span class="font-medium text-slate-800">{{ selectedEmployee.name }}</span>
            </div>
            <div class="flex items-center justify-between gap-3">
              <span>Skill 文件</span>
              <span class="truncate font-mono text-slate-700">{{ selectedEmployee.skillFileName }}</span>
            </div>
          </div>
          <pre class="min-h-0 flex-1 overflow-auto whitespace-pre-wrap bg-[#fbfbfa] p-4 text-xs leading-5 text-slate-700">{{ selectedEmployee.skillContent }}</pre>
        </aside>
      </section>

        <section v-else class="flex h-full min-h-0 flex-col overflow-hidden rounded-md border border-[#deded9] bg-white">
        <div class="shrink-0 border-b border-[#e2e2dc] px-4 py-3">
          <div class="flex items-center justify-between gap-3">
            <h2 class="text-sm font-semibold leading-5 text-slate-950">运单标准数据集</h2>
            <span class="text-xs text-slate-500">{{ waybillFields.length }} 个字段</span>
          </div>
          <p class="mt-1 text-xs leading-5 text-slate-500">
            数据员工抓取各 TMS 页面后，先按 Skill 将原始字段映射到该标准数据集。标准字段用于后续在途监控、异常识别、轨迹核验和报表输出。
          </p>
        </div>
        <div class="min-h-0 flex-1 overflow-auto">
          <table class="w-full border-collapse text-left text-sm">
            <thead class="sticky top-0 z-10 bg-[#f7f7f5]">
              <tr class="text-xs font-semibold text-slate-500">
                <th class="w-[210px] px-4 py-3">字段名称</th>
                <th class="px-4 py-3">语义</th>
                <th class="w-[260px] px-4 py-3">数据示例</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-[#ededea]">
              <tr v-for="field in waybillFields" :key="field.name" class="hover:bg-[#f7f7f5]">
                <td class="px-4 py-3 align-top font-mono text-xs font-medium text-slate-900">{{ field.name }}</td>
                <td class="px-4 py-3 align-top text-sm leading-5 text-slate-600">{{ field.semantic }}</td>
                <td class="px-4 py-3 align-top font-mono text-xs text-slate-600">{{ field.example }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        </section>
      </div>
    </main>

    <div v-if="isValidationModalOpen && validatingEmployee" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-950/40 p-6">
      <div class="flex max-h-[88vh] w-full max-w-[760px] flex-col overflow-hidden rounded-md border border-[#deded9] bg-white shadow-xl">
        <div class="flex h-12 shrink-0 items-center justify-between border-b border-[#e2e2dc] px-4">
          <div class="flex min-w-0 items-center gap-2.5">
            <div class="flex h-7 w-7 shrink-0 items-center justify-center rounded-md bg-[#f2f2ef] text-slate-700">
              <Icon :svg="strokeIconPaths.shield" :size="16" />
            </div>
            <div class="min-w-0">
              <h2 class="truncate text-sm font-semibold leading-5 text-slate-950">验证数据员工</h2>
              <p class="truncate text-xs leading-4 text-slate-500">{{ validatingEmployee.name }} · {{ validatingEmployee.loginType }}</p>
            </div>
          </div>
          <button type="button" class="rounded-md p-1 text-slate-400 hover:bg-[#f7f7f5] hover:text-slate-700" @click="closeValidationModal">
            <Icon :svg="strokeIconPaths.x" :size="16" />
          </button>
        </div>

        <div class="min-h-0 flex-1 overflow-auto">
          <div class="grid gap-4 p-4 lg:grid-cols-[minmax(0,0.9fr)_minmax(0,1.1fr)]">
            <div class="space-y-3">
              <div class="rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 py-2 text-xs leading-5 text-slate-500">
                接入地址：<span class="font-mono text-slate-700">{{ validatingEmployee.loginUrl }}</span>
              </div>

              <template v-if="currentValidationLoginType !== '手机扫码'">
                <label class="block">
                  <span class="mb-1.5 block text-xs font-medium text-slate-600">账号</span>
                  <input
                    v-model.trim="validationForm.username"
                    class="h-10 w-full rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 text-sm outline-none focus:border-slate-400"
                    placeholder="请输入目标系统账号"
                  />
                </label>

                <label class="block">
                  <span class="mb-1.5 block text-xs font-medium text-slate-600">密码</span>
                  <input
                    v-model.trim="validationForm.password"
                    type="password"
                    class="h-10 w-full rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 text-sm outline-none focus:border-slate-400"
                    placeholder="请输入目标系统密码"
                  />
                </label>
              </template>

              <div v-if="currentValidationLoginType === '无验证'" class="rounded-md border border-emerald-200 bg-emerald-50 px-3 py-2 text-xs leading-5 text-emerald-700">
                当前登录方式为无验证，填写账号和密码后即可验证。
              </div>

              <label v-else-if="currentValidationLoginType === '图形验证码'" class="block">
                <span class="mb-1.5 block text-xs font-medium text-slate-600">图形验证码</span>
                <div class="flex gap-2">
                  <div class="flex h-10 w-24 shrink-0 items-center justify-center rounded-md border border-[#deded9] bg-[#f2f2ef] font-mono text-sm font-semibold tracking-[0.22em] text-slate-700">
                    A7K9
                  </div>
                  <input
                    v-model.trim="validationForm.graphicCode"
                    class="h-10 min-w-0 flex-1 rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 text-sm outline-none focus:border-slate-400"
                    placeholder="请输入图形验证码"
                  />
                </div>
              </label>

              <label v-else-if="currentValidationLoginType === '短信验证码'" class="block">
                <span class="mb-1.5 block text-xs font-medium text-slate-600">短信验证码</span>
                <div class="flex gap-2">
                  <input
                    v-model.trim="validationForm.smsCode"
                    class="h-10 min-w-0 flex-1 rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 text-sm outline-none focus:border-slate-400"
                    placeholder="请输入短信验证码"
                  />
                  <button type="button" class="h-10 shrink-0 rounded-md border border-[#deded9] px-3 text-xs text-slate-600 hover:bg-[#f7f7f5]" @click="sendSmsCode">
                    获取验证码
                  </button>
                </div>
              </label>

              <div v-else>
                <span class="mb-1.5 block text-xs font-medium text-slate-600">手机扫码</span>
                <div class="flex flex-col items-center justify-center gap-3 rounded-md border border-[#deded9] bg-[#fbfbfa] p-4 text-center">
                  <div class="grid h-28 w-28 shrink-0 grid-cols-5 grid-rows-5 gap-1 rounded bg-white p-1.5 shadow-sm">
                    <span v-for="index in 25" :key="index" class="rounded-sm" :class="[1, 2, 4, 6, 8, 12, 14, 16, 18, 20, 22, 24, 25].includes(index) ? 'bg-slate-900' : 'bg-slate-200'" />
                  </div>
                  <div class="text-xs leading-5 text-slate-500">请使用目标系统移动端扫码确认，确认后点击验证。</div>
                </div>
              </div>

              <button type="button" class="w-full rounded-md bg-slate-900 px-3 py-2 text-sm font-medium text-white hover:bg-slate-800" @click="validateEmployee">
                验证
              </button>
            </div>

            <div class="flex min-h-[360px] flex-col overflow-hidden rounded-md border border-[#deded9] bg-[#fbfbfa]">
              <div class="flex h-10 shrink-0 items-center justify-between border-b border-[#e2e2dc] px-3">
                <h3 class="text-sm font-semibold leading-5 text-slate-950">验证结果</h3>
                <span v-if="validationResult" class="text-xs text-slate-500">{{ validationResult.checkedAt }}</span>
              </div>

              <div v-if="validationResult" class="min-h-0 flex-1 overflow-auto p-3">
                <div
                  class="mb-3 rounded-md border px-3 py-2 text-xs leading-5"
                  :class="validationResult.success ? 'border-emerald-200 bg-emerald-50 text-emerald-700' : 'border-red-200 bg-red-50 text-red-700'"
                >
                  登录是否成功：{{ validationResult.success ? '成功' : '失败' }}。{{ validationResult.message }}
                </div>

                <div class="mb-3">
                  <div class="mb-1.5 text-xs font-medium text-slate-600">获取到的运单字段名称</div>
                  <div class="flex flex-wrap gap-1.5">
                    <span v-for="fieldName in validationResult.fieldNames" :key="fieldName" class="rounded-md bg-white px-2 py-1 font-mono text-xs text-slate-600 shadow-sm">
                      {{ fieldName }}
                    </span>
                  </div>
                </div>

                <div>
                  <div class="mb-1.5 text-xs font-medium text-slate-600">数据实体结果（1条）</div>
                  <div class="overflow-hidden rounded-md border border-[#deded9] bg-white">
                    <div v-for="(value, key) in validationResult.entity" :key="key" class="grid grid-cols-[150px_minmax(0,1fr)] border-b border-[#ededea] last:border-b-0">
                      <div class="bg-[#f7f7f5] px-2 py-2 font-mono text-xs text-slate-500">{{ key }}</div>
                      <div class="min-w-0 px-2 py-2 text-xs text-slate-700">{{ value }}</div>
                    </div>
                  </div>
                </div>
              </div>

              <div v-else class="flex flex-1 items-center justify-center px-6 text-center text-xs leading-5 text-slate-400">
                {{
                  currentValidationLoginType === '手机扫码'
                    ? '使用目标系统移动端扫码确认后点击验证，这里会显示登录状态、字段名称和一条运单实体结果。'
                    : '输入账号、密码和对应验证码后点击验证，这里会显示登录状态、字段名称和一条运单实体结果。'
                }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="isCreateEmployeeModalOpen" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-950/40 p-6">
      <div class="w-full max-w-[520px] overflow-hidden rounded-md border border-[#deded9] bg-white shadow-xl">
        <div class="flex h-12 items-center justify-between border-b border-[#e2e2dc] px-4">
          <div class="flex items-center gap-2.5">
            <div class="flex h-7 w-7 items-center justify-center rounded-md bg-[#f2f2ef] text-slate-700">
              <Icon :svg="strokeIconPaths.bot" :size="16" />
            </div>
            <h2 class="text-sm font-semibold leading-5 text-slate-950">{{ employeeFormTitle }}</h2>
          </div>
          <button type="button" class="rounded-md p-1 text-slate-400 hover:bg-[#f7f7f5] hover:text-slate-700" @click="closeCreateEmployeeModal">
            <Icon :svg="strokeIconPaths.x" :size="16" />
          </button>
        </div>

        <div class="space-y-3 px-4 py-4">
          <label class="block">
            <span class="mb-1.5 block text-xs font-medium text-slate-600">数据员工名称</span>
            <input
              v-model.trim="newEmployeeForm.name"
              class="h-10 w-full rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 text-sm outline-none focus:border-slate-400"
              placeholder="例如：某客户TMS"
            />
          </label>

          <label class="block">
            <span class="mb-1.5 block text-xs font-medium text-slate-600">描述</span>
            <textarea
              v-model.trim="newEmployeeForm.description"
              class="min-h-[72px] w-full resize-none rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 py-2 text-sm outline-none focus:border-slate-400"
              placeholder="请输入该数据员工负责的目标系统、抓取范围或使用场景"
            />
          </label>

          <label class="block">
            <span class="mb-1.5 block text-xs font-medium text-slate-600">接入地址</span>
            <input
              v-model.trim="newEmployeeForm.loginUrl"
              class="h-10 w-full rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 text-sm outline-none focus:border-slate-400"
              placeholder="请输入 TMS 登录或接入地址"
            />
          </label>

          <label class="block">
            <span class="mb-1.5 block text-xs font-medium text-slate-600">登录方式</span>
            <select
              v-model="newEmployeeForm.loginType"
              class="h-10 w-full rounded-md border border-[#deded9] bg-[#fbfbfa] px-3 text-sm outline-none focus:border-slate-400"
            >
              <option v-for="type in loginTypes" :key="type" :value="type">{{ type }}</option>
            </select>
          </label>

          <div>
            <span class="mb-1.5 block text-xs font-medium text-slate-600">数据获取映射 skill 上传</span>
            <label
              class="flex min-h-[76px] cursor-pointer flex-col items-center justify-center rounded-md border border-dashed border-[#cfcfca] bg-[#fbfbfa] px-3 py-3 text-center hover:bg-[#f7f7f5]"
            >
              <Icon :svg="strokeIconPaths.file" :size="18" svg-class="mb-1 text-slate-500" />
              <span class="text-sm font-medium text-slate-700">
                {{ newEmployeeForm.skillFileName || '选择 skill 文件' }}
              </span>
              <span class="mt-1 text-xs text-slate-400">支持 .md / .txt / .yaml / .yml</span>
              <input class="hidden" type="file" accept=".md,.txt,.yaml,.yml" @change="uploadNewEmployeeSkill" />
            </label>
          </div>
        </div>

        <div class="flex items-center justify-end gap-2 border-t border-[#e2e2dc] px-4 py-3">
          <button type="button" class="rounded-md border border-[#deded9] px-3 py-1.5 text-sm text-slate-600 hover:bg-[#f7f7f5]" @click="closeCreateEmployeeModal">
            取消
          </button>
          <button type="button" class="rounded-md bg-slate-900 px-3 py-1.5 text-sm font-medium text-white hover:bg-slate-800" @click="confirmCreateEmployee">
            {{ employeeFormConfirmText }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>
