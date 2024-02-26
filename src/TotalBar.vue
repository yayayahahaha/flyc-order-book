<template>
  <div class="container">
    <div class="bar" :class="type" :style="{ width: `${width}%` }"></div>
    <thousand-text :amount="value" :no-zero="true" />
  </div>
</template>

<script>
import ThousandText from './ThousandText.vue'

const SELL_TEXT = 'sell'
const BUY_TEXT = 'buy'

export default {
  name: 'TotalBar',

  components: { ThousandText },

  props: {
    value: {
      type: Number,
      required: true,
    },

    total: {
      type: Number,
      required: true,
    },

    type: {
      validator: (value) => [SELL_TEXT, BUY_TEXT].includes(value),
      default: SELL_TEXT,
    },
  },

  computed: {
    width() {
      return (100 * this.value) / this.total
    },
  },
}
</script>

<style lang="scss" scoped>
.container {
  position: relative;

  .bar {
    position: absolute;
    right: 0px;
    top: 1px;
    bottom: 1px;

    z-index: -1;

    &.sell {
      width: 50%;
      background-color: var(--sell-quote-accumulative-total-size-bar-color);
    }
    &.buy {
      width: 50%;
      background-color: var(--buy-quote-accumulative-total-size-bar-color);
    }
  }
}
</style>
