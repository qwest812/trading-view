<template>
<div class="TVChartContainer" :id="containerId" />
</template>

<script>
let data =[{"symbol":"USD/UAH","full_name":"DECK","description":"Deckers Outdoor Corp.","exchange":"NYSE","type":"stock"},{"symbol":"A","full_name":"A","description":"Agilent Technologies Inc.","exchange":"NYSE","type":"stock"},{"symbol":"AAL","full_name":"AAL","description":"American Airlines Group Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"AAPL","full_name":"AAPL","description":"Apple Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"ABBV","full_name":"ABBV","description":"AbbVie Inc.","exchange":"NYSE","type":"stock"},{"symbol":"ABT","full_name":"ABT","description":"Abbott Laboratories","exchange":"NYSE","type":"stock"},{"symbol":"ACHN","full_name":"ACHN","description":"Achillion Pharmaceuticals, Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"ACI","full_name":"ACI","description":"Arch Coal Inc.","exchange":"NYSE","type":"stock"},{"symbol":"ACN","full_name":"ACN","description":"Accenture plc","exchange":"NYSE","type":"stock"},{"symbol":"ADBE","full_name":"ADBE","description":"Adobe Systems Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"ADSK","full_name":"ADSK","description":"Autodesk, Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"AEO","full_name":"AEO","description":"American Eagle Outfitters, Inc.","exchange":"NYSE","type":"stock"},{"symbol":"AGNC","full_name":"AGNC","description":"American Capital Agency Corp.","exchange":"NasdaqNM","type":"stock"},{"symbol":"AIG","full_name":"AIG","description":"American International Group, Inc.","exchange":"NYSE","type":"stock"},{"symbol":"AKAM","full_name":"AKAM","description":"Akamai Technologies, Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"ALXN","full_name":"ALXN","description":"Alexion Pharmaceuticals, Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"AMAT","full_name":"AMAT","description":"Applied Materials, Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"AMD","full_name":"AMD","description":"Advanced Micro Devices, Inc.","exchange":"NYSE","type":"stock"},{"symbol":"AMGN","full_name":"AMGN","description":"Amgen Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"AMZN","full_name":"AMZN","description":"Amazon.com Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"ANF","full_name":"ANF","description":"Abercrombie & Fitch Co.","exchange":"NYSE","type":"stock"},{"symbol":"ANR","full_name":"ANR","description":"Alpha Natural Resources, Inc.","exchange":"NYSE","type":"stock"},{"symbol":"APA","full_name":"APA","description":"Apache Corp.","exchange":"NYSE","type":"stock"},{"symbol":"APC","full_name":"APC","description":"Anadarko Petroleum Corporation","exchange":"NYSE","type":"stock"},{"symbol":"ARC","full_name":"ARC","description":"ARC Document Solutions, Inc.","exchange":"NYSE","type":"stock"},{"symbol":"ARIA","full_name":"ARIA","description":"Ariad Pharmaceuticals Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"ARNA","full_name":"ARNA","description":"Arena Pharmaceuticals, Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"ARR","full_name":"ARR","description":"ARMOUR Residential REIT, Inc.","exchange":"NYSE","type":"stock"},{"symbol":"AUXL","full_name":"AUXL","description":"Auxilium Pharmaceuticals Inc.","exchange":"NasdaqNM","type":"stock"},{"symbol":"AVGO","full_name":"AVGO","description":"Avago Technologies Limited","exchange":"NasdaqNM","type":"stock"}];
import { widget } from '../charting_library.min';
var rp = require('request-promise').defaults({json: true});
const api_root = 'https://min-api.cryptocompare.com';

let historyProvider={
    getBars: function(symbolInfo, resolution, from, to, first, limit) {
        console.log(symbolInfo.name);
        var split_symbol = symbolInfo.name.split('/');
        console.log(split_symbol);
        // const url = resolution === 'D' ? '/data/histoday' : resolution >= 60 ? '/data/histohour' : '/data/histominute';
        const url = resolution === 'D' ? '/data/histoday' : resolution >= 60 ? '/data/histohour' : '/data/histohour';
        const qs = {
            // e: split_symbol[0], // Coinbase
            fsym: split_symbol[0], // BTC
            tsym: split_symbol[1], // USD
            toTs:  to ? to : '',
            limit:2000,
        };
        return rp({
            url: `${api_root}${url}`,
            // url: 'test/tt.com',
            qs,
        })
            .then(data => {
                if (data.Response && data.Response === 'Error') {
                    console.log('CryptoCompare API error:',data.Message);
                    return []
                }
                if (data.Data.length) {
                    var bars = data.Data.map(el => {
                        return {
                            time: el.time * 1000, //TradingView requires bar time in ms
                            low: el.low,
                            high: el.high,
                            open: el.open,
                            close: el.close,
                            volume: el.volumefrom
                        }
                    });
                    return bars
                } else {
                    return []
                }
            })
    }
};

