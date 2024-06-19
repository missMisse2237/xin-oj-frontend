<template>
  <div id="manageQuestionView">
    <!--    <h2>管理题目</h2>-->
  </div>
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
    <template #optional="{ record }">
      <!--     a-space 用于在按钮之间添加水平间距，使界面更加整洁。-->
      <a-space>
        <a-button type="primary" @click="doUpdate(record)">修改</a-button>
        <!--        注意danger在arco中是status!-->
        <a-button status="danger" @click="doDelete(record)">删除</a-button>
      </a-space>
    </template>
  </a-table>
</template>
<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import { Question, QuestionControllerService } from "../../../generated";
import message from "@arco-design/web-vue/es/message";
import { useRouter } from "vue-router";

const tableRef = ref();
const total = ref(0);
const dataList = ref([]);
const searchParams = ref({
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

const loadData = async () => {
  const res = await QuestionControllerService.listQuestionByPageUsingPost(
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
    title: "id",
    dataIndex: "id",
  },
  {
    title: "标题",
    dataIndex: "title",
  },
  {
    title: "内容",
    dataIndex: "content",
  },
  {
    title: "标签",
    dataIndex: "tags",
  },
  {
    title: "答案",
    dataIndex: "answer",
  },
  {
    title: "提交数",
    dataIndex: "submitNum",
  },
  {
    title: "通过数",
    dataIndex: "acceptedNum",
  },
  {
    title: "用户Id",
    dataIndex: "userId",
  },
  {
    title: "创建时间",
    dataIndex: "createTime",
  },
  {
    title: "判题用例",
    dataIndex: "judgeCase",
  },
  {
    title: "判题配置",
    dataIndex: "judgeConfig",
  },
  {
    title: "Optional",
    slotName: "optional",
  },
];

const doDelete = async (question: Question) => {
  console.log(question);
  const res = await QuestionControllerService.deleteQuestionUsingPost({
    id: question.id,
  });
  if (res.code === 0) {
    message.success("删除成功");
    // 删除后要更新页面！
    loadData();
  } else {
    message.error("删除失败：" + res.message);
  }
};

const router = useRouter();

const doUpdate = (question: Question) => {
  router.push({
    path: `/update/question`,
    query: {
      id: question.id,
    },
  });
};
</script>

<style scoped>
#manageQuestionView {
  max-width: 1280px;
  margin: 0 auto;
}
</style>
