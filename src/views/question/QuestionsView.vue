<template>
  <div id="questionsView">
    <h2>浏览题目</h2>
    <a-space>
      <a-form :model="searchParams" layout="inline">
        <a-form-item field="title" label="名称" style="min-width: 240px">
          <a-input
            v-model="searchParams.title"
            placeholder="请输入题目名称..."
          />
        </a-form-item>
        <a-form-item field="tags" label="标签" style="min-width: 240px">
          <a-input-tag
            v-model="searchParams.tags"
            placeholder="请输入标签..."
          />
        </a-form-item>
        <a-form-item>
          <a-button type="primary" style="min-width: 80px" @click="doSubmit"
            >提交</a-button
          >
        </a-form-item>
      </a-form>
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
      <template #tags="{ record }">
        <!--      插槽之tags-->
        <a-space wrap>
          <a-tag
            v-for="(tag, index) of record.tags"
            :key="index"
            color="arcoblue"
            >{{ tag }}
          </a-tag>
        </a-space>
      </template>
      <template #acceptRate="{ record }">
        <!--      插槽之通过率-->
        {{
          `${
            record.submitNum ? record.acceptedNum / record.submitNum : "0"
          }% (${record.acceptedNum}/${record.submitNum})`
        }}
      </template>
      <template #createTime="{ record }">
        <!--      插槽之创建时间  -->
        {{ moment(record.createTime).format("YYYY-MM-DD") }}
      </template>
      <template #optional="{ record }">
        <!--     a-space 用于在按钮之间添加水平间距，使界面更加整洁。-->
        <a-space>
          <a-button type="primary" @click="toQeustionPage(record)"
            >开始做题
          </a-button>
        </a-space>
      </template>
    </a-table>
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import {
  Question,
  QuestionControllerService,
  QuestionQueryRequest,
} from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";
import moment from "moment";

const tableRef = ref();
const total = ref(0);
const dataList = ref([]);
const searchParams = ref<QuestionQueryRequest>({
  title: "",
  tags: [],
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

const doSubmit = () => {
  // 这里需要重置页数(不然你在第二页搜，如果搜出来的数据不到第二页，则不显示
  searchParams.value = {
    ...searchParams.value,
    current: 1,
  };
};

/**
 * 导入数据（分页条件【searchParams】变化的时候回自动执行这个！
 */
const loadData = async () => {
  const res = await QuestionControllerService.listQuestionVoByPageUsingPost(
    searchParams.value
  );

  if (res.code === 0) {
    dataList.value = res.data.records;
    total.value = res.data.total;
  } else {
    message.error("加载失败：" + res.message);
  }
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
// 列名
const columns = [
  {
    title: "题号",
    dataIndex: "id",
  },
  {
    title: "题目名称",
    dataIndex: "title",
  },
  {
    title: "标签",
    slotName: "tags",
  },
  {
    title: "通过率",
    slotName: "acceptRate",
  },
  {
    title: "创建时间",
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
#questionsView {
  max-width: 1280px;
  margin: 0 auto;
}
</style>
