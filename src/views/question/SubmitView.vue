<template>
  <div id="submitView">
    <h2>题目提交列表</h2>
    <a-space>
      <a-form :model="searchParams" layout="inline">
        <a-form-item field="title" label="名称" style="min-width: 240px">
          <a-input
            v-model="searchParams.questionId"
            placeholder="请输入题目序号..."
          />
        </a-form-item>
        <a-form-item field="language" label="语言" style="min-width: 210px">
          <!--          <a-input-tag-->
          <!--            v-model="searchParams.language"-->
          <!--            placeholder="请选择编程语言..."-->
          <!--          />-->
          <a-select
            v-model="searchParams.language"
            :style="{ width: '240px' }"
            placeholder="请选择编程语言..."
          >
            <a-option>java</a-option>
            <a-option>cpp</a-option>
            <a-option>go</a-option>
            <a-option>html</a-option>
          </a-select>
        </a-form-item>
        <a-form-item field="status" label="判题状态" style="min-width: 210px">
          <!--          <a-input-tag-->
          <!--            v-model="searchParams.status"-->
          <!--            placeholder="请选择判题状态..."-->
          <!--          />-->
          <a-select
            v-model="searchParams.status"
            :style="{ width: '240px' }"
            placeholder="请选择判题状态..."
          >
            <a-option :value="0">等待中</a-option>
            <a-option :value="1">判题中</a-option>
            <a-option :value="2">成功</a-option>
            <a-option :value="3">失败</a-option>
          </a-select>
        </a-form-item>
      </a-form>
      <a-button type="primary" style="min-width: 80px" @click="doSearch">
        查询
      </a-button>
    </a-space>
    <a-table
      :ref="tableRef"
      :columns="columns"
      :data="dataList"
      :pagination="{
        showTotal: true,
        pageSize: searchParams.pageSize,
        current: searchParams.current,
        total,
      }"
      @page-change="onPageChange"
    >
      <template #status="{ record }">
        <!--      插槽之language-->
        <span v-if="record.status === 0">待判题</span>
        <span v-else-if="record.status === 1">判题中</span>
        <span v-else-if="record.status === 2">成功</span>
        <span v-else-if="record.status === 3">失败</span>
        <span v-else>未知状态</span>
      </template>
      <template #judgeInfo="{ record }">
        <!--      插槽之judgeInfo-->
        {{ record.judgeInfo }}
      </template>
      <template #language="{ record }">
        <!--      插槽之language-->
        {{ record.language }}
      </template>
      <template #name="{ record }">
        <!--      插槽之用户名  -->
        {{ record.userVO?.userName ?? "未知用户" }}
      </template>
      <template #createTime="{ record }">
        <!--      插槽之创建时间  -->
        {{ moment(record.createTime).format("YYYY-MM-DD") }}
      </template>
    </a-table>
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import {
  Question,
  QuestionControllerService,
  QuestionSubmitQueryRequest,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import moment from "moment";

const tableRef = ref();
const total = ref(0);
const dataList = ref([]);
const searchParams = ref<QuestionSubmitQueryRequest>({
  questionId: undefined,
  language: undefined,
  status: undefined,
  pageSize: 10,
  current: 1,
});

// 分页函数——》
const onPageChange = (page: number) => {
  searchParams.value = {
    // 使用展开运算符...复制searchParams.value的所有现有属性到一个新的对象中，然后添加或更新current属性为新的页码page。这样做的目的是保留searchParams中其他可能存在的属性不变，仅更新页码。
    ...searchParams.value,
    current: page,
  };
};

/**
 * 导入数据（分页条件【searchParams】变化的时候回自动执行这个！
 */
const loadData = async () => {
  const res =
    // await QuestionSubmitControllerService.listQuestionSubmitByPageUsingPost(
    await QuestionControllerService.listQuestionSubmitByPageUsingPost({
      ...searchParams.value,
      sortField: "createTime",
      sortOrder: "DESC",
    });

  if (res.code === 0) {
    dataList.value = res.data.records;
    total.value = res.data.total;
  } else {
    message.error("加载失败：" + res.message);
  }
};

/**
 * 查询操作
 */
const doSearch = async () => {
  console.log(searchParams);
  // 这里需要重置页数(不然你在第二页搜，如果搜出来的数据不到第二页，则不显示
  searchParams.value = {
    ...searchParams.value,
    current: 1,
  };
};

/**
 * 页面加载时请求数据
 */
onMounted(() => {
  loadData();
});

/**
 * 监听传入的函数中的任何变量，任何变量的改变都会重新触发这个函数，重新加载
 */
watchEffect(() => {
  loadData();
});
// 列名 ：题号、题目名称、语言、判题状态、提交用户名称、提交时间
const columns = [
  {
    title: "题号",
    dataIndex: "questionId",
  },
  {
    title: "编程语言",
    slotName: "language",
  },
  {
    title: "判题信息",
    slotName: "judgeInfo",
  },
  {
    title: "判题状态",
    slotName: "status",
  },
  {
    title: "提交用户id",
    dataIndex: "userId",
  },
  {
    title: "提交时间",
    slotName: "createTime",
  },
  {
    slotName: "optional",
  },
];

/**
 * 跳转到做题页面
 * @param question
 */
const toQeustionPage = (question: Question) => {
  router.push({
    path: `/view/questions/${question.id}`,
  });
};

const router = useRouter();
</script>

<style scoped>
#submitView {
  max-width: 1280px;
  margin: 0 auto;
}
</style>
