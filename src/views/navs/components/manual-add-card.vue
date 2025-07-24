<template>
  <PopoverWrapper :message="ft('add', 'card')" placement="bottom-end">
    <n-button
      tertiary
      :focusable="false"
      size="small"
      class="w-[28px] !shadow-btn-shadow"
      @click="onAddCard"
    >
      <template #icon>
        <n-icon size="20" :component="Add" />
      </template>
    </n-button>
  </PopoverWrapper>
</template>

<script setup lang="tsx">
import PopoverWrapper from '@/components/popover-wrapper.vue'
import { Add } from '@vicons/carbon'
import { useEditDialog } from '@/hooks/useEditDialog.tsx'
import { useHelpi18n } from '@/hooks/useHelpi18n.ts'
import { useRefresh } from '@/hooks/useRresh.ts'
import dataManager from '@/db'
import { useSpacesStore } from '@/store/spaces.ts'

const { open } = useEditDialog()
const { ft } = useHelpi18n()
const { refreshCollections } = useRefresh()
const spacesStore = useSpacesStore()

function onAddCard() {
  const collections = spacesStore.collections
  const defaultCollectionId = collections.length > 0 ? collections[0].id : null
  const formModel = ref({
    title: '',
    url: '',
    description: '',
    collectionId: defaultCollectionId,
  })
  open({
    title: ft('add', 'card'),
    renderContent: () => (
      <n-form model={formModel.value}>
        <n-form-item label={`${ft('title')}:`}>
          <n-input v-model:value={formModel.value.title} placeholder={ft('placeholder', 'title')} />
        </n-form-item>
        <n-form-item label={`${ft('url')}:`}>
          <n-input v-model:value={formModel.value.url} placeholder={ft('placeholder', 'url')} />
        </n-form-item>
        <n-form-item label={`${ft('description')}:`}>
          <n-input v-model:value={formModel.value.description} placeholder={ft('placeholder', 'description')} />
        </n-form-item>
      </n-form>
    ),
    onPositiveClick: async () => {
      if (!formModel.value.title || !formModel.value.url || !formModel.value.collectionId) return
      await dataManager.addCard({
        title: formModel.value.title,
        url: formModel.value.url,
        description: formModel.value.description,
        collectionId: formModel.value.collectionId,
      })
      await refreshCollections()
    },
  })
}
</script> 