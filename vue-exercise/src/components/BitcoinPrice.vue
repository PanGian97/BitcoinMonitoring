<template>
    <div class="price-container">
        <p class="price-text">Current Bitcoin Price in {{ storedCurrency }}: {{ bitcoinPrice }}</p>
        <button @click="changeCurrency('EUR')">EUR</button>
        <button @click="changeCurrency('USD')">USD</button>
        <p class="average-text">Average (last 3): {{ avgLast3 }} {{ storedCurrency }}</p>
        <p class="average-text">Average (last 5): {{ avgLast5 }} {{ storedCurrency }}</p>
        <p class="average-text">Average (last 8): {{ avgLast8 }} {{ storedCurrency }}</p>
        <apexchart type="line" height="350" :options="chartOptions" :series="chartSeries"></apexchart>
    </div>
</template>
  
<script>
import axios from "axios";
import VueApexCharts from "vue3-apexcharts";

export default {
    components: {
        apexchart: VueApexCharts,
    },
    name: "BitcoinPrice",
    data() {
        return {
            storedCurrency:null,
            bitcoinPrice: null,
            priceHistory: [],
            avgLast3: null,
            avgLast5: null,
            avgLast8: null,
            chartOptions: {
                chart: {
                    id: "bitcoin-price",
                },
                xaxis: {
                    categories: [],
                },
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
                    console.log(this.bitcoinPrice)

                    this.priceHistory.unshift({
                        time: new Date().toLocaleTimeString(),
                        price: this.bitcoinPrice
                    });


                    if (this.priceHistory.length > 20) {
                        this.priceHistory.pop();
                    }
                    // Calculate the averages
                    if (this.priceHistory.length >= 3) {
                        this.avgLast3 = this.calculateAverage(3);
                    }
                    if (this.priceHistory.length >= 5) {
                        this.avgLast5 = this.calculateAverage(5);
                    }
                    if (this.priceHistory.length >= 8) {
                        this.avgLast8 = this.calculateAverage(8);
                    }
                    this.updateChart()
                })
                .catch((error) => {
                    console.error("There was an error fetching the data!", error);
                });
        },
        changeCurrency(currency) {
            localStorage.setItem('selectedCurrency', currency); // Save selected currency to localStorage
            location.reload(); // Refresh the page
        },
        calculateAverage(timestampNum) {
            let sum = 0
            console.log(this.priceHistory[0].price)
            for (let i = 0; i < timestampNum; i++) {
                sum += parseFloat(this.priceHistory[i].price);
            }

            return (sum / timestampNum).toFixed(2);  // 2 decimal places
        },
        updateChart() {
            this.chartOptions.xaxis.categories = this.priceHistory.map(
                (pricePoint) => pricePoint.time
            );
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
  
<style scoped>
body {
    margin: 0;
    padding: 0;
}

.price-container {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 50vh;
    background-color: white;
}


.price-text {
    font-size: 32px;
    color: black;
}
</style>
  