<template>
  <section style="margin-bottom: 10px">
    <Toolbar
      :title="$t('chapters')"
      :menu-items="toolbarMenuItems"
      @title-clicked="toggleExpand"
    />
    <v-divider />
    <Container v-if="expanded">
      <v-list>
        <v-list-item
          v-for="chapter in itemDetails?.metadata?.chapters"
          :key="chapter.position"
          :disabled="!itemIsAvailable(itemDetails)"
          @click="chapterClicked(chapter)"
        >
          <template #prepend>
            <div style="width: 50px">
              <v-img
                v-if="chapterThumb(chapter)"
                :src="chapterThumb(chapter)"
                loading="lazy"
                width="40"
                height="40"
                cover
                :alt="chapter.name"
                class="rounded"
                @error="onThumbError(chapter)"
              />
              <v-chip v-else>
                {{ chapter.position }}
              </v-chip>
            </div>
          </template>
          <template #title>
            <div>{{ chapter.name }}</div>
          </template>
          <template #append>
            <span v-if="chapter.end" class="text-caption"
              >{{ formatDuration(chapter.end - chapter.start) }}
            </span>
            <a
              v-if="chapter.url"
              :href="chapter.url"
              target="_blank"
              rel="noopener"
              class="chapter-link"
              :title="chapter.url"
              @click.stop
            >
              <ExternalLink :size="16" />
            </a>
          </template>
        </v-list-item>
      </v-list>
    </Container>
  </section>
</template>

<script setup lang="ts">
import { ChevronDown, ChevronUp, ExternalLink } from "@lucide/vue";
import Container from "@/components/Container.vue";
import Toolbar from "@/components/Toolbar.vue";
import {
  formatDuration,
  getMediaImageUrl,
  getMediaItemImage,
  getMediaItemImageUrl,
} from "@/helpers/utils";
import { api } from "@/plugins/api";
import { itemIsAvailable } from "@/plugins/api/helpers";
import {
  ImageType,
  MediaItemChapter,
  type MediaItem,
} from "@/plugins/api/interfaces";
import { computed, ref } from "vue";

export interface Props {
  itemDetails: MediaItem;
}
const props = defineProps<Props>();

const expanded = ref(true);

const toggleExpand = function () {
  expanded.value = !expanded.value;
};

const toolbarMenuItems = computed(() => {
  return [
    // toggle expand
    {
      label: "tooltip.collapse_expand",
      icon: expanded.value ? ChevronUp : ChevronDown,
      action: toggleExpand,
      overflowAllowed: false,
    },
  ];
});

const chapterClicked = function (chapter: MediaItemChapter) {
  if (!props.itemDetails || !itemIsAvailable(props.itemDetails)) return;
  api.playMedia(props.itemDetails.uri, undefined, {
    start_item: chapter.position.toString(),
  });
};

const failedThumbs = ref<string[]>([]);

const episodeCoverUrl = computed(() => {
  const img = getMediaItemImage(props.itemDetails, ImageType.THUMB);
  return img ? getMediaItemImageUrl(img, 256) : "";
});

const chapterThumb = function (chapter: MediaItemChapter): string {
  if (chapter.image && !failedThumbs.value.includes(chapter.image)) {
    return getMediaImageUrl(chapter.image);
  }
  if (
    episodeCoverUrl.value &&
    !failedThumbs.value.includes(episodeCoverUrl.value)
  ) {
    return episodeCoverUrl.value;
  }
  return "";
};

const onThumbError = function (chapter: MediaItemChapter): void {
  const current =
    chapter.image && !failedThumbs.value.includes(chapter.image)
      ? chapter.image
      : episodeCoverUrl.value;
  if (current && !failedThumbs.value.includes(current)) {
    failedThumbs.value.push(current);
  }
};
</script>

<style scoped>
.chapter-link {
  display: inline-flex;
  margin-left: 8px;
  color: inherit;
  opacity: 0.7;
}
.chapter-link:hover {
  opacity: 1;
}
</style>
