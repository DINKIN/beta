<template>
  <div>
    <slot
      :ok="ok"
      :cancel="cancel"
    />
  </div>
</template>
<script>
import mediator from './mediator'

export default {
  data() {
    return {
      resolve: null,
      reject: null
    }
  },
  mounted() {
    mediator.$set(mediator.modals, this.$el.id, this)
  },
  methods: {
    show(arg) {
      this.$emit('show', arg)
      return new Promise((resolve, reject) => {
        this.resolve = resolve
        this.reject = reject
      })
    },
    ok(arg) {
      if (!this.resolve) return false
      this.resolve(arg)
      this.hide(arg)
      return true
    },
    cancel(arg) {
      if (!this.reject) return false
      this.reject(arg)
      this.hide(arg)
      return true
    },
    hide(arg) {
      if (this.resolve || this.reject) this.$emit('hide', arg)
      this.resolve = null
      this.reject = null
    }
  }
}
</script>
