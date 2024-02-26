<template>
  <div>
    <h3 class="title">Order Book</h3>

    <table class="sell">
      <thead>
        <tr>
          <th class="table-header price">Price (USD)</th>
          <th class="table-header size">Size</th>
          <th class="table-header total">Total</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in sellData" :key="item.id /* TODO 這個還不確定 pk 是什麼 */">
          <td class="price">
            <thousand-text :amount="item.prize" />
          </td>
          <td class="size">
            <thousand-text :amount="item.size" />
          </td>
          <td class="total">
            <total-bar :value="item.total" :total="total___" type="sell" />
          </td>
        </tr>
      </tbody>
    </table>

    <last-price :amount="amount" />

    <!-- TODO 這個應該可以整理成一個? -->
    <table class="buy">
      <tbody>
        <tr v-for="item in buyData" :key="item.id /* TODO 這個還不確定 pk 是什麼 */">
          <td class="price">
            <thousand-text :amount="item.prize" />
          </td>
          <td class="size">
            <thousand-text :amount="item.size" />
          </td>
          <td class="total">
            <total-bar :value="item.total" :total="total___" type="buy" />
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import LastPrice from './LastPrice.vue'
import ThousandText from './ThousandText.vue'
import TotalBar from './TotalBar.vue'

export default {
  components: { LastPrice, ThousandText, TotalBar },

  data() {
    return {
      amount: 123123,
      total___: 123123123,

      sellData: [],
      buyData: [],
    }
  },

  created() {
    this.sellData = [...Array(8)].map(() => ({ prize: 123123, size: 123, total: 123 }))
    this.buyData = [...Array(8)].map(() => ({ prize: 123123, size: 123123, total: 123 }))
  },
}
</script>

<style lang="scss" scoped>
.title {
  color: var(--default-text-color);
  padding: 12px 12px;
  border-bottom-width: 1px;
  border-bottom-style: solid;
  border-bottom-color: var(--default-text-color-for-line);
}

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
</style>
