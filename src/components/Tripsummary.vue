<template>
  <div class="headerWrap">
    <div class="header">
      <div class="locationWrap">
        <div class="location">{{ `${city}, ${country}` }}</div>
        <div class="term">{{ duration }} DAY TRIP</div>
      </div>
      <div v-if="toCurrency" class="exchangeWrap">
        <i class="material-icons moneyIcon">attach_money</i>
        <div>
          <div class="exchangeRate">
            {{ `${currency} ${fromCurrency} ${equal} ${exchangeRate1} ${toCurrency}` }}
          </div>
          <div class="exchangeRate">
            {{ `${currency} ${toCurrency} ${equal} ${exchangeRate2} ${fromCurrency}` }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import countryDetails from "../../data/country_details.json";
import moment from "moment";

export default {
  name: "Tripsummary",
  data: () => ({
    country: null,
    city: null,
    tripStart: localStorage.tripStart,
    tripEnd: localStorage.tripEnd,
    duration: null,
    currency: "",
    equal: "",
    fromCurrency: "",
    toCurrency: "",
    exchangeRate1: "",
    exchangeRate2: ""
  }),
  mounted() {
    this.country = localStorage.country;
    this.city = localStorage.city;
    this.tripStart = localStorage.tripStart;
    this.tripEnd = localStorage.tripEnd;

    const start = new Date(localStorage.tripStart);
    const end = new Date(localStorage.tripEnd);
    this.duration = 1 + Number((end.getTime() - start.getTime())) / 86400000;
    this.fetchUserLocationInfo().then(() => {
      this.fetchExchangeRate();
    });
  },
  methods: {
    async fetchExchangeRate() {
      const exchangeRate1 = await axios.get(
        `https://currency-exchange.p.rapidapi.com/exchange?q=1&from=${this.fromCurrency}&to=${this.toCurrency}`,
        {
          headers: {
            "x-rapidapi-host": "currency-exchange.p.rapidapi.com",
            "x-rapidapi-key":
              "b6e4f9fc03msh80db2bc55980af4p181a67jsnb4b3c557714d"
          }
        }
      );
      this.exchangeRate1 = exchangeRate1.data.toFixed(3);
      const exchangeRate2 = await axios.get(
        `https://currency-exchange.p.rapidapi.com/exchange?q=1&from=${this.toCurrency}&to=${this.fromCurrency}`,
        {
          headers: {
            "x-rapidapi-host": "currency-exchange.p.rapidapi.com",
            "x-rapidapi-key":
              "b6e4f9fc03msh80db2bc55980af4p181a67jsnb4b3c557714d"
          }
        }
      );
      this.exchangeRate2 = exchangeRate2.data.toFixed(3);
      this.currency = 1;
      this.equal = "=";
    },
    async fetchUserLocationInfo() {
      let ipAddress = await axios("https://api.ipify.org/?format=json");
      ipAddress = ipAddress.data.ip;
      const userLocationInfo = await axios(
        `https://ip-geo-location.p.rapidapi.com/ip/${ipAddress}`,
        {
          headers: {
            "x-rapidapi-host": "ip-geo-location.p.rapidapi.com",
            "x-rapidapi-key":
              "b6e4f9fc03msh80db2bc55980af4p181a67jsnb4b3c557714d"
          }
        }
      );
      localStorage.a = userLocationInfo;
      this.toCurrency = countryDetails.filter(
        item => item.country === localStorage.country
        )[0].currency_code;
      this.fromCurrency = userLocationInfo.data.currency.code;
    }
  },
  filters: {
    moment: function(date) {
      return moment(date).format("ddd MMM D YYYY");
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}
.location {
  font-size: 42px;
}
.exchangeWrap {
  display: flex;
  padding: 10px;
  background: #333;
  margin-left: 70px;
  color: #fff;
  border-radius: 15px;
}
.exchange-title {
  font-weight: bold;
  margin: 5px 5px 5px 0px;
}
.exchangeRate {
  margin-right: 10px;
  margin-left: 10px;
}
.moneyIcon {
  font-size: 40px;
  height: 44px;
  line-height: 44px;
}
@media screen and (max-width: 480px) {
  .header {
    display: inline-block;
  }
  .location {
    font-size: 30px;
  }
  .term {
    margin-bottom: 10px;
  }
  .exchangeWrap {
    margin: 0 15%;
  }
}
</style>
