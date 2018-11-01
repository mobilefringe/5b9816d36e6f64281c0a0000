<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <h2>Promotions</h2>
                    </div>
                </div>
                <div class="main_container mobile_padding margin_30">
                    <div class="details_row">
                        <div class="details_col_3 hidden_phone">
                            <img class="img_max" src="//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/jpeg/1541089049817/sidebanner3.jpg" alt="" />    
                        </div>
                        <div class="details_col_9">
                            <!-- PROMOTIONS -->
                            <b-card no-body class="mb-1 inside_page_toggle">
                                <b-card-header header-tag="header" class="p-1" role="tab">
                                    <b-btn block @click="togglePromos = !togglePromos" :aria-expanded="togglePromos ? 'true' : 'false'" aria-controls="togglePromotions">
                                        Promotions
                                        <i v-if="togglePromos"  class="fa fa-minus f"></i>
                                        <i v-else  class="fa fa-plus"></i>
                                    </b-btn>
                                </b-card-header>
                                <b-collapse v-if="promoList.length >= 1" v-for="promo in promoList" v-model="togglePromos" role="tabpanel" id="togglePromotions" class="accordion_body">
                                    <b-card-body>
                                        <div class="row">
                                            <div class="col-md-5" v-if="">
                                                <img :src="promo.image_url" :alt="'Promotion: ' + promo.name" class="max_img" />
                                            </div>
                                            <div class="col-md-7">
                                                <h3 class="promo_name">{{promo.name}}</h3>
                                                <p class="promo_store_name">
                                                    <router-link v-if="promo.promotionable_type == 'Store'" :to="'/stores/'+ promo.store.slug">
                                                        {{ promo.store.name }}
                                                    </router-link>
                                                    <span v-else>{{ property.name }}</span>
                                                    <span>| </span>
                                                    <span v-if="isMultiDay(promo)" class="promo_date">{{ promo.start_date | moment("MMMM D", timezone)}} to {{ promo.end_date | moment("MMMM D", timezone)}}</span>
                                                    <span v-else class="promo_date">{{ promo.start_date | moment("MMMM D", timezone)}}</span>
                                                </p>
                                                <div class="promo_desc" v-html="promo.description_short"></div>
                                                <router-link :to="'/promotions/'+ promo.slug" class="hvr-icon-forward">
						                            <i class="fa fa-caret-right hvr-icon"></i> <span class="read_more">View Promotion Details</span>
				                                </router-link>
                                            </div>
                                        </div>
                                        <hr class="promo_separator" />
                                    </b-card-body>
                                </b-collapse>
                                <b-collapse v-if="promoList.length == 0" v-model="togglePromos" role="tabpanel" id="togglePromotions" class="accordion_body">
                                    <b-card-body>
                                        <div class="row">
                                            <div class="col-md-12">
                                                <p>Sorry. There are no Promotions posted at this time. Please check back soon!</p>
                                            </div>
                                        </div>
                                        <hr class="promo_separator" />
                                    </b-card-body>
                                </b-collapse>
                            </b-card>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-lazy-load", "bootstrap-vue", "json!site.json"], function (Vue, Vuex, moment, tz, VueMoment, VueLazyload, BootstrapVue, site) {
        Vue.use(BootstrapVue);
        Vue.use(VueLazyload);
        return Vue.component("promotions-component", {
            template: template, // the variable template will be injected,
            props:['inside_banner'],
            data: function () {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    togglePromos: false
                }
            },
            created (){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Promotions Banner');
                    if(temp_repo !== null && temp_repo !== undefined) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                        }
                    }
                    
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName',
                    'processedPromos'
                ]),
                promoList: function promos() {
                    var vm = this;
                    var showPromos = [];
                    _.forEach(this.processedPromos, function(value, key) {
                        var today = moment.tz(this.timezone).format();
                        var showOnWebDate = moment.tz(value.show_on_web_date, this.timezone).format();
                        if (today >= showOnWebDate) {
                            if (value.store != null && value.store != undefined) {
                                if (_.includes(value.store.store_front_url_abs, 'missing')) {
                                    value.image_url = "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/jpeg/1537545921000/PH1.jpg";
                                } else {
                                    value.image_url = value.store.store_front_url_abs;    
                                }
                            } else {
                                value.image_url = "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/jpeg/1537545921000/PH1.jpg"
                            }
                            
                            value.description_short = _.truncate(value.description, { 'length': 100, 'separator': ' ' });
                            
                            showPromos.push(value);
                        }
                    });
                    var sortedPromos = _.orderBy(showPromos, [function(o) { return o.end_date; }]);
                    if (sortedPromos.length > 0) {
                        this.togglePromos = true;
                    }
                    return sortedPromos;
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData","promotions")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                isMultiDay(promo) {
                    var timezone = this.timezone
                    var start_date = moment(promo.start_date).tz(timezone).format("MM-DD-YYYY")
                    var end_date = moment(promo.end_date).tz(timezone).format("MM-DD-YYYY")
                    if (start_date === end_date) {
                        return false
                    } else {
                        return true
                    }
                }
            }
        });
    });
</script>
