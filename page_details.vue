<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div v-if="pageBanner" class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <h2 v-html="currentPage.title"></h2>
                    </div>
                </div>
                <div class="main_container mobile_padding margin_30">
                    <div class="details_row">
                        <div class="details_col_3 hidden_phone">
                            <img class="img_max" :src="sideBanner.image_url" alt="" />    
                        </div>
                        <div class="details_col_9">
                            <div class="page_body" v-html="currentPage.body"></div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex", "lightbox"], function (Vue, Vuex, Lightbox) {
        Vue.use(Lightbox);
        return Vue.component("page-details-component", {
            template: template, // the variable template will be injected,
            props: ['id'],
            data: function data() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    currentPage: null,
				    sideBanner:null
                }
            },
            created() {
                this.updateCurrentPage(this.id);
            },
            watch: {
                $route: function () {
                    this.updateCurrentPage(this.$route.params.id);
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ])
            },
            methods: {
                updateCurrentPage(id) {
                    this.$nextTick(function() {
                        var _this = this;
                        this.property.mm_host = this.property.mm_host.replace("http:", "");
                        this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host + "/pages/" + _this.id + ".json" }).then(function (response) {
                            var temp_repo = _this.findRepoByName('Pages Banner');
                           
                            if(temp_repo !== null && temp_repo !== undefined) {
                               temp_repo = temp_repo.images;
                               _this.pageBanner = temp_repo[0];
                            }
                            else {
                                _this.pageBanner = {
                                    "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                                }
                            }
                            var temp_repo1 = _this.findRepoByName('Pages Side Banner');
                            if(temp_repo && temp_repo1.images) {
                                _this.sideBanner = temp_repo1.images[0];
                            } else {
                                _this.sideBanner = {
                                    "image_url": ""
                                }
                            } 
                            _this.currentPage = response.data;
                            _this.dataLoaded = true;
                        }, function (error) {
                            console.error( "Could not retrieve data from server. Please check internet connection and try again.");
                            _this.$router.replace({ name: 'home' });
                        });
                    });
                }
            }
        });
    });
</script>