function getLanguageFromURL() {
  const regex = new RegExp('[\\?&]lang=([^&#]*)');
  const results = regex.exec(window.location.search);
  return results === null ? null : decodeURIComponent(results[1].replace(/\+/g, ' '));
}
const config = {
    supported_resolutions: ["1", "3", "5", "15", "30", "60", "120",   "240", "D"]
};
let  Datafeed={
    onReady: cb => {
        console.log('=====onReady running');
        setTimeout(() => cb(config), 0)
    },
// only need searchSymbols when search is enabled
    searchSymbols: (userInput, exchange, symbolType, onResultReadyCallback) => {
        console.log(userInput);
        console.log(exchange);
        onResultReadyCallback(data)
    },
    resolveSymbol: (symbolName, onSymbolResolvedCallback, onResolveErrorCallback) => {
        var split_data = symbolName.split(/[:/]/);

        var symbol_stub = {
            name: symbolName,
            description: '',
            type: 'crypto',
            session: '24x7',
            timezone: 'America/New_York',
            ticker: symbolName,
            minmov: 1,
            pricescale: 100000000,
            has_intraday: true,
            intraday_multipliers: ['1', '60'],
            supported_resolution:  ["1", "3", "5", "15", "30", "60", "120",   "240", "D"],
            volume_precision: 8,
            data_status: 'streaming',
        };
        if (split_data[1].match(/USD|EUR|JPY|AUD|GBP|KRW|CNY/)) {
            symbol_stub.pricescale = 100
        }

        setTimeout(function() {
            onSymbolResolvedCallback(symbol_stub)
        }, 0)
    },
    getBars: function(symbolInfo, resolution, from, to, onHistoryCallback, onErrorCallback, firstDataRequest) {

        historyProvider.getBars(symbolInfo, resolution, from, to, firstDataRequest)
            .then(bars => {
                if (bars.length) {
                    onHistoryCallback(bars, {noData: false})
                } else {
                    onHistoryCallback(bars, {noData: true})
                }
            }).catch(err => {
            console.log({err});
            onErrorCallback(err)
        })
    },
    subscribeBars: (symbolInfo, resolution, onRealtimeCallback, subscribeUID, onResetCacheNeededCallback) => {},
    unsubscribeBars: subscriberUID => {},

    /* optional methods */
    getServerTime: cb => {},
    calculateHistoryDepth: (resolution, resolutionBack, intervalBack) => {},
    getMarks: (symbolInfo, startDate, endDate, onDataCallback, resolution) => {},
    getTimeScaleMarks: (symbolInfo, startDate, endDate, onDataCallback, resolution) => {}
};

export default {
  name: 'TVChartContainer',
  props: {
    symbol: {
      default: 'AAPL',
      type: String,
    },
    interval: {
      default: 'D',
      type: String,
    },
    containerId: {
      default: 'tv_chart_container',
      type: String,
    },
    datafeedUrl: {
      default: 'https://demo_feed.tradingview.com',
      type: String,
    },
    libraryPath: {
      default: '/charting_library/',
      type: String,
    },
    chartsStorageUrl: {
      default: 'https://saveload.tradingview.com',
      type: String,
    },
    chartsStorageApiVersion: {
      default: '1.1',
      type: String,
    },
    clientId: {
      default: 'tradingview.com',
      type: String,
    },
    userId: {
      default: 'public_user_id',
      type: String,
    },
    fullscreen: {
      default: false,
      type: Boolean,
    },
    autosize: {
      default: true,
      type: Boolean,
    },
    studiesOverrides: {
      type: Object,
    }
  },
  tvWidget: null,
  mounted() {
      const widgetOptions = {
          debug: false,
          symbol: 'BTC/UAH',
          datafeed: Datafeed, // our datafeed object
          interval: '9',
          container_id: 'tv_chart_container',
          library_path: '/charting_library/',
          locale: getLanguageFromURL() || 'en',
          disabled_features: ['use_localstorage_for_settings'],
          enabled_features: [],
          client_id: 'test',
          user_id: 'public_user_id',
          fullscreen: false,
          autosize: true,
          overrides: {
              "paneProperties.background": "#352072",
              "paneProperties.vertGridProperties.color": "#363c4e",
              "paneProperties.horzGridProperties.color": "#363c4e",
              "symbolWatermarkProperties.transparency": 180,
              "scalesProperties.textColor" : "#AAA",
              "mainSeriesProperties.candleStyle.wickUpColor": '#0f6825',
              "mainSeriesProperties.candleStyle.wickDownColor": '#7f0005',
          }
      };

    const tvWidget = new widget(widgetOptions);
    this.tvWidget = tvWidget;

    tvWidget.onChartReady(() => {
      tvWidget.headerReady().then(() => {
        const button = tvWidget.createButton();

        button.setAttribute('title', 'Click to show a notification popup');
        button.classList.add('apply-common-tooltip');

        button.addEventListener('click', () => tvWidget.showNoticeDialog({
            title: 'Notification',
            body: 'TradingView Charting Library API works correctly',
            callback: () => {
              // eslint-disable-next-line no-console
              console.log('Noticed!');
            },
          }));

        button.innerHTML = 'Check API';
      });
    });
  },
  destroyed() {
    if (this.tvWidget !== null) {
      this.tvWidget.remove();
      this.tvWidget = null;
    }
  }
}
</script>

<style lang="scss" scoped>
.TVChartContainer {
  height: calc(100vh - 80px);
}
</style>
