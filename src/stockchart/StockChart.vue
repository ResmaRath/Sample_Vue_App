<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-12">
        <form @submit.prevent="handleSubmit">
          <div class="form-group">
            <label for="stockName">Enter Stock Symbol</label>
            <input type="text" v-model="stockName" name="stockName" class="form-control" />
          </div>
          <div class="form-group">
            <button class="btn btn-primary">Load Data</button>
          </div>
          <h2>Stock Chart</h2>
          <div id="chartContainer"></div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";
import Vue from "vue";
import axios from "axios";

export default {
  data() {
    return {
      isLoading: false,
      stockName: "",
      stocks: [],
      msg: ""
    };
  },
  computed: {
    ...mapState({
      account: state => state.account,
      users: state => state.users.all,
      symbols: state => state.symbols
    })
  },
  created() {
    let getSymbol =
      localStorage.getItem("symbol") !== null
        ? localStorage.getItem("symbol")
        : "MSFT";
    this.fetchStocks(getSymbol);
  },
  methods: {
    buildUrl(symbol) {
      const url =
        "https://www.alphavantage.co/query?function=TIME_SERIES_DAILY";
      const apikey = "FI33C6T4WLQ2RC72";
      return url + "&symbol=" + symbol + "&apikey=" + apikey;
    },
    handleSubmit(e) {
      const { stockName } = this;
      localStorage.setItem("symbol", stockName);
      this.fetchStocks(stockName);
    },
    loadChart() {
      var chart = new CanvasJS.Chart("chartContainer", {
        animationEnabled: true,
        theme: "light2",
        exportEnabled: false,
        axisX: {
          interval: 1,
          valueFormatString: "DD-MMM"
        },
        axisY: {
          includeZero: false,
          prefix: "$"
        },
        toolTip: {
          content:
            "Date: {x}<br /><strong>Price:</strong><br />Open: {y[0]}, Close: {y[3]}<br />High: {y[1]}, Low: {y[2]}"
        },
        data: [
          {
            type: "candlestick",
            yValueFormatString: "$##0.00",
            dataPoints: this.stocks
          }
        ]
      });
      chart.render();
    },
    fetchStocks(stockName) {
      this.stocks = [];
      this.isLoading = true;
      this.stockName = stockName;
      localStorage.setItem("symbol", this.stockName);
      let url = this.buildUrl(this.stockName);
      axios({ method: "GET", url: url }).then(result => {
        var vm = this;
        var stockObj = result.data["Time Series (Daily)"];
        let msg =
          typeof result.data["Information"] !== "undefined"
            ? result.data["Information"]
            : "";
        this.msg = msg;
        this.isLoading = false;
        for (var key in stockObj) {
          vm.stocks.push({
            x: new Date(key),
            y: [
              parseFloat(stockObj[key]["1. open"]),
              parseFloat(stockObj[key]["2. high"]),
              parseFloat(stockObj[key]["3. low"]),
              parseFloat(stockObj[key]["4. close"])
            ]
          });
        }
        vm.loadChart();
      });
    }
  }
};
</script>