<template>
    <nav class="mt-4" aria-label="Datatable pagination" v-if="pagination.last_page > 1">
        <ul class="pagination">
            <li class="page-item" :class="{ disabled: pagination.current_page === 1 }">
                <a class="page-link"
                    @click="pageClicked( pagination.current_page - 1 )">
                    Previous
                </a>
            </li>
            <li v-if="hasFirst" class="page-item" :class="{ active: isActive(1) }">
                <a class="page-link" @click="pageClicked(1)">1</a>
            </li>
            <li class="page-item disabled" v-if="hasFirstEllipsis">
                <a href="" class="page-link">
                    ...
                </a>
            </li>
            <li class="page-item" :class="{ active: isActive(page), disabled: page === '...' }" v-for="page in pages" :key="page">
                <a class="page-link" @click="pageClicked(page)">{{ page }}</a>
            </li>
            <li class="page-item disabled" v-if="hasLastEllipsis">
                <a href="" class="page-link">
                    ...
                </a>
            </li>
            <li v-if="hasLast && pagination.last_page > 1" class="page-item"
                :class="{ active: isActive(this.pagination.last_page) }">
                <a class="page-link" @click="pageClicked(pagination.last_page)">{{pagination.last_page}}</a>
            </li>
            <li class="page-item" :class="{ disabled: pagination.current_page === pagination.last_page }">
                <a class="page-link"
                    @click="pageClicked( pagination.current_page + 1 )">
                    Next
                </a>
            </li>
        </ul>
    </nav>
</template>

<script>

    export default {
        props: {
            pagination: {
                type: Object,
                default: () => ({}),
            },
        },

        computed: {
            pages() {
                return this.pagination.last_page === undefined
                    ? []
                    : this.pageLinks();
            },

            hasFirst() {
                return this.pagination.current_page >= 4 || this.pagination.last_page < 10
            },

            hasLast() {
                return this.pagination.current_page <= this.pagination.last_page - 3 || this.pagination.last_page < 10
            },

            hasFirstEllipsis() {
                return this.pagination.current_page >= 4 && this.pagination.last_page >= 10
            },

            hasLastEllipsis() {
                return this.pagination.current_page <= this.pagination.last_page - 3 && this.pagination.last_page >= 10
            },

            shouldShowPagination() {
                if (this.pagination.last_page === undefined) {
                    return false;
                }

                if (this.pagination.count === 0) {
                    return false;
                }

                return this.pagination.last_page > 1;
            },

        },

        methods: {
            isActive(page) {
                const current_page = this.pagination.current_page || 1;

                return current_page === page;
            },

            pageClicked(page) {
                if (page === '...' ||
                    page === this.pagination.current_page ||
                    page > this.pagination.last_page ||
                    page < 1) {
                    return;
                }

                this.$emit('pageChange', page);
            },

            pageLinks() {
                const pages = [];

                let left = 2;
                let right = this.pagination.last_page - 1;

                if (this.pagination.last_page >= 10) {
                    left = Math.max(1, this.pagination.current_page - 2);
                    right = Math.min(this.pagination.current_page + 2, this.pagination.last_page);
                }

                for (let i = left; i <= right; i++) {
                    pages.push(i);
                }

                return pages;
            },
        },
    };
</script>
