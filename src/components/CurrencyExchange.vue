<template>
  <div class="main">
    <div class="converter-popup">
      <header>
        <div class="header-block"><span>Currency converter</span></div>
        <div class="header-block logo">
          <img class="baby" src="../assets/logo.svg" />
        </div>
        <div class="header-block"><span>Using Monobank API</span></div>
      </header>
      <div class="converter-content">
        <form class="converter" @reset="reset">
          <div class="customer-data">
            <label for="amount">Type an amount </label>
            <input
              id="amount"
              name="amount"
              type="number"
              placeholder="1000"
              step="0.01"
              min="1"
              required
              v-model="amount"
              @change="reportValidity($event)"
            />
          </div>
          <div class="customer-data">
            <label for="currency-from">Choose currency FROM:</label>
            <select
              name="currency-from"
              id="currency-from"
              required
              v-model="currencyFrom"
              ref="currencyFrom"
            >
              <option value="" selected>From</option>
              <option
                v-bind:value="currency"
                v-for="currency in availableCurrencies"
                :key="currency.AlphabeticCode"
                >{{ currency.AlphabeticCode }} - {{ currency.Currency }}
              </option>
            </select>
          </div>
          <div class="change-currency-order" @click="switchCurrencyOrder">
            <img src="../assets/refresh.png" alt="switch-currency-order" />
          </div>
          <div class="customer-data">
            <label for="currency-to">Choose currency TO:</label>
            <select
              name="currency-to"
              id="currency-to"
              required
              v-model="currencyTo"
              ref="currencyTo"
            >
              <option value="" selected>To</option>
              <option
                v-bind:value="currency"
                v-for="currency in availableCurrencies"
                :key="currency.AlphabeticCode"
                >{{ currency.AlphabeticCode }} - {{ currency.Currency }}
              </option>
            </select>
          </div>
          <div class="button reset">
            <button type="reset">Reset</button>
          </div>
        </form>
        <div class="result">
          <transition name="fade">
            <p class="result-amount" v-if="amount">
              <span>{{ amount > 0 ? amount : null }} </span>
              <span v-if="amount > 0">{{ currencyFrom.AlphabeticCode }}</span>
              <span v-if="result && amount > 0 && currencyFrom">
                = {{ result }} {{ currencyTo.AlphabeticCode }}</span
              >
            </p>
          </transition>
          <transition name="fade">
            <p class="result-amount one-amount-rate" v-if="result">
              <span>1 </span>
              <span>{{ currencyFrom.AlphabeticCode }}</span>
              <span>
                = {{ uahRate.toFixed(2) }} {{ currencyTo.AlphabeticCode }}</span
              >
            </p>
          </transition>
        </div>
      </div>
      <div class="mask"></div>
    </div>
    <ErrorPopup
      :headerTitle="headerTitle"
      :message="errorMessage"
      v-if="reloadPage"
    ></ErrorPopup>
  </div>
</template>
<script>
const URL = "https://api.monobank.ua/bank/currency";
import availableCurrencies from "@/assets/currencies.json";
import ErrorPopup from "./ErrorPopup";

