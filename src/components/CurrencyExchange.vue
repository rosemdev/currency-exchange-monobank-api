<template>
    <div class="main">
        <div class="converter-popup">
            <header>
                <div class="header-block"><span>Currency converter</span></div>
                <div class="header-block"><img class="baby" src="../assets/logo.svg"></div>
                <div class="header-block"><span>Using Monobank API</span></div>
            </header>
            <div class="converter-content">
              <form class="converter"
                    v-on:submit.prevent="exchange(amount, currencyFrom.currencyCode, currencyTo.currencyCode )"
              >
                <div class="customer-data">
                  <label for="amount">Type an amount </label>
                  <input id="amount" name="amount" type="number" placeholder="1000" step="0.01" required v-model="amount"
                         @change="exchange(amount, currencyFrom.currencyCode, currencyTo.currencyCode )">
                </div>
                <div class="customer-data">
                  <label for="currency-from">Choose currency FROM:</label>
                  <select name="currency-from" id="currency-from" required v-model="currencyFrom"
                          @change="exchange(amount, currencyFrom.currencyCode, currencyTo.currencyCode )">
                    <option value="" selected>From</option>
                    <option v-bind:value="{currencyCode: currency.NumericCode, currencyLetterName:currency.AlphabeticCode}"
                            v-for="currency in availableCurrencies"
                            :key="currency.AlphabeticCode">{{currency.AlphabeticCode}} - {{currency.Currency}}
                    </option>
                  </select>
                </div>
                <div class="customer-data">
                  <label for="currency-to">Choose currency TO:</label>
                  <select name="currency-to" id="currency-to" required v-model="currencyTo"
                          @change="exchange(amount, currencyFrom.currencyCode, currencyTo.currencyCode )">
                    <option value="" selected>To</option>
                    <option v-bind:value="{currencyCode: currency.NumericCode, currencyLetterName:currency.AlphabeticCode}"
                            v-for="currency in availableCurrencies"
                            :key="currency.AlphabeticCode">{{currency.AlphabeticCode}} - {{currency.Currency}}
                    </option>
                  </select>
                </div>
                <div class="button get-result"><button>Get result</button></div>
              </form>
              <div class="result">
                <p class="result-amount">
                  <span>{{amount}} {{currencyFrom.currencyLetterName}}</span>
                  <span v-if="result"> = {{result}} {{currencyTo.currencyLetterName}}</span>
                </p>
              </div>
            </div>
          <div class="mask mask-1"></div>
          <div class="mask mask-2"></div>
        </div>
    </div>
</template>
<script>

const URL = 'https://api.monobank.ua/bank/currency'
import availableCurrencies from '../assets/currencies.json'

