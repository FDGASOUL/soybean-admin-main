<template>
  <n-grid :item-responsive="true" :x-gap="16" :y-gap="16">
    <n-grid-item span="0:24 640:24 1024:16">
      <n-space :vertical="true" :size="16">
        <n-card title="项目主要技术栈" :bordered="false" size="small" class="shadow-sm rounded-16px">
          <n-grid :item-responsive="true" responsive="screen" cols="m:2 l:3" :x-gap="8" :y-gap="8">
            <n-grid-item v-for="item in technology" :key="item.id">
              <technology-card v-bind="item" />
            </n-grid-item>
          </n-grid>
        </n-card>
      </n-space>
    </n-grid-item>
    <n-grid-item span="0:24 640:24 1024:8">
      <n-space :vertical="true" :size="16">
        <n-card title="快捷操作" :bordered="false" size="small" class="shadow-sm rounded-16px">
          <n-grid :item-responsive="true" responsive="screen" cols="m:2 l:3" :x-gap="8" :y-gap="8">
            <n-grid-item v-for="item in shortcuts" :key="item.id">
              <shortcuts-card v-bind="item" />
            </n-grid-item>
          </n-grid>
        </n-card>
      </n-space>
    </n-grid-item>
    <n-grid-item span="0:24 640:24 1024:8">
      <n-card title="上传文件" :bordered="false" size="small" class="shadow-sm rounded-16px">
        <div style="margin-left: 150px">
          <n-upload accept=".csv" :custom-request="customRequest" :max="1" ref="uploadRef">
            <n-button>上传 .csv 文件</n-button>
          </n-upload>
        </div>
        <div>
          <n-input v-model:value="model.dataname" placeholder="请输入数据集名称" />
        </div>
        <div style="margin-left: 180px">
          <n-button type="info" @click="handleClick">确定</n-button>
        </div>
      </n-card>
    </n-grid-item>
    <n-grid-item span="0:24 640:24 1024:8">
      <n-card title="下载/删除文件" :bordered="false" size="small" class="shadow-sm rounded-16px">
        <div style="margin-left: 150px">
          <n-button type="info" @click="downlandData()"> 下载数据 </n-button>
        </div>
        <div style="margin-left: 150px; margin-top: 40px">
          <n-button type="error" @click="deleteData()"> 删除数据 </n-button>
        </div>
      </n-card>
    </n-grid-item>
    <n-grid-item span="0:24 640:24 1024:8">
      <n-card title="请选择要下载/删除的数据集" :bordered="false" size="small" class="shadow-sm rounded-16px">
        <n-select
          v-model:value="valueSelect"
          :options="options_edit"
          :consistent-menu-width="false"
          @update:value="handleUpdateValue"
        />
      </n-card>
    </n-grid-item>
  </n-grid>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import { onMounted } from 'vue';
import type { UploadInst } from 'naive-ui';
import { DeleteData, Uplandsome, fetchDataName } from '@/service';
import { useLoading } from '@/hooks';
import { localStg } from '~/src/utils';
import { ShortcutsCard, TechnologyCard } from './components';
defineOptions({ name: 'DashboardWorkbenchMain' });
const { startLoading, endLoading } = useLoading(false);
const valueSelect = ref('');
const options_edit = ref([]);
interface Technology {
  id: number;
  name: string;
  description: string;
  author: string;
  site: string;
  icon: string;
  iconColor?: string;
}

const model = reactive({
  dataname: ''
});
let formData = new FormData();
const formRef = ref<HTMLElement & FormInst>();
const uploadRef = ref<UploadInst | null>(null);
const show = ref(false);

const customRequest = async ({ file }) => {
  if (file.file.size > 1024 * 1024 * 20) {
    window.$message?.error('文件大小不能超过20M');
    uploadRef.value?.clear();
  } else if (options_edit.value.length >= 10) {
    window.$message?.error('数据集数量不能超过10个');
    uploadRef.value?.clear();
  } else {
    formData = new FormData();
    formData.append('data_file', file.file);
  }
};

