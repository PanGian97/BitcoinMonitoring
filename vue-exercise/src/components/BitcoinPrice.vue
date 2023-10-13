<template>
    <div class="price-container">
        <div class="header">
            <p class="price-text">Current Bitcoin Price: {{ bitcoinPrice }} {{ storedCurrency }}</p>
            <div class="currency-buttons">
                <button @click="changeCurrency('EUR')" :class="{ 'btn-active': storedCurrency === 'EUR' }">EUR</button>
                <button @click="changeCurrency('USD')" :class="{ 'btn-active': storedCurrency === 'USD' }">USD</button>
            </div>
        </div>

        <div class="average-text">
            <h3>Average Price Last:</h3>
            <div class="timeframe">
                <span>5 minutes: </span>
                <span v-if="avgLast5 !== null">{{ avgLast5 }} {{ storedCurrency }}</span>
                <span v-else class="avg-no-data">Not enough data yet</span>

            </div>
            <div class="timeframe">
                <span>30 minutes: </span>
                <span v-if="avgLast5 !== null">{{ avgLast30 }} {{ storedCurrency }}</span>
                <span v-else class="avg-no-data">Not enough data yet</span>
            </div>
            <div class="timeframe">
                <span>60 minutes: </span>
                <span v-if="avgLast5 !== null">{{ avgLast60 }} {{ storedCurrency }}</span>
                <span v-else class="avg-no-data">Not enough data yet</span>
            </div>
        </div>

        <apexchart type="line" height="350" :options="chartOptions" :series="chartSeries"></apexchart>
    </div>
</template>

  
<script>
import axios from "axios";
import VueApexCharts from "vue3-apexcharts";
import '@/assets/styles.css';
export default {
    components: {
        apexchart: VueApexCharts,
    },
    name: "BitcoinPrice",
    data() {
        return {
            storedCurrency: 'USD',//default
            bitcoinPrice: null,
            priceHistory: [],
            avgLast5: null,
            avgLast30: null,
            avgLast60: null,
            chartOptions: {
                chart: {
                    id: "bitcoin-price",
                },
                xaxis: {
                    type: 'datetime',
                    categories: [],
                    labels: {

                        datetimeUTC: false,
                        format: ' MMM dd->HH:mm:ss ',
                        show: true
                    }
                },
                tooltip: {
                    x: {
                        format: "HH:mm:ss"
                    }
                }
            },
            chartSeries: [
                {
                    name: "Bitcoin Price",
                    data: [],
                },
            ],
        };
    },
    methods: {
        fetchBitcoinPrice() {
            axios
                .get(`https://api.coinbase.com/v2/prices/spot?currency=${this.storedCurrency}`)
                .then((response) => {
                    this.bitcoinPrice = response.data.data.amount;
                    this.priceHistory.push({
                        time: new Date().getTime(), // Current timestamp
                        price: this.bitcoinPrice
                    });

                    if (this.priceHistory.length > 60) {
                        this.priceHistory.shift();
                    }
                    // Calculate the averages
                    if (this.priceHistory.length >= 5) {
                        this.avgLast5 = this.calculateAverage(5);
                    }
                    if (this.priceHistory.length >= 30) {
                        this.avgLast30 = this.calculateAverage(30);
                    }
                    if (this.priceHistory.length >= 60) {
                        this.avgLast60 = this.calculateAverage(60);
                    }
                    this.updateChart()
                })
                .catch((error) => {
                    console.error("There was an error fetching the data!", error);
                });
        },
        calculateAverage(numOfValues) {
            let sum = 0;

            for (let i = this.priceHistory.length - 1; i >= this.priceHistory.length - numOfValues; i--) {
                sum += parseFloat(this.priceHistory[i].price);
            }

            return (sum / numOfValues).toFixed(2);  // 2 decimal places
        },
        changeCurrency(currency) {
            localStorage.setItem('selectedCurrency', currency);
            location.reload();
        },

        updateChart() {
            this.chartOptions = {
                ...this.chartOptions,
                xaxis: {
                    ...this.chartOptions.xaxis,
                    categories: this.priceHistory.map(pricePoint => pricePoint.time)
                },
           };
           //this.chartOptions.xaxis.categories = this.priceHistory.map(pricePoint => pricePoint.time);  Shouldn't this also work?
            this.chartSeries[0].data = this.priceHistory.map((pricePoint) =>
                parseFloat(pricePoint.price)
            );
        }
    },
    mounted() {
        this.storedCurrency = localStorage.getItem('selectedCurrency');
        console.log(this.storedCurrency)
        this.fetchBitcoinPrice();
        this.interval = setInterval(() => {
            this.fetchBitcoinPrice();
        }, 8000);
    },
    beforeUnmount() {
        clearInterval(this.interval);
    },
};
</script>
  