<template>
  <section>
    <div class="flex">
      <div class="max-w-xs">
        <label
          for="wallet"
          class="block text-sm font-medium text-gray-700"
          >Тикер</label
        >
        <div class="mt-1 relative rounded-md shadow-md">
          <input
            autocomplete="off"
            @focus="modal = true"
            @input="filterCoins"
            v-model="ticker"
            @keydown.enter="add"
            type="text"
            name="wallet"
            id="wallet"
            class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
            placeholder="Например DOGE"
          />
        </div>
        <div
          v-if="filteredCoinList && modal && ticker.length"
          class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
        >
          <span
            v-for="coin in filteredCoinList.splice(0, 4)"
            :key="coin"
            @click="setCoin(coin)"
            class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
          >
            {{ coin }}
          </span>
        </div>

        <div
          v-if="dupplicateTicker"
          class="text-sm text-red-600"
        >
          Такой тикер уже добавлен
        </div>
      </div>
    </div>
    <add-button
      :disabled="disabled"
      @click="add"
      type="button"
    />
  </section>
</template>
<script>
import AddButton from './AddButton.vue';
export default {
  components: {
    AddButton,
  },
  created() {
    this.getCurrencyName();
  },

  data() {
    return {
      ticker: '',
      dupplicateTicker: false,
      coinList: [],
      filteredCoinList: [],
      modal: false,
    };
  },

  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  emits: {
    'add-ticker': (value) => typeof value === 'string' && value.length > 0,
  },
  methods: {
    add() {
      if (this.ticker.length === 0) {
        return;
      }

      const tickersData = JSON.parse(
        localStorage.getItem('cryptonomicon-list')
      );
      for (let tickerData of tickersData) {
        if (this.ticker === tickerData.name) {
          this.dupplicateTicker = true;
          return;
        }
      }
      this.$emit('add-ticker', this.ticker);
      this.ticker = '';
    },

    filterCoins() {
      this.filteredCoinList = this.coinList.filter((coin) => {
        return coin.toUpperCase().startsWith(this.ticker.toUpperCase());
      });
      this.ticker = this.ticker.toUpperCase();
    },

    setCoin(coin) {
      this.ticker = coin;
      this.modal = false;
    },

    getCurrencyName() {
      fetch('https://min-api.cryptocompare.com/data/all/coinlist?summary=true')
        .then((response) => response.json())
        .then((coinList) => {
          for (let coinName in coinList.Data) this.coinList.push(coinName);
        });
    },
  },

  watch: {
    ticker() {
      this.dupplicateTicker = false;
    },
  },
};
</script>
