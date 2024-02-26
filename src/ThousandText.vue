<template>
  <span>{{ computedText }}</span>
</template>

<script>
export default {
  name: 'ThousandText',

  props: {
    amount: {
      type: [String, Number],
      required: true,
    },
  },

  computed: {
    computedText() {
      return Number(this.amount).toLocaleString()
    },
  },
}
</script>
