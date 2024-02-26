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

    noZero: {
      type: Boolean,
      default: false,
    },
  },

  computed: {
    computedText() {
      const numberString = Number(this.amount).toLocaleString()
      if (this.noZero) return numberString

      return /\.\d$/.test(numberString) ? numberString : `${numberString}.0`
    },
  },
}
</script>
