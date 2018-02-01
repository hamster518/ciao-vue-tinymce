<template>
  <div ciao-vue-tinymce>
    <PhotoUpload
      ref="photoUpload"
      @update="update"
      :formDataFilename="formDataFilename"
      :photoUploadTag="photoUploadTag"
      :photoUploadRequest="photoUploadRequest"
      :hasFileBrowser="hasFileBrowser"
    />

    <textarea :id="uuid" :tinymce="uuid"></textarea>
  </div>
</template>

<script>
import 'tinymce/tinymce'
import 'tinymce/themes/modern/theme'
import 'tinymce/plugins/paste/plugin'
import 'tinymce/plugins/link/plugin'
import 'tinymce/plugins/autoresize/plugin'
import 'tinymce/plugins/image/plugin'
import 'tinymce/plugins/media/plugin'
import 'tinymce/plugins/table/plugin'
import 'tinymce/plugins/code/plugin'
import 'tinymce/plugins/advlist/plugin'
import 'tinymce/plugins/autolink/plugin'
import 'tinymce/plugins/lists/plugin'
import 'tinymce/plugins/charmap/plugin'
import 'tinymce/plugins/print/plugin'
import 'tinymce/plugins/preview/plugin'
import 'tinymce/plugins/anchor/plugin'
import 'tinymce/plugins/searchreplace/plugin'
import 'tinymce/plugins/visualblocks/plugin'
import 'tinymce/plugins/fullscreen/plugin'
import 'tinymce/plugins/insertdatetime/plugin'
import 'tinymce/plugins/contextmenu/plugin'
import 'tinymce/plugins/textcolor/plugin'
import uuidV4 from 'uuid/v4'
import PhotoUpload from './PhotoUpload/PhotoUpload.vue'
import $ from 'jquery'
export default {
  props: {
    language: {
      type: String,
      default: '',
    },
    language_url: {
      type: String,
      default: '',
    },
    photoUploadRequest: {
      type: Function,
      default: null,
    },
    photoUploadTag: {
      type: Function,
      default: null,
    },
    value: {
      type: String,
      default: '',
    },
    tools: {
      type: Array,
      default: null,
    },
    formDataFilename: {
      type: String,
      default: 'file',
    },
    config: {
      type: Object,
      default: () => {},
    },
  },
  data: function () {
    return {
      editor: null
    }
  },
  mounted: function() {
    this.init()
  },
  methods: {
    getDefaultConfig: function () {
      return {
        selector: this.editorSelector,
        skin: false,
        min_height: 300,
        language: this.language,
        language_url: this.language_url,
        destroy: true,
        plugins: [
          'advlist autolink lists link image charmap print preview anchor',
          'searchreplace visualblocks code fullscreen',
          'insertdatetime media table contextmenu paste code table',
          'textcolor',
        ],
        toolbar: this.toolbar,
        menubar: false,
        setup: editor => {
          this.editor = editor
          editor.on('blur', () => {
            this.$emit('input', editor.getContent())
            this.$emit('blur', editor.getContent())
          })

          editor.on('change', () => {
            this.$emit('change', editor.getContent())
          })

          if(this.hasFileBrowser) this.$refs.photoUpload.addTool(editor)
          this.setupCustomToolbar()
        }
      }
    },
    destroy: function() {
      tinymce.remove(this.editorSelector)
    },
    setup: function() {
      let config = $.extend({}, this.getDefaultConfig())
      for(const property in this.config) {
        config[property] = this.config[property]
      }
      tinymce.init(config)
    },
    init: function() {
      this.destroy()
      this.setup()
      this.setContent()
    },
    update: function () {
      this.$emit('input', this.editor.getContent())
    },
    setupCustomToolbar: function() {
      if(!this.tools) return

      for(const tool of this.tools) {
        this.editor.addButton(tool.text, {
          icon: tool.icon,
          onclick: () => tool.onclick(this.editor),
        })
      }
    },
    setContent: function () {
      if(!this.value) return
      tinymce.get(this.uuid).setContent(this.value)
    }
  },
  computed: {
    uuid: function() {
      return uuidV4()
    },
    editorSelector: function() {
      return `textarea[tinymce=${this.uuid}]`
    },
    hasFileBrowser: function () {
      return !!this.photoUploadRequest
    },
    toolbar: function () {
      const default_toolbar = 'code | undo redo | insert | styleselect | forecolor backcolor | bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | table | link image | fullscreen | photoUploadButton'
      if(!this.tools) return default_toolbar

      let custom = ''
      for(const tool of this.tools)
        custom += tool.text
      return `${default_toolbar} | ${custom}`
    },
  },
  watch: {
    value: function() {
      this.setContent()
    },
    language: function() {
      this.init()
    },
  },
  components: {
    PhotoUpload,
  },
}
</script>

<style lang="sass" type="text/sass" scoped>
  div[ciao-vue-tinymce]
</style>