<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <h2>Hours</h2>
                    </div>
                </div>
                <div class="main_container mobile_padding margin_30">
                    <div class="row">
                        <div class="col-md-6">
                            <div class="row">
                                <div class="col-md-12">
                                    <h3 class="hours_heading caps">Hours</h3>    
                                </div>
                            </div>
                            <div class="hours_container">
                                <div class="row hours_div" v-for="hour in hours">
                                    <div class="col-xs-6 col-sm-4 col-md-6">
                                        {{hour.day_of_week | moment("dddd", timezone)}}
                                    </div>
                                    <div class="col-xs-6 col-sm-4 col-md-6">
                                        <span class="">
                                            {{hour.open_time | moment("h:mm A", timezone)}} - {{hour.close_time | moment("h:mm A", timezone)}}    
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="row">
                                <div class="col-md-12">
                                    <h3 class="hours_heading caps">We will be open on the following Holidays</h3>
                                </div>
                            </div>
                            <div class="hours_container">
                                <div class="row hours_div"  v-for="hour in reducedHolidays">
                                    <div class="col-xs-12 col-sm-4 col-md-6">
                                        {{ hour.holiday_name }}, {{ hour.holiday_date | moment("MMM D", timezone) }}
                                    </div>
                                    <div class="col-xs-12 col-sm-4 col-md-6">
                                        {{ hour.open_time | moment("h:mm A", timezone) }} - {{ hour.close_time | moment("h:mm A", timezone) }}
                                    </div>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-md-12">
                                    <h3 class="hours_heading caps">We will be closed on the following Statutory Holidays</h3>
                                </div>
                            </div>
                            <div class="hours_container">
                                <div class="row hours_div" v-for="hour in closeHolidays">
                                    <div class="col-sm-12 col-sm-4 col-md-6">
                                        {{ hour.holiday_name }}, {{ hour.holiday_date | moment("MMM D", timezone) }}   
                                    </div>
                                    <div class="col-xs-12 col-sm-4 col-md-6">
                                        Closed
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta"], function (Vue, Vuex, moment, tz, VueMoment, Meta) {
        Vue.use(Meta);
        return Vue.component("hours-component", {
            template: template, // the variable template will be injected
            props:['inside_banner'],
            data: function data() {
                return {
                    dataLoaded: false,
                    pageBanner: null
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Hours Banner');
                    if(temp_repo && temp_repo.images) {
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
                    'getPropertyHours',
                    'getPropertyHolidayHours'
                ]),
                hours () {
                    return this.getPropertyHours;
                },
                holidayHours () {
                    return this.getPropertyHolidayHours;
                },
                reducedHolidays () {
                    var holidayHours = this.holidayHours;
                    return _.sortBy(_.filter(holidayHours, function(o) { return !o.is_closed; }), [function(o) { return o.holiday_date; }]);
                },
                closeHolidays () {
                    var holidayHours = this.holidayHours;
                    return _.sortBy(_.filter(holidayHours, function(o) { return o.is_closed; }), [function(o) { return o.holiday_date; }]);
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "categories")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                }
            }
        });
    });
</script>