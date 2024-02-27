<template>
  <table :class="type">
    <thead v-if="showHeaders">
      <tr>
        <th class="table-header price pl-5">Price (USD)</th>
        <th class="table-header size pr-5">Size</th>
        <th class="table-header total pr-5">Total</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in data" :key="item.key" class="row" :class="item.class">
        <td class="price pl-5">
          <thousand-text :amount="item.price" />
        </td>
        <td class="size pr-5" :class="item.sizeClass">
          <thousand-text :amount="item.size" :no-zero="true" />
        </td>
        <td class="total pr-5">
          <total-bar :value="item.total" :total="total" :type="type" />
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import ThousandText from './ThousandText.vue'
import TotalBar from './TotalBar.vue'

const SELL_TEXT = 'sell'
const BUY_TEXT = 'buy'

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
      validator: (value) => [SELL_TEXT, BUY_TEXT].includes(value),
      required: true,
    },
  },

  computed: {
    showHeaders() {
      return this.type === SELL_TEXT
    },
  },
}
</script>

<style lang="scss" scoped>
table {
  width: 100%;
  border-spacing: 0;

  .pl-5 {
    padding-left: 5px;
  }
  .pr-5 {
    padding-right: 5px;
  }

  tr {
    margin: 2px 0px;
  }
  td {
    line-height: 1.5;
  }

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

    .row {
      &:hover {
        background-color: var(--animation-flash-red-background-color) !important; // to avoid animate forwards behavior.
      }
    }
  }

  &.buy {
    tbody {
      .price {
        color: var(--buy-quote-price-text-color);
      }
    }

    .row {
      &:hover {
        background-color: var(
          --animation-flash-green-background-color
        ) !important; // to avoid animate forwards behavior.
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

<!-- reference -->
<!-- https://www.w3schools.com/html/html_table_padding_spacing.asp -->
