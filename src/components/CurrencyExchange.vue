<template>
    <div class="main">
        <div class="converter">
            <header>
                <div class="header-block"><span>Currency converter</span></div>
                <div class="header-block"><img class="baby" src="../assets/logo.svg"></div>
                <div class="header-block"><span>Using Monobank API</span></div>
            </header>
            <div class="converter-content">
              <div class="converter">
                <div class="customer-data">
                  <label for="amount">Type an amount </label>
                  <input id="amount" type="number" placeholder="1000" required>
                </div>
                <div class="customer-data">
                  <label for="currency-from">Choose currency FROM:</label>
                  <select name="Select currency" id="currency-from" required>
                    <option value="" selected>From</option>
                    <option value="default">default</option>
                  </select>
                </div>
                <div class="customer-data">
                  <label for="currency-to">Choose currency TO:</label>
                  <select name="Select currency" id="currency-to" required>
                    <option value="" selected>To</option>
                    <option value="test">test</option>
                  </select>
                </div>
              </div>
              <div class="result">
                <div class="button get-result"><span>Get result</span></div>
                <p class="result-amount">100 Euro</p>
              </div>
            </div>
          <div class="mask mask-1"></div>
          <div class="mask mask-2"></div>
        </div>
    </div>
</template>
<script>

//const URL = 'https://api.monobank.ua/bank/currency'
import availableCurrencies from '../assets/currencies.json'

export default {
  name: 'CurrencyExchange',
  data() {
    return {
      //availableCurrenciesParsed: JSON.parse(availableCurrencies)

    }
  },

  async beforeMount() {
   // let currencies = await this.getCurrencies(URL);
    //console.log(currencies);
  },

  created() {

    console.log(availableCurrencies);
  },

  methods: {
    async getCurrencies (url) {
       try {
        const response = await fetch(url, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json'
          },
        });

        return response.json();

       } catch (error) {
         console.log(error);
       }
      

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
      margin-top: 180px;
      min-height: 100%;

      input, select {
        width: 240px;
        border: 1px solid #af84b9;
        padding: 15px;
        font-size: 15px;
        border-radius: 16px;
      }

      select {
        appearance: none;       /* Remove default arrow */
        background-image: url('../assets/arrow.png');   /* Add custom arrow */
        background-position: 200px center;
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
        width: 240px;
        border-radius: 16px;
        padding: 15px;
        font-size: 17px;
        font-weight: 400;
        text-align: center;
        cursor: pointer;
        text-transform: uppercase;
        transition: box-shadow .3s;
        background-color: #fa5255;
        color: white;

        &:hover {
          -webkit-box-shadow: 0px 9px 21px -8px #9f99a2;
          -moz-box-shadow: 0px 9px 21px -8px #9f99a2;
          box-shadow: 0px 9px 21px -8px #9f99a2;
        }
      }

        .converter {
          max-width: 940px;
          width: 100%;
          border-radius: 32px;
          background-color: white;
          position: relative;
          z-index: 1;
            
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
              justify-content: space-around;
              padding: 20px;
              min-height: 450px;
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