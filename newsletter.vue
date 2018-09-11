<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <div class="page_container">
                            <h2>Newsletter</h2>
                        </div>
                    </div>
                </div>
                <div class="main_container margin_30">
                    <div class="details_row">
                        <div class="details_col_3 hidden_phone">
                            <h3 class="inside_page_title">Address</h3>
                            <p class="inside_page_link">
                                {{ property.address1 }} <br>
                                {{ property.city }}, {{ property.province_state }} <br>
                                {{ property.postal_code }}
                            </p>
                            <a class="animated_btn" :href="siteInfo.googleMapsURL" target="_blank">Driving Direction</a>  
                        </div>
                        <div class="details_col_9">
                            <p class="inside_page_link">Be the first to know about upcoming events and special announcements from {{ property.name }}!</p>
                            <form class="form-horizontal" action="//mobilefringe.createsend.com/t/d/s/ehjql/" method="post" @submit.prevent="validateBeforeSubmit">
                                <div class="row">
                                    <div class="col-md-6">
                                        <label for="fieldzyklkj">First Name</label>
                                        <input id="fieldzyklkj" class="margin_20 form-control" name="cm-f-vfuld" type="text" required placeholder="First Name" />
                                    </div>
                                    <div class="col-md-6">
                                        <label for="fieldzyklkt">Last Name</label>
                                        <input id="fieldzyklkt" class="margin_20 form-control" name="cm-f-vnjuu" type="text" required placeholder="Last Name" />
                                    </div>
                                    <div class="col-md-6">
                                        <label for="fieldzyklki">Postal Code</label>
                                        <input id="fieldzyklki" class="margin_20 form-control" name="cm-f-vfulr" type="text" placeholder="Postal Code"/>
                                    </div>
                                    <div class="col-md-6">
                                        <label for="cm-vuutyk-vuutyk">Email</label>
                                        <input id="cm-vuutyk-vuutyk" required class="margin_20 form-control" name="cm-ehjql-ehjql" type="email" placeholder="Email">
                                    </div>
                                    <div class="col-md-12">
                                        <div style="margin-left: 20px">
                                            <label class="checkbox">
                                                <input name="agree_newsletter" required  type="checkbox">
                                                    I agree to receive communications from {{ property.name }}.
                                            </label>
                                        </div>
            					    </div>
            					    <div class="margin_20 clearfix"></div>
                                    <div class="col-xs-12">
                                        <button class="animated_btn" type="submit" :disabled="formSuccess">Subscribe</button>
                                    </div>
                                </div>
                            </form> 
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "jquery", "vee-validate", "json!site.json"], function(Vue, Vuex, $, VeeValidate, site) {
        Vue.use(VeeValidate);
        return Vue.component("newsletter-component", {
            template: template, // the variable template will be injected
            props:['inside_banner'],
            data: function() {
                return {
                    dataLoaded: true,
                    pageBanner: null,
                    siteInfo: site,
                    form_data : {},
                    formSuccess : false,
                    formError: false
                }
            },
            created() {
                var temp_repo = this.findRepoByName('Newsletter Banner');
                if(temp_repo !== null && temp_repo !== undefined) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                        }
                    }
            },
            mounted () {
                this.form_data.email = this.$route.query.email;
                $("#cm-vuutyk-vuutyk").val(this.form_data.email);
            },
            watch : {
                $route () {
                    this.form_data.email = this.$route.query.email;
                    $("#cm-vuutyk-vuutyk").val(this.form_data.email);
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ])
            },
            methods: {
                validateBeforeSubmit(form) {
                    this.$validator.validateAll().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            
                            if(errors.length > 0) {
                                console.log("Error");
                            } else {
                                console.log("No Error");
                                // return true;
                                form.target.submit();
                            }
                        }
                    })
                }
            }
        });
    });
</script>