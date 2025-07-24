<template>
  <div
    class="hidden items-center gap-x-2 group-hover/item:flex"
    :class="{ '!flex': isShowTagAction }"
  >
    <PopoverWrapper :message="ft('add', 'card')">
      <n-button
        quaternary
        size="small"
        class="w-[28px]"
        @click="onAddCard(item)"
      >
        <template #icon>
          <n-icon :component="Add" size="18" />
        </template>
      </n-button>
    </PopoverWrapper>
    <PopoverWrapper :message="ft('edit', 'collection')">
      <n-button
        quaternary
        size="small"
        class="w-[28px]"
        @click="onEditCollection(item)"
      >
        <template #icon>
          <n-icon :component="Edit" size="18" />
        </template>
      </n-button>
    </PopoverWrapper>
    <PopoverWrapper :message="ft('move-collection')">
      <n-button
        quaternary
        size="small"
        class="w-[28px]"
        @click="onMoveCollection(item)"
      >
        <template #icon>
          <n-icon :component="FolderMoveTo" size="18" />
        </template>
      </n-button>
    </PopoverWrapper>
    <TagAction :item="item" />
  </div>
</template>

<script setup lang="tsx">
import { CollectionWithCards } from "@/type.ts"
import { FolderMoveTo, Delete, Edit, Add } from "@vicons/carbon"
import { useDialog } from "naive-ui"
import dataManager from "@/db"
import { useRefresh } from "@/hooks/useRresh.ts"
import TagAction from "./tag-action.vue"
import { useHelpi18n } from "@/hooks/useHelpi18n"
import { useEditDialog } from "@/hooks/useEditDialog.tsx"
import { useDeleteDialog } from "@/hooks/useDeleteDialog.tsx"
import { useBatchMoveCollectionDialog } from "@/hooks/useBatchMoveCollectionDialog.tsx"
import PopoverWrapper from "@/components/popover-wrapper.vue"

const { ft, gt } = useHelpi18n()
defineProps<{
  item: CollectionWithCards
}>()

const { refreshCollections, updateContextMenus } = useRefresh()
const isShowTagAction = ref(false)

provide("isShowTagAction", {
  isShowTagAction,
  setIsShowTagAction: (value: boolean) => {
    isShowTagAction.value = value
  },
})

const { open: openEditDialog } = useEditDialog()
const { open: openDeleteDialog } = useDeleteDialog()

function onAddCard(item: CollectionWithCards) {
  const formModel = ref({
    title: '',
    url: '',
    description: '',
  })
  openEditDialog({
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
      if (!formModel.value.title || !formModel.value.url) return
      await dataManager.addCard({
        title: formModel.value.title,
        url: formModel.value.url,
        description: formModel.value.description,
        collectionId: item.id,
      })
      await refreshCollections()
    },
  })
}

function onEditCollection(item: CollectionWithCards) {
  const formModel = ref({ title: item.title })
  openEditDialog({
    title: ft("edit", "collection"),
    renderContent: () => (
      <n-form model={formModel.value}>
        <n-form-item label={`${ft("title")}:`} class="!text-text-primary">
          <n-input-group>
            <n-input
              v-model:value={formModel.value.title}
              placeholder={ft("placeholder", "title")}
            />
            <n-button
              ghost
              type="error"
              onClick={() => onDeleteCollection(item)}
              v-slots={{
                icon: () => <n-icon size="16" component={Delete} />,
              }}
            />
          </n-input-group>
        </n-form-item>
      </n-form>
    ),
    onPositiveClick: async () => {
      await dataManager.updateCollectionTitle(item.id, formModel.value.title)
      await refreshCollections()
      await updateContextMenus()
    },
  })
}

const dialog = useDialog()
function onDeleteCollection(item: CollectionWithCards) {
  openDeleteDialog({
    title: ft("delete", "collection"),
    content: () => (
      <span
        class="text-text-primary"
        v-html={gt("delete-confirm", item.title)}
      />
    ),
    onPositiveClick: async () => {
      await dataManager.removeCollection(item.id)
      await refreshCollections()
      await updateContextMenus()
      dialog.destroyAll()
    },
  })
}

const { openDialog: openMoveDialog } = useBatchMoveCollectionDialog()
async function onMoveCollection(item: CollectionWithCards) {
  const { spaceId, position } = await openMoveDialog(
    gt("move-type-to", item.title),
  )
  await dataManager.batchUpdateCollections(
    [item.id],
    { spaceId: spaceId! },
    position,
  )
  await refreshCollections()
  await refreshCollections(spaceId as number)
  await updateContextMenus()
}
</script>
