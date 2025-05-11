<template>
  <div class="picture-list">
    <Waterfall
      v-if="dataList.length > 0"
      :list="dataList"
      :width="300"
      :breakpoints="breakpoints"
    >
      <template #default="{ item, url, index }">
        <a-card hoverable>
          <template #cover>
            <!-- @dragstart="handleDragStart" 禁止拖拽 -->
            <div @dragstart="handleDragStart" @click="doClickPicture(item)">
              <LazyImg :url="item.thumbnailUrl ? item.thumbnailUrl : item.url" />
            </div>
          </template>
          <a-card-meta
            :title="item.name"
            :description="`作者: ${item.user.userName}`"
            :bodyStyle="{ color: '#fff' }"
          >
            <template #avatar>
              <a-avatar size="large" :src="item.user.userAvatar" />
            </template>
          </a-card-meta>
          <template v-if="showOp" #actions>
            <ShareAltOutlined @click="(e) => doShare(item, e)" />
            <SearchOutlined @click="(e) => doSearch(item, e)" />
            <EditOutlined v-if="canEdit" @click="(e) => doEdit(item, e)" />
            <DeleteOutlined v-if="canDelete" @click="(e) => doDelete(item, e)" />
          </template>
        </a-card>
      </template>
    </Waterfall>
    <ShareModal ref="shareModalRef" :link="shareLink" />
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
import {
  DeleteOutlined,
  EditOutlined,
  SearchOutlined,
  ShareAltOutlined,
} from '@ant-design/icons-vue'
import { deletePictureUsingPost } from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import ShareModal from '@/components/ShareModal.vue'
import { ref } from 'vue'
import {LazyImg, Waterfall} from 'vue-waterfall-plugin-next'
import 'vue-waterfall-plugin-next/dist/style.css'


/**
 * 瀑布流布局
 */
const breakpoints = ref({
  3000: {
    //当屏幕宽度小于等于3000
    rowPerView: 7, // 一行8图
  },
  1800: {
    rowPerView: 6,
  },
  1500: {
    rowPerView: 5,
  },
  1200: {
    rowPerView: 4,
  },
  1000: {
    rowPerView: 3,
  },
  800: {
    rowPerView: 2,
  },
  700: {
    rowPerView: 2,
  },
  500: {
    rowPerView: 1,
  },
  300: {
    rowPerView: 1,
  },
})

interface Props {
  dataList?: API.PictureVO[]
  loading?: boolean
  showOp?: boolean
  canEdit?: boolean
  canDelete?: boolean
  onReload?: () => void
}

const props = withDefaults(defineProps<Props>(), {
  dataList: () => [],
  loading: false,
  showOp: false,
  canEdit: false,
  canDelete: false,
})

const router = useRouter()
// 跳转至图片详情页
const doClickPicture = (picture: API.PictureVO) => {
  router.push({
    path: `/picture/${picture.id}`,
  })
}

// 搜索
const doSearch = (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  // 打开新的页面
  window.open(`/search_picture?pictureId=${picture.id}`)
}

// 编辑
const doEdit = (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  // 跳转时一定要携带 spaceId
  router.push({
    path: '/add_picture',
    query: {
      id: picture.id,
      spaceId: picture.spaceId,
    },
  })
}

// 删除数据
const doDelete = async (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  const id = picture.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
    props.onReload?.()
  } else {
    message.error('删除失败')
  }
}

// ----- 分享操作 ----
const shareModalRef = ref()
// 分享链接
const shareLink = ref<string>()
// 分享
const doShare = (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  shareLink.value = `${window.location.protocol}//${window.location.host}/picture/${picture.id}`
  if (shareModalRef.value) {
    shareModalRef.value.openModal()
  }
}
</script>

<style scoped>

</style>
