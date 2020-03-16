<template>
  <div id="app">
    <h2>State: {{ connectionState }}</h2>
    <h4>Message: {{ lastMessage }}</h4>
    <div id="wrapper">
      <div id="chart"></div>

      <div id="tableWrapper">
        <table id="ratesTable">
          <thead>
            <tr>
              <th>Currency exchange</th>
              <th>Exchange rate</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>ETH/USD</td>
              <td>{{ latestEthUsdValue }}</td>
            </tr>
            <tr>
              <td>BTC/USD</td>
              <td>{{ latestBtcUsdValue }}</td>
            </tr>
            <tr>
              <td>BTC/EUR</td>
              <td>{{ latestBtcEurValue }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
/* global Highcharts */
export default {
  name: "App",
  mounted: function() {
    this.connectToSocket();
  },
  data: () => {
    return {
      connectionState: "Component initialized.",
      lastMessage: "",
      ethUsd: [],
      btcUsd: [],
      btcEur: [],
      latestEthUsdValue: null,
      latestBtcUsdValue: null,
      latestBtcEurValue: null,
      chart: null
    };
  },
  methods: {
    // Connects to bitstamp, subscribes to 3 channels of live trade rates & handles messages from them
    connectToSocket() {
      const wsUrl = "wss://ws.bitstamp.net.";
      this.connectionState = `Connecting to bitstamp... (${wsUrl})`;
      const mySocket = new WebSocket(wsUrl);
      this.connectionState = "Connected to bitstamp...";

      mySocket.addEventListener("open", () => {
        this.drawChart();

        mySocket.send(
          JSON.stringify({
            event: "bts:subscribe",
            data: {
              channel: "live_trades_ethusd"
            }
          })
        );
        mySocket.send(
          JSON.stringify({
            event: "bts:subscribe",
            data: {
              channel: "live_trades_btcusd"
            }
          })
        );
        mySocket.send(
          JSON.stringify({
            event: "bts:subscribe",
            data: {
              channel: "live_trades_btceur"
            }
          })
        );
      });

      // Listen for messages
      mySocket.addEventListener("message", event => {
        console.log("Message from server ", event.data);
        const data = JSON.parse(event.data);
        console.log("Message from server ", data.data.timestamp);
        switch (data.channel) {
          case "live_trades_ethusd":
            this.ethUsd.push([data.data.timestamp, data.data.price]);
            this.chart.series[0].setData(this.ethUsd);
            this.latestEthUsdValue = data.data.price;
            break;
          case "live_trades_btcusd":
            this.btcUsd.push([data.data.timestamp, data.data.price]);
            this.chart.series[1].setData(this.btcUsd);
            this.latestBtcUsdValue = data.data.price;
            break;
          case "live_trades_btceur":
            this.btcEur.push([data.data.timestamp, data.data.price]);
            this.chart.series[2].setData(this.btcEur);
            this.latestBtcEurValue = data.data.price;
            break;
        }
        this.chart.redraw();
        this.lastMessage = event.data;
      });
    },

    // Draws the chart for the live trades
    drawChart() {
      this.chart = Highcharts.chart("chart", {
        chart: {
          zoomType: "x"
        },
        title: {
          text: "BTC to USD to EUR to ETH exchange rate over time"
        },
        subtitle: {
          text:
            document.ontouchstart === undefined
              ? "Click and drag in the plot area to zoom in"
              : "Pinch the chart to zoom in"
        },
        xAxis: {
          type: "datetime"
        },
        yAxis: {
          title: {
            text: "Exchange rates"
          }
        },
        legend: {
          enabled: true
        },
        plotOptions: {
          area: {
            fillColor: {
              linearGradient: {
                x1: 0,
                y1: 0,
                x2: 0,
                y2: 1
              },
              stops: [
                [0, Highcharts.getOptions().colors[0]],
                [
                  1,
                  Highcharts.color(Highcharts.getOptions().colors[0])
                    .setOpacity(0)
                    .get("rgba")
                ]
              ]
            },
            marker: {
              radius: 2
            },
            lineWidth: 1,
            states: {
              hover: {
                lineWidth: 1
              }
            },
            threshold: null
          }
        },

        series: [
          {
            type: "area",
            name: "ETH to USD",
            data: this.ethUsd
          },
          {
            type: "area",
            name: "BTC to USD",
            data: this.btcUsd
          },
          {
            type: "area",
            name: "BTC to EUR",
            data: this.btcEur
          }
        ]
      });
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#wrapper {
  display: inline-flex;
  justify-content: space-between;
}

#chart {
  height: 400px;
  min-width: 1000px;
}

.highcharts-figure,
.highcharts-data-table table {
  min-width: 360px;
  max-width: 800px;
  margin: 1em auto;
}

.highcharts-data-table table {
  font-family: Verdana, sans-serif;
  border-collapse: collapse;
  border: 1px solid #ebebeb;
  margin: 10px auto;
  text-align: center;
  width: 100%;
  max-width: 500px;
}
.highcharts-data-table caption {
  padding: 1em 0;
  font-size: 1.2em;
  color: #555;
}
.highcharts-data-table th {
  font-weight: 600;
  padding: 0.5em;
}
.highcharts-data-table td,
.highcharts-data-table th,
.highcharts-data-table caption {
  padding: 0.5em;
}
.highcharts-data-table thead tr,
.highcharts-data-table tr:nth-child(even) {
  background: #f8f8f8;
}
.highcharts-data-table tr:hover {
  background: #f1f7ff;
}
</style>