async function handleClick() {
  await formRef.value?.validate();
  await uploadRef.value?.submit();
  if (model.dataname === '') {
    window.$message?.error('请输入数据集名称');
  } else {
    formData.append('data_name', model.dataname);
    model.dataname = '';
    if (formData.get('data_file') !== null) {
      show.value = true;
      await Uplandsome(formData);
      uploadRef.value?.clear();
      window.$message?.info(`上传成功`);
      show.value = false;
      getDataName();
      valueSelect.value = '';
    } else {
      window.$message?.error('请先上传文件');
    }
  }
}

const downlandData = () => {
  const xhr = new XMLHttpRequest();
  const name = valueSelect.value;
  if (name === '') {
    window.$message?.error('请在右侧指定要下载的数据集！');
  } else {
    xhr.open('POST', `http://localhost:5000/downland?name=${name}`, true);
    xhr.responseType = 'blob';
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.setRequestHeader('Authorization', `Bearer ${localStg.get('token')}`);
    xhr.onload = function () {
      if (this.status === 200) {
        const blob = this.response;
        const reader = new FileReader();
        reader.readAsDataURL(blob);
        reader.onload = function (e) {
          const a = document.createElement('a');
          a.download = 'data.csv';
          if (typeof e.target.result === 'string') {
            a.href = e.target.result;
          }
          document.body.appendChild(a);
          a.click();
          document.body.removeChild(a);
        };
      }
    };
    xhr.send(JSON.stringify({ data_name: 'left_test' }));
  }
};

async function deleteData() {
  const name = valueSelect.value;
  if (name === '') {
    window.$message?.error('请在右侧指定要删除的数据集！');
  } else {
    await DeleteData(name);
    window.$message?.info(`删除成功`);
    getDataName();
    valueSelect.value = '';
  }
}

async function getDataName() {
  startLoading();
  const { data } = await fetchDataName();
  options_edit.value = data.map(item => {
    return {
      label: item,
      value: item
    };
  });
}

async function handleUpdateValue() {
  startLoading();
  endLoading();
}

const technology: Technology[] = [
  {
    id: 0,
    name: 'Vue',
    description: '一套用于构建用户界面的渐进式框架',
    author: '尤雨溪 - Evan You',
    site: 'https://v3.cn.vuejs.org/',
    icon: 'logos:vue'
  },
  {
    id: 1,
    name: 'TypeScript',
    description: 'JavaScript类型的超集，它可以编译成纯JavaScript',
    author: '微软 - Microsoft',
    site: 'https://www.typescriptlang.org/',
    icon: 'logos:typescript-icon'
  },
  {
    id: 2,
    name: 'Vite',
    description: '下一代前端开发与构建工具',
    author: '尤雨溪 - Evan You',
    site: 'https://vitejs.cn/',
    icon: 'logos:vitejs'
  },
  {
    id: 3,
    name: 'NaiveUI',
    description: '一个 Vue 3 组件库',
    author: '图森未来 - TuSimple',
    site: 'https://www.naiveui.com/zh-CN/os-theme',
    icon: 'logos:naiveui'
  },
  {
    id: 4,
    name: 'UnoCSS',
    description: '下一代实用优先的CSS框架',
    author: 'Anthony Fu',
    site: 'https://uno.antfu.me/?s=',
    icon: 'logos:unocss'
  },
  {
    id: 5,
    name: 'Echarts',
    description: '一个基于 JavaScript 的开源可视化图表库',
    author: '百度',
    site: 'https://echarts.apache.org/zh/index.html',
    icon: 'system-uicons:globe'
  }
];

interface Shortcuts {
  id: number;
  label: string;
  icon: string;
  iconColor: string;
}

const shortcuts: Shortcuts[] = [
  { id: 0, label: '数据分布', icon: 'mdi:chart-areaspline', iconColor: '#f56c6c' },
  { id: 1, label: '关联关系', icon: 'mdi:family-tree', iconColor: '#8aca6b' },
  { id: 2, label: '缺失情况', icon: 'mdi:desktop-mac-dashboard', iconColor: '#409eff' },
  { id: 3, label: '贝叶斯网络结构', icon: 'fluent:app-store-24-filled', iconColor: '#19a2f1' },
  { id: 4, label: '数据管理', icon: 'mdi:table-large', iconColor: '#fab251' },
  { id: 5, label: '信息管理', icon: 'ic:outline-settings', iconColor: '#7238d1' }
];
function init() {
  getDataName();
}

// 初始化
onMounted(() => {
  init();
});
</script>
<style scoped></style>
