<script setup lang="ts">
import { getEmojiMatchesInText } from '@iconify/utils/lib/emoji/replace/find'
import CommonScrollIntoView from '../common/CommonScrollIntoView.vue'
import type { CustomEmoji, Emoji } from '~~/composables/tiptap/suggestion'
import { isCustomEmoji } from '~~/composables/tiptap/suggestion'
import { emojiFilename, emojiPrefix, emojiRegEx } from '~~/config/emojis'

const { items, command } = defineProps<{
  items: (CustomEmoji | Emoji)[]
  command: Function
  isPending?: boolean
}>()

const emojis = computed(() => {
  return items.map((item: CustomEmoji | Emoji) => {
    if (isCustomEmoji(item)) {
      return {
        title: item.shortcode,
        src: item.url,
        emoji: item,
      }
    }

    const skin = item.skins.find(skin => skin.native !== undefined)
    const match = getEmojiMatchesInText(emojiRegEx, skin!.native)[0]
    const file = emojiFilename(match)

    return {
      title: item.id,
      src: `/emojis/${emojiPrefix}/${file.filename}`,
      emoji: item,
    }
  })
})

let selectedIndex = $ref(0)

watch(items, () => {
  selectedIndex = 0
})

function onKeyDown(event: KeyboardEvent) {
  if (items.length === 0)
    return false

  if (event.key === 'ArrowUp') {
    selectedIndex = ((selectedIndex + items.length) - 1) % items.length
    return true
  }
  else if (event.key === 'ArrowDown') {
    selectedIndex = (selectedIndex + 1) % items.length
    return true
  }
  else if (event.key === 'Enter') {
    selectItem(selectedIndex)
    return true
  }

  return false
}

function selectItem(index: number) {
  const emoji = emojis.value[index]
  if (emoji)
    command(emoji)
}

defineExpose({
  onKeyDown,
})
</script>

<template>
  <div
    v-if="isPending || items.length"
    relative bg-base text-base shadow border="~ base rounded"
    text-sm py-2 overflow-x-hidden overflow-y-auto max-h-100
    min-w-40 max-w-50
  >
    <template v-if="isPending">
      <div flex gap-1 items-center p2 animate-pulse>
        <div i-ri:loader-2-line animate-spin />
        <span>Fetching...</span>
      </div>
    </template>
    <template v-if="items.length">
      <CommonScrollIntoView
        v-for="(item, index) in emojis" :key="index"
        :active="index === selectedIndex"
        as="button"
        :class="index === selectedIndex ? 'bg-active' : 'text-secondary'"
        block m0 w-full text-left px2 py1
        @click="selectItem(index)"
      >
        <SearchEmojiInfo :emoji="item" />
      </CommonScrollIntoView>
    </template>
  </div>
  <div v-else />
</template>
