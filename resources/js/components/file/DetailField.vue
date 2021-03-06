<template>
  <r64-panel-item
    :hide-field="hideField"
    :field="field"
    :hide-label="hideLabelInDetail"
    :label-classes="panelLabelClasses"
    :field-classes="panelFieldClasses"
    :wrapper-classes="panelWrapperClasses"
  >
    <div slot="value">
      <template v-if="shouldShowLoader">
        <ImageLoader
          :src="field.thumbnailUrl"
          class="max-w-xs"
          @missing="(value) => missing = value"
        />
      </template>

      <template v-if="displayValue && !field.thumbnailUrl">
        {{ displayValue }}
      </template>

      <span v-if="!value && !field.value && !field.thumbnailUrl">&mdash;</span>
      <span v-if="deleted">&mdash;</span>

      <portal to="modals">
        <transition name="fade">
          <confirm-upload-removal-modal
            v-if="removeModalOpen"
            @confirm="removeFile"
            @close="closeRemoveModal"
          />
        </transition>
      </portal>

      <p
        v-if="shouldShowToolbar"
        class="flex items-center text-sm mt-3"
      >
        <a
          v-if="isDownloadable"
          :dusk="field.attribute + '-download-link'"
          @keydown.enter.prevent="download"
          @click.prevent="download"
          tabindex="0"
          class="cursor-pointer dim btn btn-link text-primary inline-flex items-center"
        >
          <icon class="mr-2" type="download" view-box="0 0 24 24" width="16" height="16" />
          <span class="class mt-1">
            Download
          </span>
        </a>
      </p>
    </div>
  </r64-panel-item>
</template>

<script>
import ImageLoader from '../../nova/ImageLoader'
import R64Field from '../../mixins/R64Field'

export default {
  mixins: [R64Field],

  props: ['resource', 'resourceName', 'resourceId', 'field', 'value'],

  components: { ImageLoader },

  data: () => ({ removeModalOpen: false, missing: false, deleted: false }),

  methods: {
    /**
     * Download the linked file
     */
    download() {
      const { resourceName, resourceId } = this
      const attribute = this.field.attribute

      const href = this.parentAttribute ?
                  `/nova-r64-api/${resourceName}/${resourceId}/download/${attribute}?row=${this.parentAttribute}&value=${this.displayValue}` :
                  `/nova-api/${resourceName}/${resourceId}/download/${attribute}`

      let link = document.createElement('a')
      link.href = href
      link.download = 'download'
      link.click()
    },

    /**
     * Confirm removal of the linked file
     */
    confirmRemoval() {
      this.removeModalOpen = true
    },

    /**
     * Close the upload removal modal
     */
    closeRemoveModal() {
      this.removeModalOpen = false
    },

    /**
     * Remove the linked file from storage
     */
    async removeFile() {
      const { resourceName, resourceId } = this
      const attribute = this.field.attribute

      try {
        await Nova.request().delete(
          `/nova-api/${resourceName}/${resourceId}/field/${attribute}`
        )
        this.closeRemoveModal()
        this.deleted = true
      } catch (error) {
        this.closeRemoveModal()
      }
    }
  },

  computed: {
    hasValue() {
      return (
        Boolean(this.value || this.field.value || this.field.thumbnailUrl) &&
        !Boolean(this.deleted) &&
        !Boolean(this.missing)
      )
    },

    shouldShowLoader() {
      return !Boolean(this.deleted) && Boolean(this.field.thumbnailUrl)
    },

    shouldShowToolbar() {
      return (
        Boolean(this.isDownloadable || this.field.deletable) &&
        this.hasValue
      )
    },

    isDownloadable() {
      return this.parentAttribute ? true : this.field.downloadable
    },

    displayValue() {
      return this.value || this.field.value
    }
  }
}
</script>
