<template>
    <div id="app">
        <h1>Cryptocurrencies</h1>
        <table>
            <head></head>
            <body>
                <tr>
                    <th>Сryptocurrency</th>
                    <th>Price ($)</th>
                </tr>
                <tr v-for="item in allCoins" :key="item.id">
                    <td>{{item.fullName}}</td>
                    <td>{{item.price | currencyFilter}}</td>
                </tr>
            </body>
        </table>
    </div>
</template>

<script>
    import axios from 'axios';

    export default {
        name: 'Crypto',
        data() {
            return {
                baseData: {
                    urlTop: 'https://min-api.cryptocompare.com/data/top/mktcapfull?',
                    urlAll: 'https://min-api.cryptocompare.com/data/pricemulti?',
                    apiKey: '4671b313962b7599acb6ca48966e1323857bae63ddb003c5320ced06935955a4',
                    current: 'USD',
                    page: 0,
                    limitTop: 100,
                    limitAll: 300,
                    updateTime: 10000,
                },
                allCoins: [],
            };
        },
        methods: {
            getData() {
                axios
                    .get(this.baseData.urlTop + 'limit=' + this.baseData.limitTop + '&tsym=' + this.baseData.current + '&page=' + this.baseData.page + '&api_key=' + this.baseData.apiKey)
                    .then(response => {
                        let addCoins = [];

                        Object.entries(response.data.Data).forEach(([key, elem]) => {
                            if (!elem.RAW) return;

                            addCoins.push(
                                {
                                    id: elem.CoinInfo.Id,
                                    fullName: elem.CoinInfo.FullName,
                                    name: elem.CoinInfo.Name,
                                    price: elem.RAW.USD.PRICE,
                                }
                            );
                        });

                        this.allCoins = [...this.allCoins, ...this.sortData(addCoins)];
                    })
                    .catch(error => {
                        console.log(error);
                    })
            },
            updateData() {
                let stringOfCurrency = '';
                let arrOfCurrency = [];
                let arrOfString = [];

                Object.entries(this.allCoins).forEach(([key, value]) => {
                    stringOfCurrency += value.name + ',';
                });

                arrOfCurrency = stringOfCurrency.split(',');
                arrOfString.push('');

                arrOfCurrency.forEach(elem => {
                    if (arrOfString[arrOfString.length - 1].length + elem.length + 1 > this.baseData.limitAll) {
                        arrOfString.push(elem);
                    } else {
                        if (arrOfString[arrOfString.length - 1].length)
                            arrOfString[arrOfString.length - 1] += ',';

                        arrOfString[arrOfString.length - 1] += elem;
                    }
                });

                arrOfString.forEach(elem => {
                    axios
                        .get(this.baseData.urlAll + 'fsyms=' + elem + '&tsyms=USD' + '&api_key=' + this.baseData.apiKey)
                        .then(response => {
                            let arrOfUpdateStr = elem.split(',');
                            for (let i = 0; i < this.allCoins.length; i++) {
                                for (let i = 0; i < arrOfUpdateStr.length; i++) {
                                    if (this.allCoins[i].name === arrOfUpdateStr[i]) {
                                        this.allCoins[i].price = response.data[arrOfUpdateStr[i]][this.baseData.current];
                                    }
                                }
                            }
                        })
                        .catch(error => {
                            console.log(error);
                        })
                });
            },
            sortData(arr) {
                return arr.sort((a, b) => {
                    return b.price - a.price;
                })
            },
            scrollList() {
                if (document.documentElement.scrollTop + window.innerHeight === document.documentElement.offsetHeight) {
                    this.baseData.page++;
                    this.getData();
                }
            },
        },
        filters: {
            currencyFilter(value) {
                return value.toFixed(2);
            }
        },
        mounted() {
            setInterval(() => {
                this.updateData();
            }, this.baseData.updateTime);
        },
        created() {
            this.getData();
            window.addEventListener('scroll', this.scrollList);
        },
        destroyed() {
            window.removeEventListener('scroll', this.scrollList);
        }
    }
</script>

<style lang="scss">
    table {
        margin: 0 auto;
        border-collapse: collapse;
        td, th {
            padding: 0.5rem;
            border: 1px solid black;
        }
        th {
            background-color: #42b983;
        }
    }
</style>

