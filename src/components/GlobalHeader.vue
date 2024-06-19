<template>
  <a-row id="globalHeader" align="center" :wrap="false">
    <a-col flex="100px">
      <div></div>
    </a-col>
    <a-col flex="auto">
      <div>
        <a-menu
          mode="horizontal"
          :selected-keys="selectedKeys"
          @menu-item-click="doMenuClick"
        >
          <a-menu-item
            key="0"
            :style="{ padding: 0, marginRight: '38px' }"
            disabled
          >
            <div class="title-bar">
              <img class="logo" src="../assets/mylog.png" />
              <!--              <div class="title">Online jundge</div>-->
            </div>
          </a-menu-item>
          <a-menu-item v-for="item in visibleRouters" :key="item.path"
            >{{ item.name }}
          </a-menu-item>
        </a-menu>
      </div>
    </a-col>
    <a-col flex="100px">
      <a-space>
        <a-button type="primary" @click="doClick">{{ store.state.user?.loginUser?.userName ?? "未登录" }}</a-button>
      </a-space>
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { routes } from "../router/routes";
import { useRouter } from "vue-router";
import { computed, ref } from "vue";
import { useStore } from "vuex";
import checkAccess from "@/access/checkAccess";
import ACCESS_ENUM from "@/access/accessEnum";

const router = useRouter();
const store = useStore();
const loginUser = store.state.user.loginUser;

const visibleRouters = computed(() => {
  return routes.filter((item, index) => {
    if (item.meta?.hideInMenu) {
      return false;
    }
    // TODO 根据权限过滤菜单
    if (!checkAccess(loginUser, item?.meta?.access as string)) {
      return false;
    }
    return true;
  });
});

//默认主页
const selectedKeys = ref(["/"]);

//路由跳转时，更新选中的菜单项
router.afterEach((to, from, failure) => {
  selectedKeys.value = [to.path];
});

setTimeout(() => {
  store.dispatch("user/getLoginUser", {
    userName: "shin admin",
    userRole: ACCESS_ENUM.ADMIN,
  });
}, 3000);

const doMenuClick = (key: string) => {
  router.push({ path: key });
};

/**
 * 跳转到登录
 * @param question
 */
const doClick = () => {
  console.log("用户登录");
  router.push({
    path: `/user/login`,
  });
};
</script>
<style scoped>
.title-bar {
  align-items: center;
  height: 60px;
}

.logo {
  height: 48px;
}

.title {
  align-content: center;
  color: #444;
}
</style>
