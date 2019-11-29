<template>
    <div class="table-component">
        <div class="card">
            <div class="card-header">
                <div class="row">
                    <div class="w-auto col-lg-6" v-if="searchable">
                        <input type="search" v-model="filters.search" class="form-control col-12" placeholder="search">
                    </div>
                    <slot name="filters"></slot>
                    <div class="w-auto px-4 w-auto text-last-right ml-auto">
                        <select class="form-control" v-model.number="filters.count">
                            <option>10</option>
                            <option>25</option>
                            <option>50</option>
                        </select>
                    </div>
                </div>
            </div>
            <div class="card-body p-0">
                <div class="m-4 alert alert-danger" v-if="rows.length === 0 && loaded">
                    There are no results for your query <strong v-if="filters.search">{{ filters.search }}</strong>
                </div>

                <div class="p-4 text-center" v-if="! loaded">
                    <span class="fad fa-spinner-third fa-spin text-2xl text-center"></span>
                </div>

                <div class="table-responsive" v-if="rows.length > 0 && loaded">
                    <table class="table table-striped mb-0">
                        <thead>
                            <tr>
                                <slot name="header" :sort="trySort" :icon="getSortIconClasses"></slot>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(row, index) in rows" @click="rowClicked(row)">
                                <slot :row="row"></slot>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
        <so-pagination :pagination="pagination" @pageChange="pageChange"></so-pagination>
    </div>
</template>

<script>
import _ from 'lodash';
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
        filters: {
            type: Object,
            default: () => {
                return {
                    search: '',
                    count: 10,
                    page: 1,
                    sort: '',
                    order: 'asc',
                }
            }
        },
        extendedFilters: {
            type: Object,
            default: () => {}
        },
        searchable: {
            default: true,
        },
    },

    data() {
        return {
            rows: [],
            pagination: {},
            mergedFilters: {},
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
        this.fetch();
    },

    methods: {
        asyncFetch: _.debounce(function () {
            this.loaded = false;
            this.fetch();
        }, 250),

        fetch() {
            this.loaded = false;
            axios.get(this.endpoint, { params: {...this.filters, ...this.extendedFilters} })
                .then(response => {
                    this.rows = response.data.data;
                    this.pagination = response.data;
                })
                .finally(() => {
                    this.loaded = true;
                })
        },

        rowClicked(row) {
            this.$emit('rowClicked', row);
        },

        pageChange(payload) {
            this.filters.page = payload;
        },

        trySort(key) {
            if(this.filters.sort === key) {
                this.filters.order = this.filters.order === 'asc' ? 'desc' : 'asc';
            }

            this.filters.sort = key;
        },

        getSortIconClasses(key) {
            let classes = key === this.filters.sort ? 'fas' : 'fad';

            classes += this.filters.order === 'asc' ? ' fa-sort-up' : ' fa-sort-down';

            return classes;
        }
    },
}
</script>

