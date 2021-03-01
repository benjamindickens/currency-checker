<template>
  <div class="app-wrapper">
    <div class="currency-input">
      <label for="currencyInput" class="currency-input__label"
        >Введите название валюты:
      </label>
      <div class="currency-input__container">
        <input
          type="text"
          class="currency-input__input-box"
          id="currencyInput"
          v-model="ticker"
          @keydown.enter="add"
        />
        <div v-if="ticker" class="currency-input__currency-helper" @click="add">
          <div type="text" class="currency-input__auto-input" @click="ticker = 'BTC'">BTC</div>
        </div>
      </div>
      <button
        class="js-input-submit currency-input__submit main-btn"
        @click="add"
      >
        + Добавить
      </button>
    </div>
    <div class="currency-output" v-if="tickers.length > 0">
      <div
        class="currency-output__item output-item"
        :class="{ '--active': t.name === currentGraph }"
        v-for="t of tickers"
        :key="t"
        @click="updateGraph(t)"
      >
        <div class="output-item__name">{{ t.name }} - USD</div>
        <div class="output-item__value">{{ t.value }}</div>
        <button
          class="js-remove-currency main-btn"
          @click.stop="removeCurrency(t)"
        >
          Удалить
        </button>
      </div>
    </div>
    <div v-if="currentGraph" class="currency-graph">
      <div class="currency-graph__interface">
        <div class="currency-graph__name">{{ currentGraph }}</div>
        <button
          class="close-btn currency-graph__close-btn"
          @click="currentGraph = null"
        ></button>
      </div>
      <div class="currency-graph__container">
        <div
          v-for="(column, index) in normalizedData()"
          :key="index"
          :style="{ height: column + '%' }"
          class="currency-graph__column"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      ticker: "",
      tickers: [],
      currentGraph: null,
      graphData: []
    };
  },
  methods: {
    add() {
      const currentTicker = {
        name: this.ticker,
        value: "-"
      };
      this.tickers.push(currentTicker);
      const currentObject = this.tickers[this.tickers.length - 1];
      // для работы во вью 3 необходимо пониматьч то работа с объектами происзоит через прокси метод тоесть все объекты и массивы что
      // мы храним внутри нашшей иснтанс получает спец оболочку в виде прокси и все что мы в них помещаем по сути своей не являються теми же самими эллементами что были до перемещения
      const url = `https://min-api.cryptocompare.com/data/price?fsym=${currentTicker.name}&tsyms=USD&api_key=ba1a87d6f1a69c9acec2fea53cbfe02026e010dffbbcad9953202825578bbadd`;
      const dataUpdate = setInterval(async () => {
        await fetch(url)
          .then(response => response.json())
          .then(data => {
            currentObject.value =
              data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(1);
            // this.tickers.find(t => t.name === currentTicker.name).value =
            //   data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2);
            if (this.currentGraph === currentTicker.name) {
              this.graphData.push(data.USD);
            }
          })
          .catch(err => {
            this.tickers = this.tickers.filter(
              t => t.name !== currentTicker.name
            );
            console.log(err);
            clearInterval(dataUpdate);
            if (currentTicker.name === this.currentGraph) {
              this.currentGraph = null;
            }
          });
      }, 10000);
      this.ticker = "";
    },
    removeCurrency(currentBox) {
      this.tickers = this.tickers.filter(t => t !== currentBox);
      if (currentBox.name === this.currentGraph?.name) {
        this.currentGraph = null;
      }
    },
    normalizedData() {
      const minPrice = Math.min(...this.graphData);
      const maxPrice = Math.max(...this.graphData);
      const newArray = this.graphData.map(column => {
        const first = (column - minPrice) * 100;
        const second = maxPrice - minPrice;
        return first / second;
      });
      return newArray;
    },
    updateGraph(innerData) {
      if (this.currentGraph !== innerData.name) {
        this.currentGraph = innerData.name;
        this.graphData = [];
      }
    }
  },
  watch: {
    ticker() {
      this.ticker = this.ticker.toUpperCase();
    }
  }
};
</script>
<style lang="scss">
* {
  font-size: 2rem;
}
.currency-input {
  justify-content: start;
  text-align: center;
  display: grid;
  grid-gap: 1rem;

  &__container {
    display: grid;
    grid-template-columns: 25rem;
    border-bottom: 1px solid lightgray;
    padding-bottom: 0.2rem;
  }

  &__currency-helper {
    padding: 0.5rem 0 0.3rem;
    display: grid;
    grid-auto-flow: column;
    grid-gap: 0.2rem;
  }

  &__auto-input {
    text-align: center;
    border: 1px solid darkgray;
    color: royalblue;
    border-radius: 5px;
    padding: 0.2rem 0.4rem;
  }

  &__input-box {
    outline: none;
    border: 2px solid transparent;
    border-radius: 5px;
    background: lightgray;
    text-align: center;
    padding: 1rem 0.5rem;

    &:focus {
      border-color: royalblue;
    }
  }
}

.currency-output {
  border-top: 1px solid black;
  border-bottom: 1px solid black;
  padding: 4rem 2rem;
  display: grid;
  grid-gap: 3rem;
  width: 100%;
  grid-template-columns: repeat(auto-fit, 35rem);
  justify-content: center;
}

.output-item {
  min-height: 12rem;
  background: rgba(152, 226, 255, 0.575);
  padding: 2rem 3rem;
  border-radius: 5px;
  display: grid;
  border: 2px solid transparent;
  text-align: center;
  grid-gap: 1rem;

  &__name {
    font-size: 3.5rem;
    font-weight: 900;
  }

  &.--active {
    border-color: royalblue;
    background: white;
  }
}

.currency-graph {
  display: grid;
  grid-template-rows: auto 1fr;
  grid-gap: 2rem;
  &__container {
    height: 28rem;
    display: grid;
    background: white;
    grid-template-columns: repeat(auto-fit, minmax(0.001rem, 1fr));
    align-items: end;
    border-left: 2px solid lightgray;
    border-bottom: 2px solid lightgray;
  }
  &__column {
    background: royalblue;
  }

  &__interface {
    display: flex;
    justify-content: space-between;
  }
}
</style>
