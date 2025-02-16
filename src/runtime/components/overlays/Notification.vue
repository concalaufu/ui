<template>
  <transition appear v-bind="ui.transition">
    <div
      :class="[ui.wrapper, ui.background, ui.rounded, ui.shadow]"
      @mouseover="onMouseover"
      @mouseleave="onMouseleave"
    >
      <div :class="[ui.container, ui.rounded, ui.ring]">
        <div class="p-4">
          <div class="flex gap-3" :class="{ 'items-start': description, 'items-center': !description }">
            <UIcon v-if="icon" :name="icon" :class="ui.icon" />
            <UAvatar v-if="avatar" v-bind="avatar" :class="ui.avatar" />

            <div class="w-0 flex-1">
              <p :class="ui.title">
                {{ title }}
              </p>
              <p v-if="description" :class="ui.description">
                {{ description }}
              </p>

              <div v-if="description && actions.length" class="mt-3 flex items-center gap-2">
                <UButton v-for="(action, index) of actions" :key="index" v-bind="{ ...ui.default.action, ...action }" @click.stop="onAction(action)" />
              </div>
            </div>
            <div class="flex-shrink-0 flex items-center gap-3">
              <div v-if="!description && actions.length" class="flex items-center gap-2">
                <UButton v-for="(action, index) of actions" :key="index" v-bind="{ ...ui.default.action, ...action }" @click.stop="onAction(action)" />
              </div>

              <UButton v-if="close" v-bind="{ ...ui.default.close, ...close }" @click.stop="onClose" />
            </div>
          </div>
        </div>
        <div v-if="timeout" :class="ui.progress" :style="progressBarStyle" />
      </div>
    </div>
  </transition>
</template>

<script lang="ts">
import { ref, computed, onMounted, onUnmounted, watchEffect, defineComponent } from 'vue'
import type { PropType } from 'vue'
import { defu } from 'defu'
import UIcon from '../elements/Icon.vue'
import UAvatar from '../elements/Avatar.vue'
import UButton from '../elements/Button.vue'
import { useTimer } from '../../composables/useTimer'
import type { ToastNotificationAction } from '../../types'
import type { Avatar as AvatarType } from '../../types/avatar'
import type { Button as ButtonType } from '../../types/button'
import { useAppConfig } from '#imports'
// TODO: Remove
// @ts-expect-error
import appConfig from '#build/app.config'

// const appConfig = useAppConfig()

export default defineComponent({
  components: {
    UIcon,
    UAvatar,
    UButton
  },
  props: {
    id: {
      type: [String, Number],
      required: true
    },
    title: {
      type: String,
      required: true
    },
    description: {
      type: String,
      default: null
    },
    icon: {
      type: String,
      default: null
    },
    avatar: {
      type: Object as PropType<Partial<AvatarType>>,
      default: null
    },
    close: {
      type: Object as PropType<Partial<ButtonType>>,
      default: () => appConfig.ui.notification.default.close
    },
    timeout: {
      type: Number,
      default: 5000
    },
    actions: {
      type: Array as PropType<ToastNotificationAction[]>,
      default: () => []
    },
    callback: {
      type: Function,
      default: null
    },
    ui: {
      type: Object as PropType<Partial<typeof appConfig.ui.notification>>,
      default: () => appConfig.ui.notification
    }
  },
  emits: ['close'],
  setup (props, { emit }) {
    // TODO: Remove
    const appConfig = useAppConfig()

    const ui = computed<Partial<typeof appConfig.ui.notification>>(() => defu({}, props.ui, appConfig.ui.notification))

    let timer: any = null
    const remaining = ref(props.timeout)

    const progressBarStyle = computed(() => {
      const remainingPercent = remaining.value / props.timeout * 100

      return { width: `${remainingPercent || 0}%` }
    })

    function onMouseover () {
      if (timer) {
        timer.pause()
      }
    }

    function onMouseleave () {
      if (timer) {
        timer.resume()
      }
    }

    function onClose () {
      if (timer) {
        timer.stop()
      }

      if (props.callback) {
        props.callback()
      }

      emit('close')
    }

    function onAction (action: ToastNotificationAction) {
      if (timer) {
        timer.stop()
      }

      if (action.click) {
        action.click()
      }

      emit('close')
    }

    onMounted(() => {
      if (!props.timeout) {
        return
      }

      timer = useTimer(() => {
        onClose()
      }, props.timeout)

      watchEffect(() => {
        remaining.value = timer.remaining.value
      })
    })

    onUnmounted(() => {
      if (timer) {
        timer.stop()
      }
    })

    return {
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      progressBarStyle,
      onMouseover,
      onMouseleave,
      onClose,
      onAction
    }
  }
})
</script>
