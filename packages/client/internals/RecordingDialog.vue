<script setup lang="ts">
import { useVModel } from '@vueuse/core'
import { computed, nextTick } from 'vue'
import { bitsPerSecond, compressionOption, compressionOptions, convertMbpsToBps, frameRate, frameRateOptions, getFilename, mimeType, recordCamera, recorder, recordingName } from '../logic/recording'
import DevicesSelectors from './DevicesSelectors.vue'
import Modal from './Modal.vue'

const props = defineProps({
  modelValue: {
    default: false,
  },
})

const emit = defineEmits<{
  (e: any): void
}>()
const value = useVModel(props, 'modelValue', emit)

const bitsPerSecondInMbps = computed({
  get: () => bitsPerSecond.value / (1024 * 1024),
  set: (value: number) => bitsPerSecond.value = value * (1024 * 1024),
})

const { startRecording } = recorder

function isCustomSettings() {
  return compressionOption.value.display === 'Custom settings'
}

function close() {
  value.value = false
}

async function start() {
  close()
  await nextTick()

  const selectedFrameRate = isCustomSettings() ? frameRate.value : compressionOption.value.frameRate
  const selectedBitrate = convertMbpsToBps(isCustomSettings() ? bitsPerSecond.value : compressionOption.value.bitrate)

  startRecording({
    mimeType: mimeType.value,
    frameRate: selectedFrameRate,
    bitsPerSecond: selectedBitrate,
  })
}
</script>

<template>
  <Modal v-model="value" class="px-6 py-4 recording-dialog flex flex-col gap-2">
    <div class="flex gap-2 text-xl">
      <div class="i-carbon:video my-auto" />Recording
    </div>
    <div class="grid grid-cols-2 gap-4">
      <div class="flex flex-col gap-2 py-2">
        <div class="form-text">
          <label for="title">Recording Name</label>
          <input
            v-model="recordingName"
            class="bg-transparent text-current"
            name="title"
            type="text"
            placeholder="Enter the title..."
          >
          <div class="text-xs w-full opacity-50 py-2">
            <div>This will be used in the output filename that might <br>help you better organize your recording chips.</div>
          </div>
        </div>
        <div class="form-check">
          <input
            v-model="recordCamera"
            name="record-camera"
            type="checkbox"
          >
          <label for="record-camera" @click="recordCamera = !recordCamera">Record camera separately</label>
        </div>

        <div class="form-text">
          <label for="title">Compression</label>
          <select
            v-model="compressionOption"
            class="bg-transparent text-current"
            name="title"
          >
            <option v-for="item in compressionOptions" :key="item.display" :value="item">
              {{ item.display }}
            </option>
          </select>
          <div class="text-xs w-full opacity-50 py-2">
            <div>Higher compression results in smaller file size but may affect quality.</div>
          </div>
        </div>
        <div v-if="compressionOption.display === 'Custom settings'" class="flex flex-col gap-2">
          <!-- Custom Frame Rate Dropdown -->
          <div class="form-text flex">
            <label for="custom-frame-rate">Custom Frame Rate</label>
            <div class="flex items-center gap-1">
              <select
                v-model="frameRate"
                class="bg-transparent text-current flex-1"
                name="custom-frame-rate"
              >
                <option v-for="option in frameRateOptions" :key="option" :value="option">
                  {{ option }} fps
                </option>
              </select>
            </div>
          </div>

          <!-- Custom Bitrate Slider -->
          <div class="form-text flex">
            <label for="custom-bitrate">Custom Bitrate</label>
            <div class="flex items-center gap-2">
              <input
                v-model.number="bitsPerSecondInMbps"
                type="range"
                class="w-full flex-1 accent-indigo-500"
                name="custom-bitrate"
                min="10"
                max="100"
                step="1"
              >
              <span class="text-xs opacity-50">{{ bitsPerSecondInMbps }} Mbps</span>
            </div>
          </div>
        </div>
        <!--
        <div class="form-text">
          <label for="title">Frame Rate</label>
          <select
            v-model="frameRate"
            class="bg-transparent text-current"
            name="title"
          >
            <option v-for="item in frameRateOptions" :key="item" :value="item">
              {{ item }} fps
            </option>
          </select>
          <div class="text-xs w-full opacity-50 py-2">
            <div>Higher frame rates result in smoother videos but <br>may increase file size and processing requirements.</div>
          </div>
        </div>

        <div class="form-text">
          <label for="title">Bitrate</label>
          <select
            v-model="bitsPerSecond"
            class="bg-transparent text-current"
            name="title"
          >
            <option v-for="item in bitrateOptions" :key="item" :value="convertMbpsToBps(item)">
              {{ item }} Mbps
            </option>
          </select>
          <div class="text-xs w-full opacity-50 py-2">
            <div>Higher bitrate results in better quality but larger file size.</div>
          </div>
        </div> -->

        <div class="text-xs w-full opacity-50">
          <div class="mt-2 opacity-50">
            Enumerated filenames
          </div>
          <div class="font-mono">
            {{ getFilename('screen', mimeType) }}
          </div>
          <div v-if="recordCamera" class="font-mono">
            {{ getFilename('camera', mimeType) }}
          </div>
        </div>
      </div>
      <DevicesSelectors />
    </div>
    <div class="flex my-1">
      <button class="slidev-form-button" @click="close">
        Cancel
      </button>
      <div class="flex-auto" />
      <button class="slidev-form-button primary" @click="start">
        Start
      </button>
    </div>
  </Modal>
</template>

<style lang="postcss">
.recording-dialog {
  .form-text {
    @apply flex flex-col;

    label {
      @apply text-xs uppercase opacity-50 tracking-widest py-1;
    }
  }

  .form-check {
    @apply leading-5;

    * {
      @apply my-auto align-middle;
    }

    label {
      @apply ml-1 text-sm select-none;
    }
  }

  input[type='text'],
  input[type='number'] {
    @apply border border-main rounded px-2 py-1;
  }
}
</style>
