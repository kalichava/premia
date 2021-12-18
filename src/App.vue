<template>
  <div class="wrapper">
    <div class="container">
      <div class="column">
        <div class="box box--data">
          <div class="add4bottom">
            <span class="icon" v-html="pair.wbtc.icon"></span>
            
            <span v-if="pair.wbtc.liquidity">
              <strong class="big add4left">{{ pair.wbtc.liquidity }}</strong>
              <span class="lighter add4left">WBTC</span>
            </span>
            <span v-else class="add4left" v-html="api.loadingIcon"></span>

          </div>
          <div>
            <span>
              1 WBTC
              <span v-if="pair.wbtc.swapPrice">
                = {{ pair.wbtc.swapPrice }} WETH
                <span class="lighter add4left">({{ pair.wbtc.usdPrice }})</span>
              </span>
            </span>
          </div>
        </div>
        <div class="box">
          <apexchart
            type="area"
            height="275"
            :options="charts.liquidity.options"
            :series="charts.liquidity.series"
          />
          <tabs
            :tabs="charts.liquidity.tabs"
            :active="charts.liquidity.currentTabId"
            chart="liquidity"
            @changeTab="onChangeTab"
          />
          <div class="labels">
            <div class="add4bottom"><span>Liquidity</span><span class="light add4left" v-if="pair.latest.date" ref="liquidityDate">{{ pair.latest.date }}</span></div>
            <div class="big">
              <strong v-if="pair.latest.liquidity" ref="liquidityAmount">{{ pair.latest.liquidity }}</strong>
              <span class="light" v-else>Loading...</span>
            </div>
          </div>
        </div>
      </div>
      <div class="column">
        <div class="box box--data">
          <div class="add4bottom">
            <span class="icon" v-html="pair.weth.icon"></span>
            
            <span v-if="pair.weth.liquidity">
              <strong class="big add4left">{{ pair.weth.liquidity }}</strong>
              <span class="lighter add4left">WETH</span>
            </span>
            <span v-else class="add4left" v-html="api.loadingIcon"></span>

          </div>
          <div>
            <span>
              1 WETH 
              <span v-if="pair.weth.swapPrice">
                = {{ pair.weth.swapPrice }} WBTC
                <span class="lighter add4left">({{ pair.weth.usdPrice }})</span>
              </span>
            </span>
          </div>
        </div>
        <div class="box">
          <apexchart
            type="bar"
            height="290"
            :options="charts.volume.options"
            :series="charts.volume.series"
          />
          <tabs
            :tabs="charts.volume.tabs"
            :active="charts.volume.currentTabId"
            chart="volume"
            @changeTab="onChangeTab"
          />
          <div class="labels">
            <div class="add4bottom"><span>Volume</span><span class="light add4left" v-if="pair.latest.date" ref="volumeDate">{{ pair.latest.date }}</span></div>
            <div class="big">
              <strong v-if="pair.latest.volume" ref="volumeAmount">{{ pair.latest.volume }}</strong>
              <span class="light" v-else>Loading...</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
  import VueApexCharts from "vue-apexcharts"
  import Tabs from "./components/Tabs.vue"
  import axios from "axios"
  import { fromUnixTime, getUnixTime, differenceInDays, format } from 'date-fns'

  export default {
    name: 'App',

    components: {
      apexchart: VueApexCharts,
      tabs: Tabs,
    },

    data() {
      return {
        api: {
          url: 'https://api.thegraph.com/subgraphs/name/sushiswap/exchange',
          loadingIcon: '<svg xmlns="http://www.w3.org/2000/svg" style="position:relative; top:2px; display:inline-block; shape-rendering:auto" width="21" height="21" viewBox="0 0 100 100" preserveAspectRatio="xMidYMid"><circle cx="50" cy="50" fill="none" stroke="#dcdcdc" stroke-width="9" r="31" stroke-dasharray="146.08405839192537 50.69468613064179"><animateTransform attributeName="transform" type="rotate" repeatCount="indefinite" dur="1.6949152542372883s" values="0 50 50;360 50 50" keyTimes="0;1"/></circle></svg>',

          // first record with full data created at Sep 29 2020
          firstTimestamp: 1601330400,
        },

        charts: {
          liquidity: {
            options: {
              chart: {
                 width: '100%',
                id: 'liquidityChart',
                toolbar: {
                  show: false
                },
                fontFamily: 'inherit',
                foreColor: '#474c50',
                offsetX: 8,
              },

              dataLabels: {
                enabled: false
              },

              stroke: {
                curve: 'smooth',
                width: 1,
              },

              colors: ['#00FF97'],

              fill: {
                type: "gradient",
                gradient: {
                  shadeIntensity: 1,
                  opacityFrom: 0.5,
                  opacityTo: 0,
                  stops: [0, 80, 100]
                }
              },

              grid: {
                show: false
              },
              
              xaxis: {
                tickAmount: 7,
                axisBorder: {
                  show: false
                },
                axisTicks: {
                  show: false
                },
                labels: {
                  rotate: 0
                }
              },

              yaxis: {
                show: false
              },

              tooltip: {
                custom: this.customTooltip('liquidity')
              },

              labels: new Array(7).fill('')
            },
            series: [{
              name: 'values',
              data: new Array(7).fill(0)
            }],
            tabs: [
              {
                name: "1W",
                period: 7
              },
              {
                name: "1M",
                period: 30,
              },
              {
                name: "All",
                period: false,
              }
            ],
            currentTabId: 0
          },

          volume: {
            options: {
              chart: {
                id: 'volumeChart',
                toolbar: {
                  show: false
                },
                fontFamily: 'inherit',
                foreColor: '#474c50',
                sparkline: {
                  enabled: true
                },
                offsetX: -4,
                animations: {
                  enabled: true,
                  easing: 'easeinout',
                  speed: 200,
                  animateGradually: {
                      enabled: true,
                      delay: 50
                  },
                  dynamicAnimation: {
                      enabled: true,
                      speed: 100
                  }
                },
              },

              dataLabels: {
                enabled: false
              },

              colors: ['#F643CF'],

              fill: {
                type: 'gradient',
                gradient: {
                  shade: 'dark',
                  type: "vertical",
                  shadeIntensity: 1,
                  gradientToColors: ['#EB7A4A'],
                  opacityFrom: 1,
                  opacityTo: 0.5,
                  stops: [0, 100],
                  colorStops: []
                }
              },

              plotOptions: {
                bar: {
                  borderRadius: 3,
                  horizontal: false,
                  columnWidth: '100%'
                },
              },

              grid: {
                show: false
              },
              
              xaxis: {
                tickPlacement: 'on',
                axisBorder: {
                  show: false
                },
                axisTicks: {
                  show: false
                },
                labels: {
                  show: false
                },
                categories: []
              },

              yaxis: {
                show: false
              },

              tooltip: {
                custom: this.customTooltip('volume')
              }
            },
            series: [{
              name: 'values',
              data: new Array(7).fill(0)
            }],
            tabs: [
              {
                name: "1W",
                period: 7
              },
              {
                name: "1M",
                period: 30,
              },
              {
                name: "All",
                period: false,
              }
            ],
            currentTabId: 0
          },

        },

        pair: {
          id: '0xceff51756c56ceffca006cd410b03ffc46dd3a58',
          latest: {
            date: false,
            volume: false,
            liquidity: false,
          },
          wbtc: {
            liquidity: false,
            swapPrice: false,
            usdPrice: false,
            icon: '<svg width="20" height="20" fill="none" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M9.998 20C4.476 19.999 0 15.522 0 10 0 4.476 4.477 0 10 0c5.522 0 9.999 4.476 10 9.998A10 10 0 0 1 9.998 20ZM9.996.778a9.215 9.215 0 1 0 .015 18.429A9.215 9.215 0 0 0 9.996.777Zm5.262 3.462a7.774 7.774 0 0 0-10.492 0l-.562-.549a8.56 8.56 0 0 1 11.603 0l-.549.55Zm.496.506.55-.55v-.012a8.56 8.56 0 0 1 0 11.603l-.55-.549a7.774 7.774 0 0 0 0-10.492ZM4.24 15.247a7.774 7.774 0 0 1 0-10.487l-.549-.549a8.56 8.56 0 0 0 0 11.604l.55-.568Zm.516.503a7.774 7.774 0 0 0 10.493 0l.561.55a8.56 8.56 0 0 1-11.603 0l.55-.55Zm8.68-7.583c-.109-1.146-1.098-1.53-2.348-1.648V4.941h-.966V6.49h-.773V4.941H8.39v1.59H6.43v1.034s.714-.013.703 0a.5.5 0 0 1 .549.424v4.35a.339.339 0 0 1-.117.236.335.335 0 0 1-.25.084c.013.01-.702 0-.702 0l-.183 1.155h1.942v1.614h.966v-1.59h.773v1.583h.968v-1.596c1.633-.099 2.771-.502 2.914-2.03.116-1.23-.463-1.78-1.387-2 .562-.277.91-.79.83-1.628Zm-1.352 3.44c0 1.103-1.734 1.078-2.525 1.066l-.188-.002v-2.13c.067 0 .149-.002.242-.004.813-.018 2.47-.055 2.47 1.07ZM9.536 9.566c.663.012 2.098.037 2.098-.967 0-1.029-1.39-.993-2.067-.975l-.196.003v1.937l.165.002Z" fill="#646464"/></svg>'
          },
          weth: {
            liquidity: false,
            swapPrice: false,
            usdPrice: false,
            icon: '<svg width="12" height="21" fill="none" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M10.815 9.455 5.92.954l-4.9 8.511 4.9-2.325 4.895 2.315Zm-4.898 4.559L.424 11.4 5.917 21l5.494-9.6-5.494 2.614Zm0-1.326-4.475-2.12 4.476-2.123 4.483 2.12-4.483 2.123Z" fill="#646464"/></svg>'
          },
          data: false
        },

        formatters: {
          usd: new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: 'USD',
          }),

          swapPrice: (x) => {
            return parseFloat(x).toFixed(4)
          },

          liquidity: (x) => {
            return parseFloat(x).toFixed(2)
          },

          dateFromUnix: (unixTime) => {
            return format(fromUnixTime(unixTime), 'LLL d')
          }
        },

        
      }
    },

    methods: {

      onChangeTab(e) {
        this.charts[e.chart].currentTabId = e.index
        this.setChartData(e.chart)
      },

      customTooltip(chartName) {
        return ({series, seriesIndex, dataPointIndex, w}) => {
          let selector = chartName === 'liquidity' ? 'reserveUSD' : 'volumeUSD'

          // set correct index
          dataPointIndex = this.charts[chartName].tabs[this.charts[chartName].currentTabId].period - dataPointIndex - 1
          // update HTML
          this.$refs[chartName + 'Date'].innerHTML = this.formatters.dateFromUnix(this.pair.data.dayData[dataPointIndex].date)
          this.$refs[chartName + 'Amount'].innerHTML = this.formatters.usd.format(this.pair.data.dayData[dataPointIndex][selector])
          return false
        }
      }, 

      setChartData(chartName) {
        this.pair.data.dayData = JSON.parse(localStorage.getItem('pair')).dayData
        const { currentTabId } = this.charts[chartName]
        const period = this.charts[chartName].tabs[currentTabId].period
        let selector = chartName === 'liquidity' ? 'reserveUSD' : 'volumeUSD'

        let options = this.charts[chartName].options
        options = { ...options,
          labels: this.pair.data.dayData.slice(0, period).map(el => this.formatters.dateFromUnix(el.date)).reverse()
        }

        this.$set(this.charts[chartName], 'options', options)
        this.$set(this.charts[chartName], 'series', [
          { 
            name: 'values',
            data: this.pair.data.dayData.slice(0, period).map(el => parseInt(el[selector], 10)).reverse()
          }
        ])
      },

      // async fetchFirstTimestamp() {
      //   try {
      //     const res = await axios.post(this.api.url, {
      //       headers: {
      //         'Content-Type': 'application/json'
      //       },
      //       query: `
      //         {
      //           pair(id: "${this.pair.id}") {
      //             dayData(first: 1) {
      //               date
      //             }
      //           }
      //         }
      //       `
      //     })
      //     console.log(res.data.data.pair.dayData[0].date)
      //   } catch (err) {
      //     console.error(err)
      //   }
      // },

      // NOTE: hardcoded res.data.data.pair.dayData[0].date, to reduce requests

      async fetchData() {
        try {
          const diff = differenceInDays(new Date(), fromUnixTime(this.api.firstTimestamp))

          this.charts.volume.tabs[this.charts.volume.tabs.length - 1].period =
          this.charts.liquidity.tabs[this.charts.liquidity.tabs.length - 1].period = diff

          // get data from localStorage to reduce network load
          if (localStorage.getItem('fetchDate') === this.formatters.dateFromUnix(getUnixTime(new Date())) && localStorage.getItem('pair')) {
            this.$set(this.pair, 'data', JSON.parse(localStorage.getItem('pair')))

          } else {
            const res = await axios.post(this.api.url, {
              headers: {
                'Content-Type': 'application/json'
              },
              query: `
                {
                  pair(id: "${this.pair.id}") {
                    token0Price
                    token1Price
                    dayData (first:${diff}, orderBy: date, orderDirection: desc){
                      date
                      volumeUSD
                      reserveUSD
                    }
                    token0 {
                      name
                      liquidity
                      dayData (first:1, orderBy: date, orderDirection: desc) {
                        date
                        priceUSD
                      }
                    }
                    token1 {
                      name
                      liquidity
                      dayData (first:1, orderBy: date, orderDirection: desc) {
                        date
                        priceUSD
                      }
                    }
                  }
                }
              `
            })
            
            // save data from API
            this.$set(this.pair, 'data', res.data.data.pair)
            localStorage.setItem('pair', JSON.stringify(this.pair.data))
            localStorage.setItem('fetchDate', this.formatters.dateFromUnix(getUnixTime(new Date())))
          }

          this.pair.wbtc.swapPrice = this.formatters.swapPrice(this.pair.data.token1Price)
          this.pair.weth.swapPrice = this.formatters.swapPrice(this.pair.data.token0Price)
          
          this.pair.wbtc.liquidity = this.formatters.liquidity(this.pair.data.token0.liquidity)
          this.pair.weth.liquidity = this.formatters.liquidity(this.pair.data.token1.liquidity)

          this.pair.wbtc.usdPrice = this.formatters.usd.format(this.pair.data.token0.dayData[0].priceUSD)
          this.pair.weth.usdPrice = this.formatters.usd.format(this.pair.data.token1.dayData[0].priceUSD)

          this.pair.latest.date = this.formatters.dateFromUnix(this.pair.data.dayData[0].date)
          this.pair.latest.volume = this.formatters.usd.format(this.pair.data.dayData[0].volumeUSD)
          this.pair.latest.liquidity = this.formatters.usd.format(this.pair.data.dayData[0].reserveUSD)

        } catch (err) {
          console.error(err)
        }
      }
    },

    async mounted() {
      await this.fetchData()
      this.setChartData('volume')
      this.setChartData('liquidity')
    }
  }
</script>


<style lang="scss">
  @import "./scss/index.scss";
</style>
