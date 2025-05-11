<template>
  <div class="picture-list">
    <!-- 图片列表 -->
    <a-list
      :grid="{ gutter: 16, xs: 1, sm: 2, md: 3, lg: 4, xl: 5, xxl: 6 }"
      :data-source="pictureList"
      :loading="loading"
    >
      <template #renderItem="{ item: picture }">
        <a-list-item style="padding: 0">
          <!-- 单张图片 -->
          <a-card hoverable @click="doClickPicture(picture)">
            <template #cover>
              <img
                :alt="picture.name"
                :src="picture.thumbnailUrl ?? picture.url"
                style="height: 180px; object-fit: cover"
              />
            </template>
            <a-card-meta :title="picture.name">
              <template #description>
                <a-flex>
                  <!--审核信息-->
                  <a-tag color="blue" v-if="picture.reviewStatus === 0">审核中</a-tag>
                  <a-tag color="green" v-if="picture.reviewStatus === 1">已通过</a-tag>
                  <a-tag color="red" v-if="picture.reviewStatus === 2">未通过</a-tag>
                </a-flex>
              </template>
            </a-card-meta>

          </a-card>
        </a-list-item>
      </template>
    </a-list>
  </div>

</template>

<script setup lang="ts">
import {useLoginUserStore} from "@/stores/useLoginUserStore.ts";
import {onMounted, reactive, ref} from "vue";
import {message} from "ant-design-vue";
import {
  listPictureVoByMyselfByPageUsingPost
} from "@/api/pictureController.ts";

const loginUser = useLoginUserStore().loginUser;

// 定于图片列表
const pictureList = ref<API.PictureVO[]>([])

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({
  current: 1,
  pageSize: 15,
  sortField: 'createTime',
  sortOrder: 'descend',
})

// 根据用户id获取图片列表
const getPictureList = async () => {

  const userId = loginUser.id;
  if (userId == null || userId <= 0) {
    message.error("用户id不能为空");
  }
  // 转换搜索参数
  const params = {
    ...searchParams,
    userId: userId,
  }
  const res = await listPictureVoByMyselfByPageUsingPost(params)
  if (res.data.code === 0 && res.data.data){
    pictureList.value = res.data.data?.records ?? [];
    message.success("加载成功")
  } else {
    message.error("加载失败："+res.data.message)
  }
}

onMounted(()=>{
  getPictureList();
})

</script>
<style scoped>

</style>
