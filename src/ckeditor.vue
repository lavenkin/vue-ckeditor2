<template>
    <div class="ckeditor">
        <textarea
          :name="name"
          :id="id"
          :value="value"
          :types="types"
          :config="config">
        </textarea>
    </div>
</template>

<script>
let inc = 0

export default {
  name: 'vue-ckeditor',
  props: {
    name: {
      type: String,
      default: () => `editor-${++inc}`
    },
    value: {
      type: String
    },
    id: {
      type: String,
      default: () => `editor-${inc}`
    },
    types: {
      type: String,
      default: () => `classic`
    },
    config: {
      type: Object,
      default: () => {}
    }
  },
  data () {
    return { destroyed: false }
  },
  computed: {
    instance () {
      return CKEDITOR.instances[this.id]
    }
  },
  watch: {
    value (val) {
      let html = this.instance.getData()
      if (val !== html) {
        this.instance.setData(val, null, true)
      }
    }
  },
  mounted () {
    if (typeof CKEDITOR === 'undefined') {
      console.log('CKEDITOR is missing (http://ckeditor.com/)')
    } else {
      if (this.types === 'inline') {
          CKEDITOR.inline(this.id, this.config)
      } else {
          CKEDITOR.replace(this.id, this.config)
      }

      this.instance.on('change', () => {
        let html = this.instance.getData()
        if (html !== this.value) {
          this.$emit('input', html)
          this.$emit('update:value', html)
        }
      })

      this.instance.on('blur', () => {
        this.$emit('blur', this.instance)
      })

      this.instance.on('focus', () => {
        this.$emit('focus', this.instance)
      })
    }
  },
  beforeDestroy () {
    if (!this.destroyed) {
      this.instance.focusManager.blur(true)
      this.instance.removeAllListeners()
      this.instance.destroy()
      this.destroyed = true
    }
  }
}
</script>
<style>
.ckeditor::after {
  content: "";
  display: table;
  clear: both;
}
</style>
