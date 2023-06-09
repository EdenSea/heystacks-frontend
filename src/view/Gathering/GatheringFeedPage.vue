<script lang="ts" setup>
import { onBeforeMount, reactive, ref, toRef } from 'vue'

import LayoutLight from '@/layouts/LayoutLight.vue'
import SearchInput from '@/components/ui/SearchInput.vue'
import TagButton from '@/components/atoms/TagButton.vue'
import FeedCard from '@/components/atoms/FeedCard.vue'
import TitleHr from '@/components/atoms/TitleHr.vue'
import HeaderNav from '@/components/layoutElements/HeaderNav.vue'
import HeaderContent from '@/components/layoutElements/HeaderContent.vue'
import Footer from '@/components/layoutElements/Footer.vue'
import AddNewDocModal from '@/components/docs/AddNewDocModal.vue'
import TipModal from '@/components/payments/TipModal.vue'

import { Dialog, DialogPanel } from '@headlessui/vue'
import { PlusSmIcon } from '@heroicons/vue/solid'
import { HeartIcon } from '@heroicons/vue/outline'

import useDocs, { DocsFilters } from '@/composables/web2/useDocs'
import useTags from '@/composables/web2/useTags'
import gatheringSlug from '@/composables/utils/useGatheringSlug'

// consts
const initialFilters = {
  page: 0,
  search: '',
  sort: 'Hot',
  type: -1,
  tags: [],
  limit: 2,
}

// state
const filters = reactive<DocsFilters>({ ...initialFilters })
const newDocumentModal = ref(false)

// composables
const { docs, refetch: refetchDocs } = useDocs(filters)
const { tags, refetch: refetchTags } = useTags(1, toRef(filters, 'sort'))

// methods
function toggleTags(tag: string) {
  filters.page = 0
  const tagIdx = filters.tags.indexOf(tag)
  if (tagIdx < 0) {
    filters.tags.push(tag)
    if (!docs.value?.result.length) filters.search = ''
  } else {
    filters.tags.splice(tagIdx, 1)
  }
}

// lifecycle
onBeforeMount(() => {
  if (docs.value) refetchDocs.value()
  if (tags.value) refetchTags.value()
})
</script>
<template>
  <LayoutLight>
    <template #header-nav>
      <HeaderNav />
    </template>
    <template #header-content>
      <HeaderContent
        :navLink="{
          label: 'Backstage >',
          path: `/g/${gatheringSlug}/backstage`,
        }"
      />
    </template>
    <template #command-band>
      <div class="mx-auto mb-4 max-w-screen-md text-center flex-center">
        <SearchInput
          :feed="true"
          :initialSearch="filters.search"
          :searching="false"
          :existingTags="[]"
          @search="
            search => {
              filters.search = search
            }
          "
          @selected-tag="
            tag => {
              alert(tag)
            }
          "
        />
        <button
          title="Share an interesting Google doc"
          class="py-2 px-4 m-2 bg-thgreen8 btn-white"
          @click="newDocumentModal = true"
        >
          <div style="width: 1rem" class="inline-block">
            <PlusSmIcon class="w-6 h-6" />
          </div>
          <span class="hidden sm:inline-block ml-2"> New </span>
        </button>
        <!-- <button title="Tip" class="py-2 px-4 m-2 bg-thgreen8 btn-white" @click="tipModal = true">
          <span class="hidden sm:inline-block ml-2"> Tip </span>
        </button> -->
      </div>
      <div class="flex-wrap mx-auto max-w-screen-md text-center flex-center">
        <TagButton
          v-for="tag in (docs?.remindingTags?.length ? docs?.remindingTags : tags ?? [])
            .slice(0, 10)
            .map(tag => ({ id: tag.id, text: tag.title, title: tag.title }))"
          :key="`tag-${tag.title}`"
          :tag="tag"
          :selected="filters.tags.includes(tag.title)"
          @click="toggleTags(tag.title)"
        />
        <span v-if="tags?.length > 10" class="ml-2"> ... </span>
      </div>
    </template>
    <template #content>
      <div class="mx-auto mb-12">
        <TitleHr :title="'Results'" />
        <FeedCard v-for="(doc, did) in docs?.result" :key="`doc-${did}`" :index="did">
          <h5>{{ doc.title }}</h5>
          <small class="block mb-2 text-grey"
            >{{ new Date(doc.createdAt).toLocaleDateString() }}
          </small>
          <p class="mb-2">{{ doc.description }}</p>
          <div class="mb-2">
            •
            <div v-for="tag in doc.tags" :key="`tag-${did}-${tag}`" class="inline-block ml-1">
              <span class="link" @click="toggleTags(tag)">
                {{ tag }}
              </span>
              •
            </div>
          </div>
          <div>
            <small class="flex mr-2">
              <HeartIcon class="w-[1rem]" />
              <span class="ml-1">
                {{ doc.upvotes }}
              </span>
            </small>
            <!-- <small class="mr-2"> Comments: {{ doc.comments }} </small> -->
          </div>
        </FeedCard>
      </div>
      <div v-if="!docs?.hasNextPage" class="flex justify-center">
        <button class="px-1 text-xs rounded border border-black" @click="filters.limit += 2">
          Load More
        </button>
      </div>

      <div v-if="docs?.length > 10" class="flex-center">
        <button
          v-if="filters.page > 0"
          title="Previous page"
          class="py-2 px-4 m-2 bg-thgreen1 btn-white"
          @click="filters.page -= 1"
        >
          {{ `<` }}
        </button>
        <button
          title="Previous page"
          class="py-2 px-4 m-2 bg-thgreen1 btn-white"
          @click="filters.page += 1"
        >
          {{ `>` }}
        </button>
      </div>

      <Dialog :open="newDocumentModal" @close="newDocumentModal = false">
        <div class="fixed inset-0 bg-black/30 backdrop-blur-[2px]" aria-hidden="true" />
        <div class="flex inset-0 z-50 justify-center items-center p-4">
          <DialogPanel class="p-8 max-w-[40rem] bg-white rounded">
            <AddNewDocModal
              :existingTags="tags?.map(tag => ({ text: tag.title, value: tag.title }))"
              :gathering="gatheringSlug"
              @close="newDocumentModal = false"
            ></AddNewDocModal>
          </DialogPanel>
        </div>
      </Dialog>
      <!-- <Dialog :open="tipModal" @close="tipModal = false">
        <div class="fixed inset-0 bg-black/30 backdrop-blur-[2px]" aria-hidden="true" />
        <div class="flex fixed inset-0 z-50 justify-center items-center p-4">
          <DialogPanel class="p-8 w-full bg-white rounded max-w-[40rem]">
            <TipModal />
          </DialogPanel>
        </div>
      </Dialog> -->

      <div
        v-if="newDocumentModal"
        class="backdrop"
        @click="
          () => {
            newDocumentModal = false
          }
        "
      ></div>
    </template>
    <template #footer>
      <Footer />
    </template>
  </LayoutLight>
</template>
