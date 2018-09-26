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
                             <img class="img_max" src="//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/jpeg/1537546048000/SideBanner6.jpg" alt="" />    
                        </div>
                        <div class="details_col_9">
        				    <h3 class="inside_page_header">Short Term Leasing</h3>
                            <div class="margin_40 page_body" v-if="tempLeasing" v-html="tempLeasing.body"></div>
                            <form class="form-horizontal padding_top_20" action="form-submit" v-on:submit.prevent="validateBeforeSubmitTemp">
        						<div class="form-group ">
        							<div class="col-xs-12 margin_20" :class="{'has-error': errors.has('legalNameTemp')}">
        								<label for="legalNameTemp">Legal Name of Organization<span class="req_star"> *</span></label>
        								<input id="legalNameTemp" v-model="form_data.legalNameTemp" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="legalNameTemp" type="text" data-vv-delay="500" data-vv-as="Legal Name of Organization">
        								<span v-show="errors.has('legalNameTemp')" class="form-control-feedback">{{ errors.first('legalNameTemp') }}</span>
        							</div>
        						</div>
        						<div class="form-group">
        							<div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('firstNameTemp')}">
        							    <label for="firstNameTemp">Contact First Name<span class="req_star"> *</span></label>
        								<input id="firstNameTemp" v-model="form_data.firstNameTemp" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="firstNameTemp" type="text" data-vv-delay="500" data-vv-as="First Name">
        								<span v-show="errors.has('firstNameTemp')" class="form-control-feedback">{{ errors.first('firstNameTemp') }}</span>
        							</div>
        							<div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('lastNameTemp')}">
        							    <label for="lastNameTemp">Contact Last Name<span class="req_star"> *</span></label>
        								<input id="lastNameTemp" v-model="form_data.lastNameTemp" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="lastNameTemp" type="text" data-vv-delay="500" data-vv-as="Last Name">
        								<span v-show="errors.has('lastNameTemp')" class="form-control-feedback">{{ errors.first('lastNameTemp') }}</span>
        							</div>
        						</div>
        						<div class="form-group">
        						    <div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('phoneTemp')}">
    									<label for="phoneTemp">Contact Phone Number<span class="req_star"> *</span></label>
    									<input id="phoneTemp" v-model="form_data.phoneTemp" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="phoneTemp" type="text" data-vv-delay="500" data-vv-as="Phone Number">
    									<span v-show="errors.has('phoneTemp')" class="form-control-feedback">{{ errors.first('phoneTemp') }}</span>
    								</div>
        							<div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('emailTemp')}">
        								<label for="emailTemp">Contact Email Address<span class="req_star"> *</span></label>
        								<input id="emailTemp" v-model="form_data.emailTemp" v-validate="'required|email'" class="form-control" :class="{'input': true}" name="emailTemp" type="email" data-vv-delay="500" data-vv-as="Email Address">
        								<span v-show="errors.has('emailTemp')" class="form-control-feedback">{{ errors.first('emailTemp') }}</span>
        							</div>
        						</div>
        						<div class="form-group">
        						    <div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('startDate')}">
    									<label for="startDate">Requested Start Date <span class="req_star"> *</span></label>
    									<input id="startDate" v-model="form_data.startDate" v-validate="'required|date_format:MM/DD/YYYY'" class="form-control" :class="{'input': true}" name="startDate" type="text" data-vv-delay="500" data-vv-as="Requested Dates" placeholder="MM/DD/YYYY">
    									<span v-show="errors.has('startDate')" class="form-control-feedback">{{ errors.first('startDate') }}</span>
    								</div>
    								<div class="col-sm-6 col-xs-12 margin_20" :class="{'has-error': errors.has('endDate')}">
    									<label for="endDate">Requested End Date <span class="req_star"> *</span></label>
    									<input id="endDate" v-model="form_data.endDate" v-validate="'required|date_format:MM/DD/YYYY'" class="form-control" :class="{'input': true}" name="endDate" type="text" data-vv-delay="500" data-vv-as="Requested Dates" placeholder="MM/DD/YYYY">
    									<span v-show="errors.has('endDate')" class="form-control-feedback">{{ errors.first('endDate') }}</span>
    								</div>
    							</div>
    						    <div class="form-group">
        							<div class="col-xs-12 margin_20" :class="{'has-error': errors.has('location')}">
        								<label for="location">Requested Location<span class="req_star"> *</span> (subject to availability)</label>
        								<input id="location" v-model="form_data.location" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="location" type="text" data-vv-delay="500" data-vv-as="Requested Location">
        								<span v-show="errors.has('location')" class="form-control-feedback">{{ errors.first('location') }}</span>
        							</div>
        						</div>
        						<div class="form-group">
        						    <div class="col-xs-12">
    									<label for="use_of_space">Proposed Use of Space</label>
    									<textarea id="use_of_space" class="form-control" v-model="form_data.use_of_space"></textarea>
    								</div>
    							</div>
    							<div class="margin_40"></div>
        						<div class="form-group">
        							<div class="col-xs-12">
        								<button class="animated_btn" type="submit" :disabled="formSuccessTemp">
        								    Submit <i class="fa fa-angle-right" aria-hidden="true"></i>
    								    </button>
        							</div>
        						</div>
        					</form>
        					<div id="send_contact_success" class="alert alert-success text-left" role="alert" v-show="formSuccessTemp">
        						<span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
        						<span class="sr-only">Success</span>
        						Thank you! A member from our Specialty Leasing Team will contact you shortly.
        					</div>
        					<div id="send_contact_error" class="alert alert-danger text-left" role="alert" v-show="formErrorTemp">
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
        return Vue.component("leasing-temporary-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    permLeasing: null,
                    tempLeasing: null,
                    form_data: {},
                    formSuccessTemp: false,
                    formErrorTemp: false
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Leasing Banner');
                    if(temp_repo !== null && temp_repo !== undefined) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                        }
                    }
                    
                    if(response && response[0]) {
                        this.tempLeasing = response[0].data;
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
                        let results = await Promise.all([this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host + "/pages/malvern-temporary-leasing.json"})]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                validateBeforeSubmitTemp() {
                    this.validNumError = false;
                    this.$validator.validate().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            //format email
                            send_data = {};
                            send_data.url = "https://www.mallmaverick.com/send_contact_email";
                            var temp_formdata = {}; //JSON.stringify(this.serializeObject(this.form_data));
                            temp_formdata.send_to = "malvern@davpart.com";
                            temp_formdata.subject = this.property.name + " Short Term Leasing Form"; 
                            temp_formdata.body = {};
                            temp_formdata.body["Legal Name of Organization"] =  this.form_data.legalNameTemp;
                            temp_formdata.body["Contact First Name"] =   this.form_data.firstNameTemp, 
                            temp_formdata.body["Contact Last Name"] = this.form_data.lastNameTemp,
                            temp_formdata.body["Contact Phone Number"] = this.form_data.phoneTemp, 
                            temp_formdata.body["Contact Email Address"] =  this.form_data.emailTemp, 
                            temp_formdata.body["Requested Start Date"] =  this.form_data.startDate, 
                            temp_formdata.body["Requested End Date"] =  this.form_data.endDate, 
                            temp_formdata.body["Requested Location"] =  this.form_data.location, 
                            temp_formdata.body["Proposed Use of Space"] =  this.form_data.use_of_space, 
                            
                            send_data.form_data = Utility.serializeObject(temp_formdata);
                            var vm = this;
                            $.ajax({
                                url : send_data.url,
                                type: "POST",
                                data : temp_formdata,
                                success: function(data, textStatus, jqXHR){
                                    vm.formSuccessTemp = true;
                                },
                                error: function (jqXHR, textStatus, errorThrown){
                                  console.log("Data load error: " + error.message);
                                  vm.formErrorTemp = true;
                                }
                            });
                        }
                    })
                }
            }
        });
    });
</script>