<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <h2>Leasing</h2>
                    </div>
                </div>
                <div class="main_container mobile_padding margin_30">
                    <div class="details_row">
                        <div class="details_col_3 hidden_phone">
                            <img class="img_max" :src="sideBanner.image_url" alt="" />    
                        </div>
                        <div class="details_col_9">
                            <h3 class="inside_page_header">Permanent Leasing</h3>
                            <div class="margin_40 page_body" v-if="permLeasing" v-html="permLeasing.body"></div>
        					<form class="form-horizontal padding_top_20" action="form-submit" v-on:submit.prevent="validateBeforeSubmitPerm">
        						<div class="form-group ">
        							<div class="col-xs-12 margin_20" :class="{'has-error': errors.has('legalName')}">
        								<label for="legalName">Legal Name of Organization<span class="req_star"> *</span></label>
        								<input id="legalName" v-model="form_data.legalName" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="legalName" type="text" data-vv-delay="500" data-vv-as="Legal Name of Organization">
        								<span v-show="errors.has('legalName')" class="form-control-feedback">{{ errors.first('legalName') }}</span>
        							</div>
        						</div>
        						<div class="form-group">
        							<div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('firstName')}">
        							    <label for="firstName">Contact First Name<span class="req_star"> *</span></label>
        								<input id="firstName" v-model="form_data.firstName" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="firstName" type="text" data-vv-delay="500" data-vv-as="First Name">
        								<span v-show="errors.has('firstName')" class="form-control-feedback">{{ errors.first('firstName') }}</span>
        							</div>
        							<div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('lastName')}">
        							    <label for="lastName">Contact Last Name<span class="req_star"> *</span></label>
        								<input id="lastName" v-model="form_data.lastName" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="lastName" type="text" data-vv-delay="500" data-vv-as="Last Name">
        								<span v-show="errors.has('lastName')" class="form-control-feedback">{{ errors.first('legalName') }}</span>
        							</div>
        						</div>
        						<div class="form-group">
        						    <div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('phone')}">
    									<label for="phone">Contact Phone Number<span class="req_star"> *</span></label>
    									<input id="phone" v-model="form_data.phone" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="phone" type="text" data-vv-delay="500" data-vv-as="Phone Number">
    									<span v-show="errors.has('phone')" class="form-control-feedback">{{ errors.first('phone') }}</span>
    								</div>
        							<div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('email')}">
        								<label for="email">Contact Email Address<span class="req_star"> *</span></label>
        								<input id="email" v-model="form_data.email" v-validate="'required|email'" class="form-control" :class="{'input': true}" name="email" type="email" data-vv-delay="500" data-vv-as="Email Address">
        								<span v-show="errors.has('email')" class="form-control-feedback">{{ errors.first('email') }}</span>
        							</div>
        						</div>
    							<div class="form-group">
        						    <div class="col-xs-12 margin_20">
    									<label for="size">Square Footage Required</label>
    									<select id="size" v-model="form_data.size" class="form-control">
                                            <option value="">Select square footage</option>
                                            <option value="Less than 500 sq.ft.">Less than 500 sq.ft.</option>
                                            <option value="500 - 1000 sq. ft.">500 - 1000 sq. ft.</option>
                                            <option value="1000 - 2500 sq. ft.">1000 - 2500 sq. ft. </option>
                                            <option value="More than 2500 sq. ft.">More than 2500 sq. ft.</option>
                                        </select>
    								</div>
        						</div>
        						<div class="form-group">
        						    <div class="col-xs-12 margin_20">
    									<label for="comments">Comments</label>
    									<textarea id="comments" class="form-control" v-model="form_data.comments"></textarea>
    								</div>
    							</div>
        						<div class="form-group">
        							<div class="col-xs-12">
        								<button class="animated_btn" type="submit" :disabled="formSuccessPerm">
        								    Submit <i class="fa fa-angle-right" aria-hidden="true"></i>
    								    </button>
        							</div>
        						</div>
        					</form>
        					<div id="send_contact_success" class="alert alert-success text-left" role="alert" v-show="formSuccessPerm">
        						<span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
        						<span class="sr-only">Success</span>
        						Thank you! A member from our Permanent Leasing Team will contact you shortly.
        					</div>
        					<div id="send_contact_error" class="alert alert-danger text-left" role="alert" v-show="formErrorPerm">
        						<span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
        						<span class="sr-only">Error:</span>
        						There was an error when trying to submit your request. Please try again later.
        					</div>
        				</div>
    				</div>
    			</div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "jquery", "vee-validate", "utility"], function(Vue, Vuex, $, VeeValidate, Utility) {
        Vue.use(VeeValidate);
        return Vue.component("leasing-permanent-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    permLeasing: null,
                    form_data: {},
                    formSuccessPerm: false,
                    formErrorPerm: false,
				    sideBanner:null
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Leasing Banner');
                    if(temp_repo  && temp_repo.images) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                        }
                    }
                    var temp_repo1 = this.findRepoByName('Leasing Side Banner');
                    if(temp_repo1 && temp_repo1.images) {
                        this.sideBanner = temp_repo1.images[0];
                    } else {
                        this.sideBanner = {
                            "image_url": ""
                        }
                    } 
                    if(response && response[0]) {
                        this.permLeasing = response[0].data;
                    }

                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName'
                ]),
            },
            methods: {
                loadData: async function () {
                    this.property.mm_host = this.property.mm_host.replace("http:", "");
                    try {
                        let results = await Promise.all([this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host + "/pages/malvern-permanent-leasing.json"})]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                validateBeforeSubmitPerm() {
                    this.$validator.validate().then((result) => {
                        console.log(result)
                        if (result) {
                            let errors = this.errors;
                            //format email
                            send_data = {};
                            send_data.url = "https://www.mallmaverick.com/custom_email";
                            var perm_formdata = {}; //JSON.stringify(this.serializeObject(this.form_data));
                            perm_formdata.send_to = "sankavy@mobilefringe.com"//"huntleyj@davpart.com";
                            perm_formdata.from_email = this.form_data.email
                            perm_formdata.subject = this.property.name +" Long Term Leasing Form"; 
                            perm_formdata.body = {};
                            perm_formdata.body["Legal Name of Organization"] =  this.form_data.legalName;
                             
                            perm_formdata.body["Contact First Name"] =   this.form_data.firstName, 
                            perm_formdata.body["Contact Last Name"] = this.form_data.lastName,
                            perm_formdata.body["Contact Phone Number"] = this.form_data.phone, 
                            perm_formdata.body["Contact Email Address" ] =  this.form_data.email, 
                            perm_formdata.body["Square Footage Required"] =  this.form_data.size, 
                            perm_formdata.body["Comments"] =  this.form_data.comments,
                            
                            send_data.form_data = Utility.serializeObject(perm_formdata);
                            console.log("Data ", send_data.form_data)
                            var vm = this;
                            $.ajax({
                                url : send_data.url,
                                type: "POST",
                                data : perm_formdata,
                                success: function(data, textStatus, jqXHR){
                                    vm.formSuccessPerm = true;
                                },
                                error: function (jqXHR, textStatus, errorThrown){
                                  console.log("Data load error: " + error.message);
                                  vm.formErrorPerm = true;
                                }
                            });
                        }
                    })
                }
            }
        });
    });
</script>