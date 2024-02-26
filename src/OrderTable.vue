<template>
  <table :class="type">
    <thead>
      <tr>
        <th class="table-header price">Price (USD)</th>
        <th class="table-header size">Size</th>
        <th class="table-header total">Total</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in data" :key="item.key">
        <td class="price">
          <thousand-text :amount="item.price" />
        </td>
        <td class="size" :class="item.class">
          <thousand-text :amount="item.size" />
        </td>
        <td class="total">
          <total-bar :value="item.total" :total="total" :type="type" />
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import ThousandText from './ThousandText.vue'
import TotalBar from './TotalBar.vue'

export default {
  name: 'OrderTable',

  components: { ThousandText, TotalBar },

  props: {
    data: {
      type: Array,
      required: true,
    },

    total: {
      type: Number,
      required: true,
    },

    type: {
      validator: (value) => ['sell', 'buy'].includes(value),
      required: true,
    },
  },

  data() {
    return {}
  },

  computed: {},

  watch: {},

  created() {},

  methods: {},
}
</script>

<style lang="scss" scoped>
table {
  padding: 0px 6px;
  width: 100%;

  .table-header {
    font-weight: 400;
    color: var(--quote-table-head-text-color);
  }

  tbody {
    font-weight: bold;
    color: var(--default-text-color);
  }

  .price {
    width: 30%;
    text-align: left;
  }
  .size {
    width: 20%;
    text-align: right;
  }
  .total {
    width: 50%;
    text-align: right;
  }

  &.sell {
    tbody {
      .price {
        color: var(--sell-quote-price-text-color);
      }
    }
  }

  &.buy {
    tbody {
      .price {
        color: var(--buy-quote-price-text-color);
      }
    }
  }
}

.fade-out-red {
  animation-duration: 0.3s;
  animation-name: fade_out_red;
  animation-fill-mode: forwards;
}
.fade-out-green {
  animation-duration: 0.3s;
  animation-name: fade_out_green;
  animation-fill-mode: forwards;
}

@keyframes fade_out_red {
  0% {
    background-color: var(--animation-flash-red-background-color);
  }
  100% {
    background-color: transparent;
  }
}
@keyframes fade_out_green {
  0% {
    background-color: var(--animation-flash-green-background-color);
  }
  100% {
    background-color: transparent;
  }
}
</style>
