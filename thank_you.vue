<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div v-if="pageBanner" class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <h2>Thank You</h2>
                    </div>
                </div>
                <div class="main_container mobile_padding margin_30">
                    <div class="row">
                        
                        <div class="col-sm-12">
                            <div class="page_body">
                                Your subscription has been confirmed. You've been added to our list and will hear from us soon.
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex"], function (Vue, Vuex) {
        return Vue.component("thank-you", {
            template: template, // the variable template will be injected,
            props: ['id'],
            data: function data() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    currentPage: null
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
                        var temp_repo = _this.findRepoByName('Pages Banner');
                        if(temp_repo  && temp_repo.images) {
                           temp_repo = temp_repo.images;
                           this.pageBanner = temp_repo[0];
                        }
                        else {
                            this.pageBanner = {
                                "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                            }
                        }
                        _this.dataLoaded = true;
                        
                    });
                }
            }
        });
    });
</script>