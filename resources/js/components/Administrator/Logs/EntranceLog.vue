<template>
    <div>
        <div class="section">
            <div class="columns is-centered">
                <div class="column is-10">
                    <div class="box">

                        <div class="is-flex is-justify-content-center mb-2" style="font-size: 20px; font-weight: bold;">ENTRANCE LOGS</div>


                        <div class="columns">
                            <div class="column">
                                <b-field label="Page">
                                    <b-select v-model="perPage" @input="setPerPage">
                                        <option value="5">5 per page</option>
                                        <option value="10">10 per page</option>
                                        <option value="15">15 per page</option>
                                        <option value="20">20 per page</option>
                                    </b-select>
                                    <b-select v-model="sortOrder" @input="loadAsyncData">
                                        <option value="asc">ASC</option>
                                        <option value="desc">DESC</option>

                                    </b-select>
                                </b-field>
                            </div>

                            <div class="column">
                                <b-field label="Search">
                                    <b-input type="text"
                                                v-model="search.fullname" placeholder="Search Name"
                                                @keyup.native.enter="loadAsyncData"/>
                                    <p class="control">
                                            <b-tooltip label="Search" type="is-success">
                                        <b-button type="is-primary" icon-right="account-filter" @click="loadAsyncData"/>
                                            </b-tooltip>
                                    </p>
                                </b-field>
                            </div>
                        </div>

                        <div class="columns">
                            <div class="column">
                                <b-field label="Select Date">
                                    <b-datepicker
                                        v-model="search.start_date" placeholder="Start Date"/>

                                        <b-datepicker
                                            v-model="search.end_date" placeholder="End Date"/>
                                </b-field>
                            </div>
                        </div>
                        

                        <b-table
                            :data="data"
                            :loading="loading"
                            paginated
                            backend-pagination
                            :total="total"
                            :per-page="perPage"
                            @page-change="onPageChange"
                            aria-next-label="Next page"
                            aria-previous-label="Previous page"
                            aria-page-label="Page"
                            aria-current-label="Current page"
                            backend-sorting
                            :default-sort-direction="defaultSortDirection"
                            @sort="onSort">

                            <div class="buttons">
                                <b-button label="Print Preview" icon-left="printer"
                                @click="printPreview"
                                type="is-info"></b-button>
                            </div>

                            <b-table-column field="user_id" label="Student Id" v-slot="props">
                                {{ props.row.student_id }}
                            </b-table-column>

                            <b-table-column field="fullname" label="Name" v-slot="props">
                                {{ props.row.fullname }}
                            </b-table-column>

                            <b-table-column field="program" label="Program" v-slot="props">
                                {{ props.row.program }}
                            </b-table-column>

                            <b-table-column field="year_level" label="Year Level" centered v-slot="props">
                                {{ props.row.year_level }}
                            </b-table-column>

                            <b-table-column field="contact_no" label="Contact No." v-slot="props">
                                {{ props.row.contact_no }}
                            </b-table-column>

                            <b-table-column field="date_entry" label="Date Entry" v-slot="props">
                                {{ props.row.date_entry }}
                            </b-table-column>
                            
                        </b-table>
                    </div>
                </div><!--col -->
            </div><!-- cols -->
        </div><!--section div-->



    </div>
</template>

<script>

export default{
    data() {
        return{
            data: [],
            total: 0,
            loading: false,
            sortField: 'entrance_log_id',
            sortOrder: 'desc',
            page: 1,
            perPage: 15,
            defaultSortDirection: 'asc',


            search: {
                fullname: '',
                start_date: new Date(new Date().getFullYear(), new Date().getMonth(), 1),
                end_date: new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0)
            },

            isModalCreate: false,

            fields: {},
            errors: {},
            

            btnClass: {
                'is-success': true,
                'button': true,
                'is-loading':false,
            },

           
        }

    },

    methods: {
        /*
        * Load async data
        */
        loadAsyncData() {
            const params = [
                `sort_by=${this.sortField}.${this.sortOrder}`,
                `fulname=${this.search.fullname}`,
                `start=${this.$formatDate(this.search.start_date)}`,
                `end=${this.$formatDate(this.search.end_date)}`,
                `perpage=${this.perPage}`,
                `page=${this.page}`
            ].join('&')

            this.loading = true
            axios.get(`/get-entrance-logs?${params}`)
                .then(({ data }) => {
                    this.data = [];
                    let currentTotal = data.total
                    if (data.total / this.perPage > 1000) {
                        currentTotal = this.perPage * 1000
                    }

                    this.total = currentTotal
                    data.data.forEach((item) => {
                        //item.release_date = item.release_date ? item.release_date.replace(/-/g, '/') : null
                        this.data.push(item)
                    })
                    this.loading = false
                })
                .catch((error) => {
                    this.data = []
                    this.total = 0
                    this.loading = false
                    throw error
                })
        },
        /*
        * Handle page-change event
        */
        onPageChange(page) {
            this.page = page
            this.loadAsyncData()
        },

        onSort(field, order) {
            this.sortField = field
            this.sortOrder = order
            this.loadAsyncData()
        },

        setPerPage(){
            this.loadAsyncData()
        },

        printPreview(){
            const params = [
                `fulname=${this.search.fullname}`,
                `start=${this.$formatDate(this.search.start_date)}`,
                `end=${this.$formatDate(this.search.end_date)}`,
            ].join('&')

            window.location = '/entrance-logs-print-preview?' + params
        }

    },

    mounted() {
        this.loadAsyncData()
    }
}
</script>


<style>


</style>