export default {
  name: 'CurrencyExchange',
  data() {
    return {
      monoCurrencies: '',
      availableCurrencies,
      uanCurrencyCode: 980,
      amount: '',
      currencyFrom: '',
      currencyTo: '',
      result: undefined
    }
  },

  async beforeMount() {
   this.monoCurrencies = await this.getCurrencies(URL);

   if(this.monoCurrencies !== null) {
     this.addToLocalStorage(this.monoCurrencies);
   } else {
     this.monoCurrencies = this.getDataFromLocaleStorage();
   }
  },

  methods: {
    async getCurrencies(url) {
      try {
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json'
          },
        });

        if (response.status === 200) {
          return response.json();
        } else {
          return null;
        }
      } catch (error) {
        console.log(error);
      }
    },

    exchange(amount, currencyFromCode, currencyToCode) {
      // eslint-disable-next-line
      //debugger

      amount = parseFloat(amount);
      currencyFromCode = parseInt(currencyFromCode);
      currencyToCode = parseInt(currencyToCode);

      let currencyFromExchangeRate = '';
      let currencyToExchangeRate = '';
      let uanExchangeData = '';
      let uanRate = '';

      //help to know exchange type
      // UAN to ANY (returns true) or
      // ANY to UAN (returns false)
      let isExchangeFromUan = currencyFromCode === this.uanCurrencyCode;

      //Exchange UAN
      if (currencyFromCode === this.uanCurrencyCode || currencyToCode === this.uanCurrencyCode) {
        uanExchangeData = this.mono_findUanCurrencyRateObj(this.monoCurrencies, {currencyFromCode, currencyToCode});
        uanRate = this.mono_getUanRate(uanExchangeData, isExchangeFromUan);

      } else {
        //Exchange not UAN
        currencyFromExchangeRate = this.mono_findMultiCurrencyRateObj(this.monoCurrencies, currencyFromCode);
        currencyToExchangeRate = this.mono_findMultiCurrencyRateObj(this.monoCurrencies, currencyToCode);
        uanRate = this.mono_getMultiCurrencyRate(currencyFromExchangeRate, currencyToExchangeRate);
      }

      console.log('test', amount, currencyFromCode, currencyToCode);
      console.log(currencyFromExchangeRate, currencyToExchangeRate);

      return this.result = (amount * uanRate).toFixed(2);
    },

    //Exchange UAN
    mono_findUanCurrencyRateObj(arr, searchExchangeCodes) {
      let {currencyFromCode, currencyToCode} = searchExchangeCodes;
      let searchCode = '';

      //currencyCodeA => returns ANY currency, which is unique identifier
      //currencyCodeB => returns UAN currency, which is not unique identifier
      //from UAN to ANY => search by currencyToCode, which is unique
      if(currencyFromCode === this.uanCurrencyCode) {
        searchCode = currencyToCode;
      } else {
        //from ANY to UAN => search by currencyFromCode, which is unique
        searchCode = currencyFromCode;
      }

      return this.mono_findMultiCurrencyRateObj(arr, searchCode);

    },

    //Exchange not UAN
    mono_findMultiCurrencyRateObj(arr, searchCode) {
      return arr.find(item => {
        return item.currencyCodeA === searchCode && item.currencyCodeB === this.uanCurrencyCode;
      });
    },

    addToLocalStorage(data) {
      let serialData = JSON.stringify(data);
      localStorage.setItem("monocurruncies", serialData);
    },

    getDataFromLocaleStorage() {
      return JSON.parse(localStorage.getItem("monocurruncies"));
    },

    //Exchange UAN
    mono_getUanRate(currencyRate, isExchangeFromUan) {
      let uanRate;
      let availableRate = currencyRate.rateSell ? currencyRate.rateSell : currencyRate.rateCross

      if(isExchangeFromUan) {
        uanRate = 1 / availableRate;
      } else {
        uanRate = availableRate;
      }

      return uanRate
    },

    //Exchange not UAN
    mono_getMultiCurrencyRate(currencyFrom, currencyTo) {
      let uanRate;

      //checking different cases when rateSell or rateCross are available or not
      if (currencyFrom.rateSell && currencyTo.rateSell) {
        uanRate = currencyFrom.rateSell / currencyTo.rateSell;

      } else if (currencyFrom.rateSell && currencyTo.rateCross) {
        uanRate = currencyFrom.rateSell / currencyTo.rateCross;

      } else if (currencyFrom.rateCross && currencyTo.rateSell) {
        uanRate = currencyFrom.rateCross / currencyTo.rateSell;
      } else {
        uanRate = currencyFrom.rateCross / currencyTo.rateCross;
      }

      return uanRate;
    }
  }
}
</script>
<style lang="less">

    .main {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;

      input, select, button {
        width: 240px;
        padding: 15px;
        font-size: 16px;
        border-radius: 16px;
        height: 50px;
        font-family: 'Poppins', sans-serif;
      }

      input, select {
        border: 1px solid #af84b9;
      }

      select {
        appearance: none;       /* Remove default arrow */
        background-image: url('../assets/arrow.png');   /* Add custom arrow */
        background-position: 212px center;
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

      .button {
        margin: 30px 15px;

        button {
          font-weight: 400;
          text-align: center;
          cursor: pointer;
          text-transform: uppercase;
          transition: box-shadow .3s;
          background-color: #fa5255;
          color: white;
          font-family: 'Poppins', sans-serif;

          &:hover {
            -webkit-box-shadow: 0px 9px 21px -8px #9f99a2;
            -moz-box-shadow: 0px 9px 21px -8px #9f99a2;
            box-shadow: 0px 9px 21px -8px #9f99a2;
          }
        }
      }
        .converter-popup {
          max-width: 940px;
          width: 100%;
          border-radius: 32px;
          margin-top: 180px;

            header {
              display: flex;
              align-items: center;
              justify-content: space-between;
              height: 96px;
              background-color: #f2f4f5;
              border-top-left-radius: 32px;
              border-top-right-radius: 32px;
              padding: 20px;

            }

            .converter-content {
              display: flex;
              flex-direction: column;
              justify-content: center;
              padding: 20px;
              min-height: 420px;
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

                  label {
                    display: block;
                    margin: 5px;
                  }
                }
              }

              .result {
                align-self: center;

                .result-amount {
                  font-weight: 400;
                  font-size: 32px;
                  text-align: center;
                }
              }

            }
        }

      .mask {
        position: absolute;
        height: 100px;
        border-radius: 32px;
        background-color: rgba(31,33,43,0.32);
      }
      .mask-1 {
        margin-top: -50px;
        width: 830px;
        margin-left: 55px;
        filter: blur(48px);
      }

      .mask-2 {
        margin-top: -90px;
        width: 880px;
        margin-left: 30px;
        filter: blur(32px);
      }
    }
</style>