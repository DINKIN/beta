<template>
  <component
    :is="element"
    v-bind="$attrs"
  >
    <template v-for="(text, i) in modifiedText">
      <template v-if="text.type === 'text'">{{ text.value }}</template>
      <img
        v-else
        :key="`emoji-${i}`"
        :src="`https://twemoji.maxcdn.com/2/72x72/${toCodePoint(text.value)}.png`"
        :alt="text.value"
        class="emoji"
      >
    </template>
  </component>
</template>
<script>
import twemoji from 'twemoji'
import emojiRegex from 'emoji-regex/es2015'

export default {
  name: 'Emojify',
  props: {
    element: {
      type: [String, Object],
      default: 'span'
    },
    text: {
      type: String,
      default: '',
      required: true
    }
  },
  computed: {
    modifiedText() {
      const text = this.text
      const regex = emojiRegex()
      let match,
        ep = 0
      const res = []
      while ((match = regex.exec(text))) {
        const emoji = match[0]
        if (ep !== match.index)
          res.push({
            type: 'text',
            value: text.substring(ep, match.index)
          })
        res.push({
          type: 'emoji',
          value: emoji
        })
        ep = match.index + emoji.length
      }
      if (ep !== text.length)
        res.push({
          type: 'text',
          value: text.substring(ep, text.length)
        })
      return res.filter(obj => obj.value)
    }
  },
  methods: {
    toCodePoint: twemoji.convert.toCodePoint
  }
}
</script>
`
})