export default {
  name: "CurrencyExchange",
  components: {
    ErrorPopup
  },
  data() {
    return {
      monoCurrencies: "",
      availableCurrencies,
      uahCurrencyCode: 980,
      uahRate: "",
      reloadPage: false,
      amount: "",
      currencyFrom: "",
      currencyTo: "",
      errorMessage:
        "Sorry for the inconvenience! Currently we are observing some errors on the server side. " +
        "Please try to reload the page. Thank you!",
      headerTitle: "Internal Server Error"
    };
  },

  async beforeMount() {
    this.monoCurrencies = await this.getCurrencies(URL);
    // eslint-disable-next-line
    //debugger;

    if (this.monoCurrencies) {
      this.addToLocalStorage(this.monoCurrencies);
    } else if (this.getDataFromLocaleStorage() === null) {
      this.reloadPage = true;
    } else {
      this.monoCurrencies = this.getDataFromLocaleStorage();
    }
  },

  computed: {
    result() {
      // eslint-disable-next-line
      //debugger;

      if (
        !this.amount ||
        !this.currencyFrom.NumericCode ||
        !this.currencyTo.NumericCode
      ) {
        return;
      }

      const amount = parseFloat(this.amount);
      const currencyFromCode = parseInt(this.currencyFrom.NumericCode);
      const currencyToCode = parseInt(this.currencyTo.NumericCode);

      let currencyFromExchangeRate = "";
      let currencyToExchangeRate = "";
      let uahExchangeData = "";
      //let uahRate = "";

      //help to know exchange type
      // UAH to ANY (returns true) or
      // ANY to UAH (returns false)
      let isExchangeFromUah = currencyFromCode === this.uahCurrencyCode;

      //Exchange UAH
      if (
        (currencyFromCode === this.uahCurrencyCode ||
          currencyToCode === this.uahCurrencyCode) &&
        currencyToCode !== currencyFromCode
      ) {
        uahExchangeData = this.mono_findUahCurrencyRateObj(
          this.monoCurrencies,
          { currencyFromCode, currencyToCode }
        );
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        this.uahRate = this.mono_getUahRate(uahExchangeData, isExchangeFromUah);
      } else if (
        currencyFromCode === currencyToCode &&
        currencyToCode === this.uahCurrencyCode
      ) {
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        this.uahRate = 1;
      } else {
        //Exchange not UAH
        currencyFromExchangeRate = this.mono_findMultiCurrencyRateObj(
          this.monoCurrencies,
          currencyFromCode
        );
        currencyToExchangeRate = this.mono_findMultiCurrencyRateObj(
          this.monoCurrencies,
          currencyToCode
        );
        // eslint-disable-next-line vue/no-side-effects-in-computed-properties
        this.uahRate = this.mono_getMultiCurrencyRate(
          currencyFromExchangeRate,
          currencyToExchangeRate
        );
      }

      console.log("test", amount, currencyFromCode, currencyToCode);
      console.log(currencyFromExchangeRate, currencyToExchangeRate);

      return (amount * this.uahRate).toFixed(2);
    }
  },

  errorCaptured() {
    // eslint-disable-next-line
    debugger;
  },

  methods: {
    async getCurrencies(url) {
      try {
        const response = await fetch(url, {
          method: "GET",
          credentials: "same-origin",
          headers: {
            "Content-Type": "application/json"
          }
        });

        if (response.status === 200) {
          return response.json();
        } else {
          return null;
        }
      } catch (error) {
        console?.error?.(error);
      }
    },

    reportValidity(event) {
      this.isValid = event.target.reportValidity();
    },

    reset(event) {
      event.currentTarget.reset();
      Object.assign(this, {
        amount: "",
        currencyFrom: "",
        currencyTo: ""
      });
    },

    //Exchange UAH
    mono_findUahCurrencyRateObj(arr, searchExchangeCodes) {
      let { currencyFromCode, currencyToCode } = searchExchangeCodes;
      let searchCode = "";

      //currencyCodeA => returns ANY currency, which is unique identifier
      //currencyCodeB => returns UAH currency, which is not unique identifier
      //from UAH to ANY => search by currencyToCode, which is unique
      if (currencyFromCode === this.uahCurrencyCode) {
        searchCode = currencyToCode;
      } else {
        //from ANY to UAH => search by currencyFromCode, which is unique
        searchCode = currencyFromCode;
      }

      return this.mono_findMultiCurrencyRateObj(arr, searchCode);
    },

    //Exchange not UAH
    mono_findMultiCurrencyRateObj(arr, searchCode) {
      return arr.find(item => {
        return (
          item.currencyCodeA === searchCode &&
          item.currencyCodeB === this.uahCurrencyCode
        );
      });
    },

    addToLocalStorage(data) {
      if (!data) {
        return;
      }
      let serialData = JSON.stringify(data);
      localStorage.setItem("monocurruncies", serialData);
    },

    getDataFromLocaleStorage() {
      let currencies = localStorage.getItem("monocurruncies");

      if (currencies) {
        return JSON.parse(currencies);
      } else {
        return null;
      }
    },

    //Exchange UAH
    mono_getUahRate(currencyRate, isExchangeFromUah) {
      let uahRate;
      let availableRate = currencyRate.rateSell ?? currencyRate.rateCross;

      if (isExchangeFromUah) {
        uahRate = 1 / availableRate;
      } else {
        uahRate = availableRate;
      }

      return uahRate;
    },

    //Exchange not UAH
    mono_getMultiCurrencyRate(currencyFrom, currencyTo) {
      let uahRate;

      //checking different cases when rateSell or rateCross are available or not
      uahRate =
        (currencyFrom.rateSell ?? currencyFrom.rateCross) /
        (currencyTo.rateSell ?? currencyTo.rateCross);

      return uahRate;
    },

    switchCurrencyOrder() {
      //swap currencies
      const { currencyFrom, currencyTo } = this.$refs;

      [currencyFrom.selectedIndex, currencyTo.selectedIndex] = [
        currencyTo.selectedIndex,
        currencyFrom.selectedIndex
      ];

      [this.currencyFrom, this.currencyTo] = [
        this.currencyTo,
        this.currencyFrom
      ];
    }
  }
};
</script>
<style lang="less">
//default/mobile styles
.main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  max-width: 330px;
  width: 100%;
  margin: auto;
  font-size: 14px;

  input,
  select,
  button {
    width: 205px;
    padding: 15px;
    font-size: 14px;
    border-radius: 16px;
    height: 50px;
    font-family: "Poppins", sans-serif;
  }

  input,
  select {
    border: 1px solid #af84b9;
  }

  select {
    appearance: none; /* Remove default arrow */
    background-image: url("../assets/arrow.png"); /* Add custom arrow */
    background-position: 180px center;
    background-repeat: no-repeat;
    background-size: 10px;

    &:required:invalid {
      color: gray;
    }

    option {
      color: black;
    }
    option[value=""][disabled] {
      display: none;
    }
  }

  label {
    display: block;
    margin: 5px;
  }

  .button {
    margin: 15px;
    align-self: flex-end;

    button {
      font-weight: 400;
      text-align: center;
      cursor: pointer;
      text-transform: uppercase;
      transition: box-shadow 0.3s;
      background-color: #fa5255;
      color: white;
      font-family: "Poppins", sans-serif;

      &:hover {
        -webkit-box-shadow: 0px 9px 21px -8px #9f99a2;
        -moz-box-shadow: 0px 9px 21px -8px #9f99a2;
        box-shadow: 0px 9px 21px -8px #9f99a2;
      }
    }
  }

  .change-currency-order {
    img {
      width: 25px;
    }
  }

  .converter-popup {
    width: 100%;
    border-radius: 32px;
    margin-top: 25px;

    header {
      display: flex;
      align-items: center;
      justify-content: space-around;
      flex-wrap: wrap;
      background-color: #f2f4f5;
      border-top-left-radius: 32px;
      border-top-right-radius: 32px;
      padding: 10px;

      .logo {
        img {
          max-width: 270px;
        }
      }
    }

    .converter-content {
      display: flex;
      flex-direction: column;
      justify-content: center;
      padding: 20px;
      font-weight: 200;
      position: relative;
      z-index: 1;
      background-color: white;
      border-bottom-left-radius: 32px;
      border-bottom-right-radius: 32px;

      .converter {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-wrap: wrap;

        .customer-data {
          margin: 15px;
        }

        .change-currency-order {
          flex-basis: 100%;
          margin: 0;
          img {
            transform: translate(-50%);
            margin-left: 50%;
          }
        }
      }

      .result {
        align-self: center;
        min-height: 112px;

        .result-amount {
          font-weight: 400;
          font-size: 21px;
          text-align: center;
          margin-bottom: 5px;
        }
        .one-amount-rate {
          margin: 5px;
          font-size: 14px;
          color: #ccc;
        }
      }
    }
  }

  .mask {
    position: relative;
    height: 100px;
    border-radius: 32px;

    &:before,
    &:after {
      position: absolute;
      content: "";
      height: 100%;
      width: 80%;
      background-color: rgba(31, 33, 43, 0.32);
    }

    &:before {
      margin-top: -50px;
      max-width: 830px;
      left: 55px;
      filter: blur(48px);
    }

    &:after {
      margin-top: -90px;
      max-width: 880px;
      left: 30px;
      filter: blur(32px);
    }
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
  opacity: 1;
}

.fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}

//tablet styles
@media (min-width: 768px) {
  .main {
    max-width: 700px;

    .change-currency-order {
      cursor: pointer;
    }

    .converter-popup {
      margin-top: 180px;
    }
  }
}

@media (min-width: 1024px) {
  .main {
    .converter-popup {
      .converter-content {
        .converter {
          .change-currency-order {
            flex-basis: 0;
            margin-top: 25px;

            img {
              transform: none;
              margin: 0;
            }
          }

          .reset {
            margin-left: 0;
          }
        }
      }
    }
    .mask {
      &:before,
      &:after {
        width: 100%;
      }
    }
  }
}

//desktop small screen styles
@media (min-width: 1024px) and (max-width: 1600px) {
  .main {
    max-width: 820px;

    input,
    select,
    button {
      width: 180px;
      padding: 10px;
      font-size: 12px;
      height: 40px;
    }

    select {
      background-position: 155px center;
    }
  }
}

//desktop 1920 styles
@media (min-width: 1600px) {
  .main {
    max-width: 890px;

    input,
    select,
    button {
      width: 205px;
      padding: 15px;
      font-size: 14px;
      border-radius: 16px;
      height: 50px;
    }

    .converter-popup {
      header {
        .logo {
          img {
            max-width: 400px;
          }
        }
      }

      .converter-content {
        .result {
          .result-amount {
            font-size: 32px;
          }

          .one-amount-rate {
            font-size: initial;
          }
        }
      }
    }
  }
}
</style>
