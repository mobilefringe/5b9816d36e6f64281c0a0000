<template>
    <footer v-if="dataLoaded" v-cloak>
        <section class="footer_menu main_container">
            <div class="row">
                <div class="col-xs-12 col-sm-6 col-md-4 footer_hours">
                    <p class="footer_heading">SHOPPING CENTRE HOURS</p>
                    <img src="//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1525870162000/clock.png" class="clock_icon" alt="">
                    <div class="footer_hours_container">
                        <p>
                            Monday to Friday: <br/>
                            <span v-for="hour in weekdayHours">
                                {{hour.open_time | moment("h:mm a", timezone)}} - {{hour.close_time | moment("h:mm a", timezone)}}    
                            </span>
                        </p>
                        <p>
                            Saturday:<br/>
                            <span v-for="hour in saturdayHours">
                                {{hour.open_time | moment("h:mm a", timezone)}} - {{hour.close_time | moment("h:mm a", timezone)}}    
                            </span>
                        </p>
                        <p>
                            Sunday:<br/>  
                            <span v-for="hour in sundayHours">
                                {{hour.open_time | moment("h:mm a", timezone)}} - {{hour.close_time | moment("h:mm a", timezone)}}    
                            </span>
                        </p>
                    </div>
                    <div class="clearfix"></div>
                </div>
                <div class="col-xs-12 col-sm-6 col-md-4 footer_newsletter">
                    <p class="footer_heading">NEWSLETTER SUBSCRIPTION</p>
                    <label for="emailAddress" class="accessibility">Enter Email Address</label>
                    <input id="emailAddress" v-model="newsletter_email" type="text" placeholder="Susbcribe to Newsletter" class="newsletter_control" required  @keyup.enter=="newsletterRoute"/>
                    <button @click="newsletterRoute" class="newsletter_btn animated_btn">Subscribe</button>
                </div>
                <div class="col-xs-12 col-sm-12 col-md-4 footer_insta">
                    <router-link class="pull-right insta_view_more" to="/leasing">View More <i class="fa fa-caret-right"></i></router-link>
                    <p class="footer_heading text-uppercase">Leasing</p>
                    <div class="insta-feed-container">
                        <div class="insta-feed-image-container" v-for="(item, index) in leasingFeed">
                            <a :href="item.image_url" data-lightbox="leasing images">
                                <div class="insta-feed-image" v-bind:style="{ backgroundImage: 'url(' + item.image_url + ')' }"></div>
                                <p style="display:none;">{{item.id}}</p>
                            </a>
                        </div>
                    </div> 
                </div>
            </div>
        </section>
        <section class="footer_privacy ">
            <div class="main_container row">
                <div class="col-sm-4 col-md-6">
                    <p class="footer_text">&#169; {{copyright_year}} {{property.name}}</p>
                    <p class="footer_text">Powered by <a href="https://www.mallmaverick.com/" target="_blank">Mall Maverick</a></p>
                </div>
                <div class="col-sm-8 col-md-6">
                    <p class="footer_text"><a :href="siteInfo.googleMapsURL" target="_blank">{{ getPropertyAddress }}</a></p> 
                    <p class="footer_text"><a :href="'tel:' +  property.contact_phone">{{ property.contact_phone }}</a> | <a href=" /pages/lindsaysquare-privacy-policy">Privacy Policy</a> | <router-link to="/jobs" exact>Jobs</router-link> | <a :href="siteInfo.propertyManagementURL" target="_blank">{{ siteInfo.propertyManagementName }}</a></p>
                </div>
            </div>
        </section>
    </footer>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "json!site.json", "lightbox"], function (Vue, Vuex, moment, tz, VueMoment, site, lightbox) {
        return Vue.component("footer-component", {
            template: template, // the variable template will be injected,
            data: function data() {
                return {
                    dataLoaded: false,
                    leasingFeed: null,
                    siteInfo: site,
                    newsletter_email: ""
                }
            },
            created () {
                var socialFeed = this.findRepoByName("Leasing Images");
                if(socialFeed != null && socialFeed !== undefined && socialFeed.images.length > 0){
                    socialFeed = socialFeed.images;
                    this.leasingFeed = _.slice(socialFeed, [0], [14]);
                }
                this.dataLoaded = true;
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'getPropertyHours',
                    'getPropertyHolidayHours',
                    'findRepoByName'
                ]),
                weekdayHours() {
                    return _.filter(this.getPropertyHours, function(o) { return o.day_of_week == 1 });
                },
                saturdayHours() {
                    return _.filter(this.getPropertyHours, function(o) { return o.day_of_week == 6 });
                },
                sundayHours() {
                    return _.filter(this.getPropertyHours, function(o) { return o.day_of_week == 0 });
                },
                copyright_year() {
                    return moment().year();
                },
                getPropertyAddress() {
                    return this.property.address1 + ', ' + this.property.city + ', ' + this.property.country + ' ' + this.property.province_state
                }
            },
            methods: {
                newsletterRoute() {
                    this.show_menu = false;
                    this.$router.push("/newsletter?email=" + this.newsletter_email);
                    this.newsletter_email = "";
                }
            }
        });
    });
</script>