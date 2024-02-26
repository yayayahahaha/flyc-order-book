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

  mounted() {
    this.initLastPriceSocket()
  },

  methods: {
    initLastPriceSocket() {
      const orderbookSocket = new WebSocket('wss://ws.btse.com/ws/futures')

      // Connection opened
      orderbookSocket.addEventListener('open', () => {
        orderbookSocket.send(
          JSON.stringify({
            op: 'subscribe',
            args: ['tradeHistoryApi:BTCPFC'],
          })
        )
      })

      // Listen for messages
      orderbookSocket.addEventListener('message', (event) => {
        console.log('Message from server ', event.data)
      })
    },
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

<!-- reference -->
<!-- https://medium.com/@alekswebnet/import-svg-to-vite-vue-projects-8168826a76c1 -->
