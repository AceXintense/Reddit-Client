<template>
    <div>
        <loader v-if="!listings"></loader>

        <listing-size @sizeChanged="changeSize"></listing-size>

        <div class="card">
            <div :class="{'card-header': true, 'transparent': transparency}">
                <h1>{{ subreddit }}</h1>
                <button class="btn btn-primary settings-btn" @click="toggleSettings()">
                    <i class="fa fa-sliders" aria-hidden="true"></i>
                </button>
            </div>
            <div class="view" v-if="listings">
                <loader v-if="listings.length <= 0"></loader>
                <div class="subreddit" v-for="data in listings">
                    <div class="row">
                        <div class="col-lg-2">
                            <div v-if="thumbnailURLIsValid(data.data.thumbnail)">
                                <a :href="data.data.thumbnail" target="_blank">
                                    <img :src="data.data.thumbnail" alt="">
                                </a>
                            </div>
                            <div v-if="!thumbnailURLIsValid(data.data.thumbnail)">
                                <p class="no-thumbnail">No Thumbnail</p>
                            </div>
                        </div>
                        <div class="col-lg-8" @click="selectListing(data.data)">
                            <p>{{data.data.title}}</p>
                            <p class="author">{{data.data.author}}</p>
                        </div>
                        <div class="col-lg-2" @click="selectListing(data.data)">
                            <div class="meta">
                                <div class="up">
                                    <i class="fa fa-chevron-up" aria-hidden="true"></i>
                                    <span>{{data.data.ups}}</span>
                                </div>
                                <div class="down">
                                    <i class="fa fa-chevron-down" aria-hidden="true"></i>
                                    <span>{{data.data.downs}}</span>
                                </div>
                                <div class="comments">
                                    <i class="fa fa-comment" aria-hidden="true"></i>
                                    <span>{{data.data.num_comments}} Comments</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="pagination">

                    <button class="btn btn-default" v-if="before !== null" @click="previousPage">Previous Page</button>
                    <button class="btn btn-primary" v-if="after !== null" @click="nextPage">Next Page</button>

                </div>

            </div>
        </div>

    </div>
</template>


<script>
    import ListingSize from '@/components/ListingSize';

    export default {
        name: 'Listing-List',
        props: ['transparency', 'subreddit'],
        components: {'listing-size': ListingSize},
        data() {
            return {
                listings: [],
                listingSize: 25,
                after: null,
                before: null
            }
        },
        methods: {
            nextPage() {
                this.getListings('forward');
            },
            previousPage() {
                this.getListings('backward');
            },
            toggleSettings() {
                this.$emit('toggleSettings');
            },
            thumbnailURLIsValid(url) {
                const regex = /((http|https):\/\/(\S*))/g;
                return regex.test(url);
            },
            selectListing(data) {
                this.$emit('selectedListing', data)
            },
            changeSize(size) {
                this.listingSize = size;
                this.getListings();
            },
            getListings(direction) {
                this.listings = null;
                let queryString = '&';
                switch (direction) {
                    case 'forward':
                        queryString += 'after=' + this.after;
                        break;
                    case 'backward':
                        queryString += 'before=' + this.before;
                        break;
                }

                axios.get(baseURL + '/r/' + this.subreddit + '.json?limit=' + this.listingSize + '&count=' + this.listingSize + queryString).then(
                    (response) => {
                        this.listings = response.data.data.children;
                        this.before = response.data.data.before;
                        this.after = response.data.data.after;
                    }
                );
            }
        },
        watch: {
            'subreddit'(to, from) {
                this.getListings();
            }
        }
    }
</script>

<style scoped>

    .view {
        padding-bottom: 50px;
    }

    .pagination {
        position: fixed;
        bottom: 10px;
        padding-left: 10px;
    }

    .pagination .btn-primary {
        margin-left: 10px;
    }

</style>