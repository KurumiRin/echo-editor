<script setup lang="ts">
import { reactive, watchEffect, ref, onMounted } from 'vue'
import { Switch } from '@/components/ui/switch'
import { Icon } from '@/components/icons'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Button } from '@/components/ui/button'
import { Checkbox } from '@/components/ui/checkbox'
import { onClickOutside } from '@vueuse/core'
import type { Editor } from '@tiptap/vue-3'
import { useLocale } from '@/locales'
import { useFocus } from '@vueuse/core'

interface Props {
  editor: Editor
}
const props = withDefaults(defineProps<Props>(), {})
const emits = defineEmits(['onSetLink', 'onClickOutside'])

const { t } = useLocale()

let form = reactive({
  text: '',
  link: '',
})
const inputRef = ref<HTMLInputElement | null>(null)
const { focused } = useFocus(inputRef)
const openInNewTab = ref<boolean>(false)
const target = ref(null)
onClickOutside(target, event => emits('onClickOutside', event))

watchEffect(() => {
  const { href: link, target } = props.editor.getAttributes('link')
  const { from, to } = props.editor.state.selection
  const text = props.editor.state.doc.textBetween(from, to, ' ')
  form = {
    link,
    text,
  }
  openInNewTab.value = target === '_blank' ? true : false
})
function handleSubmit() {
  emits('onSetLink', form.link, form.text, openInNewTab.value)
}
onMounted(() => {
  focused.value = true
})
</script>

<template>
  <div ref="target" class="p-2 rounded-lg bg-card shadow-sm border">
    <form @submit.prevent="handleSubmit" class="flex flex-col gap-2">
      <Label> {{ t('editor.link.dialog.text') }} </Label>
      <div class="flex w-full max-w-sm items-center gap-1.5">
        <div class="relative w-full max-w-sm items-center">
          <Input type="text" v-model="form.text" required class="w-80" placeholder="输入文本" />
        </div>
      </div>
      <Label>{{ t('editor.link.dialog.link') }}</Label>
      <div class="flex w-full max-w-sm items-center gap-1.5">
        <div class="relative w-full max-w-sm items-center">
          <Input type="url" ref="inputRef" v-model="form.link" required class="pl-10" />
          <span class="absolute start-0 inset-y-0 flex items-center justify-center px-2">
            <Icon class="size-5 text-muted-foreground" name="Link" />
          </span>
        </div>
      </div>
      <div class="flex items-center space-x-2 mt-1">
        <Checkbox v-model="openInNewTab" id="openInNewTab" />
        <Label for="openInNewTab">{{ t('editor.link.dialog.openInNewTab') }}</Label>
      </div>
      <Button type="submit" class="mt-2 self-end">{{ t('editor.link.dialog.button.apply') }} </Button>
    </form>
  </div>
</template>
