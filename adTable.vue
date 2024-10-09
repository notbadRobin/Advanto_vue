<template>
    <div>

        <div class="d-flex justify-content-evenly">
            <select v-model="selectedMonth" @change="fetchPaginatedData(), fetchStats()" ange="fetchPaginatedData" class="form-select w-25" aria-label="Default select example">
                <option v-for="(monthName, monthNumber) in monthObject" :key="monthNumber" :value="monthNumber">
                    {{ monthName }}
                </option>
            </select>

            <h5>Page: {{ currentPage }}</h5>
            
            <div class="d-flex justify-content-center">
                <input v-model="searchQuery" type="text" placeholder="Search products" @keyup.enter="searchProducts(this.currentPage)"  class="form-control" aria-label="Sizing example input" aria-describedby="inputGroup-sizing-default">          
                <button @click="searchProducts(this.currentPage)" type="button" class="btn btn-primary">Search</button>
                <button @click="clear()" type="button" class="btn btn-success">Clear</button>
            </div>
    
            <div v-if="error" class="error">{{ error }}</div>
        
            <div v-else-if="results && !results.length">
                <p>No results found.</p>
            </div>
        </div>

        <div v-if="error" class="error">{{ error }}</div><br><br>
  
        <div v-if="data">
            <table class="table table-striped table-bordered">
                <thead>
                    <tr class="table-dark">
                        <th style="text-transform: uppercase;" v-for="(d, ind) in Object.keys(data[0])" :key="ind">{{ d }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr class="table-info" v-for="(d, ind) in data" :key="ind">
                        <td>{{ d.id }}</td>
                        <td>{{ d.title }}</td>
                        <td>{{ d.price }}</td>
                        <td>{{ d.description }}</td>
                        <td>{{ d.category }}</td>
                        <td>{{ d.image }}</td>
                        <td>{{ d.sold }}</td>
                        <td>{{ d.dateOfSale }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div>
            <div class="d-flex justify-content-evenly">
                <button @click="fetchPrevData()" type="button" class="btn btn-warning">Previous</button>
                <div>
                    <h6>
                        Page No : {{ currentPage }}
                    </h6>
                </div>
                <button @click="fetchNextData()" type="button" class="btn btn-warning">Next</button>
            </div>
        </div>
        <br><br>
        <div v-for="(monthName, monthNumber) in monthObject" :key="monthNumber">
            <div v-if="selectedMonth == monthNumber">
                <h5>
                    Statistics - {{ monthName }}
                </h5> 
            </div>
        </div>
        <div class="d-flex justify-content-center rounded">
            <div class="w-50">
                <table class="table table-striped table-bordered">
                    <tbody>
                        <tr>
                            <td class="table-dark">Total Sale</td>
                            <td class="table-info">{{ statistics.total }}</td>
                        </tr>
                        <tr>
                            <td class="table-dark">Total Sold Item</td>
                            <td class="table-info">{{ statistics.itemSold }}</td>
                        </tr>
                        <tr>
                            <td class="table-dark">Total Not Sold Items</td>
                            <td class="table-info">{{ statistics.itemNotSold }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </div>
</template>
  
<script>
    import axios from 'axios';
  
    export default {
        data() {
            return {
                data: null,
                currentPage: 1,
                selectedMonth: 3, 
                monthObject: {
                    1: 'January',
                    2: 'February',
                    3: 'March',
                    4: 'April',
                    5: 'May',
                    6: 'June',
                    7: 'July',
                    8: 'August',
                    9: 'September',
                    10: 'October',
                    11: 'November',
                    12: 'December'
                },
                error: null,
                searchQuery: '',
                statistics: {
                    total: 0,
                    itemSold: 0,
                    itemNotSold: 0
                }
            };
        },
        methods: {
            clear() {
                this.searchQuery = ''; 
                this.selectedMonth = 3;
                this.fetchPaginatedData()
            },

            fetchStats() {
                this.fetchTotalAmt();
                this.fetchSoldItems();
                this.fetchNotSoldItems();
            },

            async fetchNextData() {
                var page = this.currentPage + 1;
                await this.searchProducts(page);
            },

            async fetchPrevData() {
                var page = this.currentPage - 1;
                this.searchProducts(page);
            },

            async fetchTotalAmt() {
                const month = this.selectedMonth;
        
                await axios.get(`http://localhost:3000/prices?month=${month}`)
                .then(response => {
                    console.log(response.data)
                    this.statistics.total = response.data.total;
                    console.log(this.statistics.total);

                })
                .catch(error => {
                    console.error('Error fetching data:', error);
                    this.error = 'Failed to fetch data.';
                });
            },

            async fetchSoldItems() {
                const month = this.selectedMonth;
        
                await axios.get(`http://localhost:3000/sold-items?month=${month}`)
                .then(response => {
                    console.log(response.data)
                    this.statistics.itemSold = response.data.totalSoldItems;
                    console.log(this.statistics.itemSold);

                })
                .catch(error => {
                    console.error('Error fetching data:', error);
                    this.error = 'Failed to fetch data.';
                });
            },

            async fetchNotSoldItems() {
                const month = this.selectedMonth;
        
                await axios.get(`http://localhost:3000/not-sold-items?month=${month}`)
                .then(response => {
                    console.log(response.data)
                    this.statistics.itemNotSold = response.data.totalNotSoldItems;
                    console.log(this.statistics.itemNotSold);

                })
                .catch(error => {
                    console.error('Error fetching data:', error);
                    this.error = 'Failed to fetch data.';
                });
            },

            async searchProducts(page) {
                this.selectedMonth = 3;
                if (this.searchQuery.trim() === '') {
                    this.error = 'Please enter a search term';
                    return;
                }
                this.error = null;
    
                await axios.get(`http://localhost:3000/search?q=${this.searchQuery}&page=${page}`)
                .then(response => {
                    this.data = response.data.data; // Set the results from the response
                    this.currentPage = response.data.currentPage;
                    console.log(response.data);
                })
                .catch(error => {
                    console.error('Error fetching search results:', error);
                    this.error = 'Failed to fetch search results.';
                });
            },
            
            fetchPaginatedData() {
                const month = this.selectedMonth || 3;
                this.currentPage = 1
  
                axios.get(`http://localhost:3000/paginate?page=${this.currentPage}&month=${month}`)
                .then(response => {
                    this.data = response.data.data;
                    this.currentPage = response.data.currentPage;
                })
                .catch(error => {
                    console.error('Error fetching data:', error);
                    this.error = 'Failed to fetch data.';
                });
            }
        },
        async mounted() {
            this.fetchPaginatedData();
            this.fetchTotalAmt();
            this.fetchSoldItems();
            this.fetchNotSoldItems();
        }
    };
</script>
  
  