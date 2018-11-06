<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <div class="page_container">
                            <h2>{{ currentStore.name }}</h2>
                        </div>
                    </div>
                </div>
                <div class="main_container margin_30" id="store_details_container">
                    <div class="details_row">
                        <div class="details_col_3">
                            <div v-if="currentStore.no_logo" class="store_details_image center-block">
                                <div class="no_logo">
                                    <p class="store_details_name">{{ currentStore.name }}</p>
                                </div>    
                            </div>
                            <img v-else class="store_details_image center-block" :src="currentStore.store_front_url_abs" :alt="currentStore.name + ' Logo'" />
                            <div v-if="currentStore.phone">
                                <h3 class="inside_page_title">Phone</h3>
                                <a class="store_details_phone" :href="'tel:' + currentStore.phone">{{ currentStore.phone }}</a>    
                            </div>
                            <a v-if="currentStore.website" class="animated_btn" :href="'http://' + currentStore.website" target="_blank">Visit Website</a>
                        </div>
                        <div class="details_col_9">
                            <div id="map" class="margin_20">
                                <mapplic-map ref="svgmap_ref" :height="300" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :maxscale= "5" :storelist="processedStores" :floorlist="floorList" :svgWidth="2500" :svgHeight="2500" @updateMap="updateSVGMap" :key="currentStore.id"></mapplic-map>
                            </div>
                            <div class="inside_page_header">Store Hours & Information</div>
                            <ul v-if="storeHours" class="store_details_hours_list">
                                <li v-for="hour in storeHours" :class="{ today: hour.todays_hours }">
                                    <div v-if="hour.is_closed">
                                        <span class="hours_list_day">{{hour.day_of_week | moment("dddd", timezone)}} </span>CLOSED
                                    </div>
                                    <div v-else-if="hour.open_full_day">
                                        <span class="hours_list_day">{{hour.day_of_week | moment("dddd", timezone)}} </span>Open 24 Hours
                                    </div>
                                    <div v-else>
                                        <span class="hours_list_day">{{hour.day_of_week | moment("dddd", timezone)}} </span>{{hour.open_time | moment("h:mma", timezone)}} - {{hour.close_time | moment("h:mma", timezone)}}
                                    </div>
                                </li>
                            </ul>
                            <div class=" margin_30 store_details_desc" v-html="currentStore.rich_description"></div>
                            <div v-if="currentStore.promotions">
                                <b-card no-body class="mb-1 inside_page_toggle">
                                    <b-card-header header-tag="header" class="p-1" role="tab">
                                        <b-btn block @click="togglePromos = !togglePromos" :aria-expanded="togglePromos ? 'true' : 'false'" aria-controls="togglePromotions">
                                            Promotions
                                            <i v-if="togglePromos"  class="fa fa-minus f"></i>
                                            <i v-else  class="fa fa-plus"></i>
                                        </b-btn>
                                    </b-card-header>
                                    <b-collapse v-for="promo in storePromotions" v-model="togglePromos" role="tabpanel" id="togglePromotions" class="accordion_body">
                                        <b-card-body>
                                            <div class="row">
                                                <div class="col-md-12">
                                                    <h3 class="promo_name">{{promo.name}}</h3>
                                                    <p class="promo_date" v-if="isMultiDay(promo)">
                        							    {{ promo.start_date | moment("MMMM D", timezone)}} to {{ promo.end_date | moment("MMMM D", timezone)}}
                                                    </p>
                                                    <p class="promo_date" v-else>{{ promo.start_date | moment("MMMM D", timezone)}}</p>
                                                    <div class="promo_desc" v-html="promo.description_short"></div>
                                                    <router-link :to="'/promotions/'+ promo.slug" >
							                            <i class="fa fa-caret-right"></i> <span class="read_more">View Promotion Details</span>
					                                </router-link>
                                                </div>
                                            </div>
                                            <hr class="promo_separator" />
                                        </b-card-body>
                                    </b-collapse>
                                </b-card>
                            </div>
                            <div v-if="currentStore.jobs">
                                <b-card no-body class="mb-1 inside_page_toggle">
                                    <b-card-header header-tag="header" class="p-1" role="tab">
                                        <b-btn block @click="toggleJobs = !toggleJobs" :aria-expanded="toggleJobs ? 'true' : 'false'" aria-controls="toggleJobs">
                                            Jobs
                                            <i v-if="toggleJobs"  class="fa fa-minus f"></i>
                                            <i v-else  class="fa fa-plus"></i>
                                        </b-btn>
                                    </b-card-header>
                                    <b-collapse v-for="job in storeJobs" v-model="toggleJobs" role="tabpanel" id="toggleJobs" class="accordion_body">
                                        <b-card-body>
                                            <div class="row">
                                                <div class="col-md-12">
                                                    <h3 class="promo_name">{{job.name}}</h3>
                                                    <p class="promo_date" v-if="isMultiDay(job)">
                        							    {{ job.start_date | moment("MMMM D", timezone)}} to {{ job.end_date | moment("MMMM D", timezone)}}
                                                    </p>
                                                    <p class="promo_date" v-else>{{ job.start_date | moment("MMMM D", timezone)}}</p>
                                                    <router-link :to="'/jobs/'+ job.slug" >
							                            <i class="fa fa-caret-right"></i> <span class="read_more">View Job Details</span>
					                                </router-link>
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
            </div>
        </transition>
    </div>
