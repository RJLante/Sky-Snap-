<template>
  <div id="addPicturePage">
    <h2 style="margin-bottom: 16px">
      {{ route.query?.id ? '编辑图片' : '创建图片' }}
    </h2>
    <!-- 选择上传方式 -->
    <a-tabs v-model:activeKey="uploadType">
      <a-tab-pane key="file" tab="文件上传">
        <!-- 图片上传组件 -->
        <PictureUpload :picture="picture" :onSuccess="onSuccess" />
      </a-tab-pane>
      <a-tab-pane key="url" tab="链接上传" force-render>
        <!-- URL 图片上传组件 -->
        <UrlPictureUpload :picture="picture" :onSuccess="onSuccess" />
      </a-tab-pane>
    </a-tabs>

    <!-- 图片信息表单 -->
    <a-form v-if="picture" name="pictureForm" layout="vertical" :model="pictureForm" @finish="handleSumbit">
      <a-form-item name="name" label="名称">
        <a-input v-model:value="pictureForm.name" placeholder="请输入名称" allow-clear />
      </a-form-item>
      <a-form-item name="introduction" label="简介">
        <a-textarea v-model:value="pictureForm.introduction" placeholder="请输入简介"
                    :auto-size="{ minRows: 2, maxRows: 5}" allow-clear />
      </a-form-item>
      <a-form-item name="category" label="分类">
        <a-auto-complete v-model:value="pictureForm.category" placeholder="请输入分类" :options="categoryOptions"
                         allow-clear />
      </a-form-item>
      <a-form-item name="tags" label="标签">
        <a-select v-model:value="pictureForm.tags" mode="tags" placeholder="请输入标签" :options="tagOptions"
                  allow-clear />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" html-type="submit" style="width: 100%">创建</a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">

import PictureUpload from '@/components/PictureUpload.vue'
import { onMounted, reactive, ref } from 'vue'
import {
  editPictureUsingPost,
  getPictureVoByIdUsingGet,
  listPictureTagCategoryUsingGet
} from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import { useRoute, useRouter } from 'vue-router'
import UrlPictureUpload from '@/components/UrlPictureUpload.vue'

const picture = ref<API.PictureVO>()
const pictureForm = reactive<API.PictureEditRequest>({})
const uploadType = ref<'file' | 'url'>('file')

/**
 * 图片上传成功
 * @param newPicture
 */
const onSuccess = (newPicture: API.PictureVO) => {
  picture.value = newPicture
  pictureForm.name = newPicture.name
}

const router = useRouter()

/**
 * 提交表单
 */
const handleSumbit = async (values: any) => {
  const pictureId = picture.value?.id
  if (!pictureId) {
    return
  }
  const res = await editPictureUsingPost({
    id: pictureId,
    ...values
  })
  // 操作成功
  if (res.data.code === 0 && res.data.data) {
    message.success('创建成功')
    // 跳转到图片详情页
    router.push({
      path: `/picture/${pictureId}`
    })
  } else {
    message.error('创建失败' + res.data.message)
  }
}

const categoryOptions = ref<string[]>()
const tagOptions = ref<string[]>()

/**
 * 获取标签和分类选项
 */
const getTagCategoryOptions = async () => {
  const res = await listPictureTagCategoryUsingGet()
  if (res.data.code === 0 && res.data.data) {
    tagOptions.value = (res.data.data.tagList ?? []).map((data: string) => {
      return {
        value: data,
        label: data
      }
    })
    categoryOptions.value = (res.data.data.categoryList ?? []).map((data: string) => {
      return {
        value: data,
        label: data
      }
    })
  } else {
    message.error('获取标签分类列表失败' + res.data.message)
  }
}

const route = useRoute()

// 获取老数据
const getOldPicture = async () => {
  // 获取到 id
  const id = route.query?.id
  if (id) {
    const res = await getPictureVoByIdUsingGet({
      id
    })
    if (res.data.code === 0 && res.data.data) {
      const data = res.data.data
      picture.value = data
      pictureForm.name = data.name
      pictureForm.introduction = data.introduction
      pictureForm.category = data.category
      pictureForm.tags = data.tags
    }
  }
}

onMounted(() => {
  getTagCategoryOptions()
  getOldPicture()
})

</script>

<style scoped>
#addPicturePage {
  max-width: 720px;
  margin: 0 auto;
}
</style>
