<script setup lang="ts">
import type { ModalPlacement } from "../types/typings";
import { IconCircleXFilled } from "@tabler/icons-vue";
// import { useGlobal } from "@/stores/Global";
import { ref, computed, watch, onMounted } from "vue";

// const { $set_scrollbar_width_var } = useGlobal();

type BgColor = "black" | "white";

interface Props {
  placement?: ModalPlacement;
  show?: boolean;
  showButton?: boolean;
  scroll?: { x: boolean; y: boolean };
  blur?: boolean;
  closable?: boolean;
  bgColor?: BgColor;
  stretch?: boolean;
  hoverTransparency?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  placement: "center",
  show: false,
  showButton: false,
  scroll: () => ({ x: false, y: true }),
  blur: false,
  closable: true,
  bgColor: "black",
  stretch: false,
  hoverTransparency: false,
});

const model = defineModel<boolean>();
const contentOpacity = ref(1);

const isOpen = computed({
  get: () => model.value,
  set: (val) => (model.value = val),
});

watch(
  () => props.show,
  (newVal) => {
    if (newVal !== undefined) {
      isOpen.value = newVal;
    }
  }
);

watch(
  () => model.value,
  (newVal) => {
    if (newVal !== undefined) {
      isOpen.value = newVal;
    }
  }
);

watch(
  () => props.hoverTransparency,
  (new_val) => {
    if (!new_val) {
      contentOpacity.value = 1;
    }
  }
);

watch(isOpen, (newVal) => {
  // $set_scrollbar_width_var();

  model.value = newVal;
  if (newVal && props.hoverTransparency) {
    contentOpacity.value = 1;
  }
});

function openModal() {
  isOpen.value = true;
}

function closeModal() {
  if (props.closable) {
    isOpen.value = false;
  }
}

function onContentMouseEnter() {
  if (props.hoverTransparency) {
    contentOpacity.value = 1;
  }
}

function onContentMouseLeave() {
  if (props.hoverTransparency) {
    contentOpacity.value = 0.1;
  }
}

onMounted(() => {
  if (model.value !== undefined) {
    isOpen.value = model.value;
  }
});

/**
 * Tento kod osetruje aby se modal zaviral pouze po primem
 * kliknuti a pusteni mysi na sedou plochu mimo 'content' a
 * ne po kliknuti na obsah, posunuti mysi mimo a pusteni.
 */
const mouseDownOnOverlay = ref(false);
function onOverlayMouseDown(event: MouseEvent) {
  if (event.target === event.currentTarget) {
    mouseDownOnOverlay.value = true;
  } else {
    mouseDownOnOverlay.value = false;
  }
}

function onOverlayMouseUp(event: MouseEvent) {
  if (mouseDownOnOverlay.value && event.target === event.currentTarget) {
    closeModal();
  }
  mouseDownOnOverlay.value = false;
}
</script>

<template>
  <div>
    <button @click="openModal" v-if="props.showButton" class="relative">
      <slot name="button"></slot>
    </button>

    <Teleport to="body">
      <transition>
        <div
          data-modal
          v-if="isOpen"
          @mousedown="onOverlayMouseDown"
          @mouseup="onOverlayMouseUp"
          class="z-10 h-auto fixed inset-0 flex justify-center bg-black/70"
          :class="{
            'bg-black': bgColor == 'black',
            'bg-white': bgColor == 'white',
            'items-center': placement == 'center',
            'items-end pb-2': placement == 'bottom',
            'items-start pt-2': placement == 'top',
            'backdrop-blur-sm': props.blur,
          }"
        >
          <div
            @mouseenter="onContentMouseEnter"
            @mouseleave="onContentMouseLeave"
            @click.stop
            class="md:m-2 bg-transparent text-black rounded-xl modal-content"
            :class="{
              'm-1 absolute inset-0 max-h-dvh': placement == 'fullscreen',
              'max-h-[calc(100dvh-1rem)]': placement != 'fullscreen',
              'w-full m-2': props.stretch,
              'lg:max-w-2/3 m-2': !props.stretch,
              'overflow-auto': props.scroll.x && props.scroll.y,
              'overflow-x-auto overflow-y-hidden':
                props.scroll.x && !props.scroll.y,
              'overflow-x-hidden overflow-y-auto':
                !props.scroll.x && props.scroll.y,
              'overflow-hidden': !props.scroll.x && !props.scroll.y,
            }"
            :style="{ opacity: contentOpacity }"
          >
            <slot name="content"></slot>
          </div>
          <div
            v-if="closable"
            @click.stop="closeModal"
            class="z-10 absolute flex items-center justify-center md:m-3 m-1 top-0 right-0 w-8 h-8 bg-black/20 text-white rounded-full transition duration-150 hover:bg-black/50 cursor-pointer"
          >
            <!-- <IconComponent name="times-s" size="1.9rem" /> -->
            <IconCircleXFilled
              :size="32"
              class="trans-150 opacity-50 hover:opacity-100"
            />
          </div>
        </div>
      </transition>
    </Teleport>
  </div>
</template>

<style>
/*
Dulezite!!
Important!!
This code make sure that when there is a least one modal, the body wont scroll
but when last modal dissapears body would be scroll again.
(modal must have attribute "data-modal")
*/

body:has([data-modal]) {
  overflow: hidden;
  margin-right: var(--scrollbar-width);
}

/**
Vue transition
 */
.v-enter-active,
.v-leave-active {
  transition: opacity 150ms ease-in-out;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.modal-content {
  transition: opacity 0.2s ease;
}
</style>
