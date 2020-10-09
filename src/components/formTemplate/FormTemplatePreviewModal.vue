<template>
  <div
    class="h-vh100 bg-black bg-opacity-high z-50 fixed w-screen top-0 left-0 flex justify-center items-center"
  >
    <div
      class="flex flex-col items-center text-left w-full h-full relative"
      @click="closeModal"
    >
      <!-- top toolbar -->
      <div class="flex justify-start w-full flex-shrink-0">
        <button class="p-3" @click.prevent="closeModal" title="Close">
          <Close class="text-white" />
        </button>
      </div>

      <!-- content -->
      <div
        class="w-full my-auto overflow-y-auto bg-white flex-grow"
        @click.stop=""
      >
        <PmuFormEmptyPreview
          :tenantSlug="tenantSlug"
          :templateId="templateId"
        />
      </div>

      <!-- caption -->
      <div
        @click.stop=""
        class="w-full content-gradient pb-14 absolute bottom-0"
      >
        <div class="py-10 px-4 text-on-background-image text-opacity-high">
          <h1 class="tg-body-mobile mb-2">
            {{ currentTemplateGet.name }}
          </h1>
          <p class="tg-caption-mobile">{{ createdAt }}</p>
        </div>
      </div>

      <!-- bottom toolbar -->
      <div class="flex justify-end w-full z-20" @click.stop="">
        <div class="flex flex-col justify-center w-full">
          <div class="cta-wrapper">
            <Button
              title="Preview"
              class="uppercase"
              background="bg-primary"
              textColor="text-on-primary"
              @clicked="previewFlow"
            />
          </div>
        </div>

        <!-- share -->
        <a
          v-if="isShareApiSupported()"
          @click.stop="
            shareDataUrl({
              data: imagePreviewBase64,
              filename: previewFileName
            })
          "
          class="cursor-pointer p-4"
          title="Share"
        >
          <ShareIcon class="text-white w-6 h-6" />
        </a>

        <!-- download -->
        <a
          @click.stop="downloadFile"
          class="cursor-pointer p-4"
          :class="[loadingPdf ? 'pointer-events-none' : '']"
          title="Download"
        >
          <DownloadIcon class="text-white w-6 h-6" />
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import Close from '@/assets/icons/close.svg';
import DownloadIcon from '@/assets/icons/download.svg';
import ShareIcon from '@/assets/icons/share.svg';
import {
  isShareApiSupported,
  urlToFile,
  shareDataUrl,
  formatDateTime
} from '@/helpers.js';
import noPageScrollbarMixin from '@/utils/noPageScrollbarMixin.js';
import { mapActions, mapGetters } from 'vuex';
import PmuFormEmptyPreview from '@/components/pmu/PmuFormEmptyPreview.vue';
import downloadjs from 'downloadjs';

export default {
  name: 'FormTemplatePreviewModal',
  data: () => ({
    imagePreviewBase64: '',
    loadingPdf: false
  }),
  mixins: [noPageScrollbarMixin],
  props: ['tenantSlug', 'templateId'],
  components: {
    Close,
    DownloadIcon,
    ShareIcon,
    PmuFormEmptyPreview
  },
  computed: {
    ...mapGetters('formTemplate', ['currentTemplateGet']),
    createdAt() {
      return formatDateTime(this.currentTemplateGet.createdAt);
    },
    previewFileName() {
      return `${this.currentTemplateGet.name}.pdf`;
    }
  },
  methods: {
    ...mapActions('tenant', ['pmuEmptyPdfPreview']),
    previewFlow() {
      this.$router.push({ name: 'PmuSignFlowMock' });
    },
    shareDataUrl,
    isShareApiSupported,
    urlToFile,
    async downloadFile() {
      this.loadingPdf = true;
      const pdfPreviewBase64 = await this.pmuEmptyPdfPreview({
        params: {
          tenantSlug: this.tenantSlug,
          templateId: this.templateId
        }
      }).catch(error => {
        console.log(error);
        alert('Something went wrong in generating PDF preview');
      });
      this.loadingPdf = false;

      if (pdfPreviewBase64) {
        downloadjs(pdfPreviewBase64, this.previewFileName, 'application/pdf');
      }
    },
    closeModal() {
      this.$emit('close');
    },
    remove() {
      this.$emit('remove', this.file);
      this.closeModal();
    }
  }
};
</script>

<style scoped>
.content-gradient {
  background: linear-gradient(180deg, transparent 0%, #000000 93.85%);
}
.cta-wrapper {
  transform: translateY(-50%) translateX(28px);
}
</style>