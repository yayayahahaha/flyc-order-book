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
      const numberString = Number(this.amount).toLocaleString()
      return /\.\d$/.test(numberString) ? numberString : `${numberString}.0`
    },
  },
}
</script>
