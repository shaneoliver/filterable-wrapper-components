<template>
    <div class="filterable-cards">

        <div class="card mb-12">
            <div class="card-body">
                <div class="row">
                    <div class="col-12 col-md-6 col-lg-3" v-if="searchable">
                        <input type="search" v-model="filters.search" class="form-control col-12" placeholder="search">
                    </div>

                    <div class="col-12 col-md-3 col-lg-2 px-2" v-if="filters.sort">
                        <select class="custom-select" v-model="filters.sort">
                            <option value="" disabled>Sort by</option>
                            <option v-for="(column, key, index) in sortColumns" :value="key">{{ column }}</option>
                        </select>
                    </div>

                    <div class="px-2" v-if="filters.sort">
                        <button class="btn btn-light" @click="changeOrder">
                            <span class="fad"
                                :class="{
                                    'fa-sort-alpha-down': filters.order === 'asc',
                                    'fa-sort-alpha-down-alt': filters.order === 'desc',
                                }"
                            ></span>
                        </button>
                    </div>

                    <slot name="filters"></slot>

                    <div class="text-last-right px-4 w-auto ml-auto">
                        <select class="custom-select text-right" v-model.number="filters.count">
                            <option>12</option>
                            <option>24</option>
                            <option>48</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="loaded && results.length === 0">
            <div class="alert alert-info">
                There are no results for your query <strong v-if="filters.search">{{ filters.search }}</strong>
            </div>
        </div>

        <div class="filter-results row" v-if="results.length > 0 && loaded">
            <template v-for="(result, index) in results">
                <slot :result="result"></slot>
            </template>
        </div>

        <so-pagination :pagination="pagination" @pageChange="pageChange"></so-pagination>
    </div>
</template>

<script>
import { debounce } from 'lodash';
import SoPagination from './SoPagination';

export default {

    components: {
        SoPagination,
    },

    props: {
        endpoint: {
            type: String,
            default: '',
        },
        method: {
            type: String,
            default: 'get',
        },
        sortColumns: {
            type: Object,
            default: () => {
                return {}
            }
        },
        search: {
            type: String,
            default: '',
        },
        count: {
            type: Number,
            default: 24,
        },
        page: {
            type: Number,
            default: 1,
        },
        sort: {
            type: String,
            default: '',
        },
        order: {
            type: String,
            default: 'asc',
        },
        extendedFilters: {
            type: Object,
            default: () => {}
        },
        searchable: {
            default: true,
        }
    },

    data() {
        return {
            results: [],
            pagination: {},
            filters: {},
            loaded: false,
        }
    },

    watch: {
        'filters.search': {
            handler() {
                this.filters.page = 1;
                this.asyncFetch();
            },
        },
        'filters.page': {
            handler() {
                this.fetch();
            }
        },
        'filters.sort': {
            handler() {
                this.fetch();
            }
        },
        'filters.order': {
            handler() {
                this.fetch();
            }
        },
        'filters.count': {
            handler() {
                this.filters.page = 1;
                this.fetch();
            }
        },
        extendedFilters: {
            handler() {
                this.asyncFetch();
            },
            deep: true,
        }
    },

    mounted() {
         this.filters = {
            search: this.search,
            count: this.count,
            page: this.page,
            sort: this.sort,
            order: this.order
        };

        this.fetch();
    },

    methods: {
        asyncFetch: debounce(function () {
            this.loaded = false;
            this.fetch();
        }, 250),

        fetch() {
            this.loaded = false;
            axios.get(this.endpoint, { params: { ...this.filters, ...this.extendedFilters } })
                .then(response => {
                    this.results = response.data.data;
                    this.pagination = response.data;
                })
                .finally(() => {
                    this.loaded = true;
                })
        },

        pageChange(payload) {
            this.filters.page = payload;
        },

        changeOrder() {
            this.filters.order = this.filters.order === 'asc' ? 'desc' : 'asc';
        }
    },
}
</script>

