<template>
  <div class="container" :class="currentClass">
    <h2>
      <thousand-text :amount="amount" />
    </h2>
    <ArrowIcon class="icon" />
  </div>
</template>

<script>
import ThousandText from './ThousandText.vue'
import ArrowIcon from './assets/IconArrowDown.svg?component'

export default {
  name: 'LastPrice',

  components: { ThousandText, ArrowIcon },

  props: {
    amount: {
      type: Number,
      required: true,
    },
  },

  data() {
    return {
      currentClass: 'buy',
    }
  },
}
</script>

<style lang="scss" scoped>
.container {
  margin: 6px 0px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon {
  margin-left: 6px;
}

.sell {
  background-color: var(--sell-quote-accumulative-total-size-bar-color);
  color: var(--sell-quote-price-text-color);
}
.buy {
  background-color: var(--buy-quote-accumulative-total-size-bar-color);
  color: var(--buy-quote-price-text-color);

  .icon {
    transform: rotate(180deg);
  }
}
</style>