</template>

<script>
    define(['Vue', 'vuex', 'moment', "vue!mapplic-map", 'json!site.json'], function(Vue, Vuex, moment, MapplicComponent, Site) {
        return Vue.component("store-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    currentStore: null,
                    storePromotions: null,
                    togglePromos: false,
                    storeJobs: null,
                    toggleJobs: false,
                    storeHours: [],
                }
            },
            props:['id'],
            beforeRouteUpdate(to, from, next) {
                this.loadData().then(response => {
                    this.dataLoaded = true;
                    this.updateCurrentStore(to.params.id);
                });
                next();
            },
            created (){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Directory Banner');
                    if(temp_repo  && temp_repo.images) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                        }
                    }
                    
                    this.dataLoaded = true;
                    this.updateCurrentStore(this.id);
                });
            },
            watch: {
                currentStore: function() {
                    this.currentStore.zoom = 2;
                    
                    if (_.includes(this.currentStore.store_front_url_abs, 'missing')) {
                        this.currentStore.no_logo = true
                    } else {
                        this.currentStore.no_logo = false
                    }
                    
                    var vm = this;
                    if (this.currentStore.store_hours) {
                        var storeHours = [];
                        _.forEach(this.currentStore.store_hours, function (value, key) {
                            storeHours.push(vm.findHourById(value));
                        });
                        this.storeHours = storeHours;
                    }
                    
                    var vm = this;
                    var temp_promo = [];
                    _.forEach(this.currentStore.promotions, function(value, key) {
                        var current_promo = vm.findPromoById(value);
                        
                        if (_.includes(current_promo.image_url, 'missing')) {
                            current_promo.image_url = "http://placehold.it/1560x800/757575";
                        }
                        current_promo.description_short = _.truncate(current_promo.description, { 'length': 150, 'separator': ' ' });

                        temp_promo.push(current_promo);
                    }); 
                    this.storePromotions = temp_promo;
                    this.togglePromos = true;
                    
                    var vm = this;
                    var temp_job = [];
                    _.forEach(this.currentStore.jobs, function(value, key) {
                        var current_job = vm.findJobById(value);
                        temp_job.push(current_job);
                    }); 
                    this.storeJobs = temp_job;
                    this.toggleJobs = true;
                },
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedStores',
                    'findStoreBySlug',
                    'findHourById',
                    'findPromoById',
                    'findJobById',
                    'findRepoByName'
                ]),
                getSVGurl() {
                    return "https://www.mallmaverick.com" + this.property.svgmap_url;
                },
                svgMapRef() {
                    return this.$refs.svgmap_ref;
                },
                allStores() {
                    this.processedStores.map(function(store){
                        store.zoom = 4;
                    })
                    return this.processedStores;
                },
                floorList () {
                    var floor_list = [];
                    var floor_1 = {};
                    floor_1.id = "first-floor";
                    floor_1.title = "Level 1";
                    floor_1.map = this.getSVGurl;
                    floor_1.z_index = 1;
                    floor_1.show = true;
                    
                    floor_list.push(floor_1);
                    
                    
                    return floor_list;
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData","promotions"), this.$store.dispatch("getData", "jobs"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                updateCurrentStore (id) {
                    this.currentStore = this.findStoreBySlug(id);
                    if (this.currentStore === null || this.currentStore === undefined){
                        this.$router.replace({ path: '/'});
                    }
                },
                updateSVGMap(map) {
                    this.map = map;
                    this.svgMapRef.showLocation(this.currentStore.svgmap_region);
                    this.svgMapRef.addActiveClass(this.currentStore.svgmap_region);
                },
                dropPin(store) {
                    this.svgMapRef.showLocation(store.svgmap_region);
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
