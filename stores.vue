<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <div class="page_container">
                            <h2>Directory</h2>
                        </div>
                    </div>
                </div>
                <div class="main_container margin_30">
                    <div class="">
                        <div class="row store_nav">
                            <div class="col-md-3 hidden_phone">
                                <a class="store_nav_link active_store_nav hvr-underline-from-center" href="/stores">Directory</a>
                            </div>
                            <div class="col-md-3 hidden_phone">
                                <a class="store_nav_link hvr-underline-from-center" href="/map">Centre Map</a>
                            </div>
                            <div class="col-md-3 hidden_phone">
                                <span>Sort By: </span>
                                <a class="store_nav_link hvr-underline-from-center" v-on:click="changeMode('alphabetical')">Alphabetical</a>
                            </div>
                            <div class="col-md-3">
                                <v-select v-model="selectedCat" :options="dropDownCats" :searchable="false" :on-change="filteredByCategory" class="category-select" placeholder="Select a Category" id="selectByCat" transition="menu-fade"></v-select>
                            </div>
                        </div>
                        <div class="row center">
                            <div class="col-md-12">
                                <span class="legend"><span class="promo_exist"><i class="fas fa-tag"></i></span> Promotion</span>  
                                <span class="legend"><span class="new_store"><i class="fas fa-star"></i></span> New Store </span>
                                <span class="legend"><span class="coming_soon_store"><i class="far fa-clock"></i></span> Coming Soon</span>
                            </div>
                        </div>
                        <div class="row" v-if="sortByStores">
                            <div class="col-md-6">
                                <div v-if="listOne" v-for="(stores, index) in listOne">
                                    <div class="list_header">
                                        <div class="store_initial_container">
                                            {{index}}
                                        </div>
                                    </div>
                                    <div class="store-section" v-for="store in stores">
                                        <p class="store_list_name">
                                            <router-link :to="{ name: 'storeDetails', params: { id: store.slug }}">
                                                {{store.name}}
                                            </router-link>
                                            <span v-if="store.is_new_store" class="pull-right new_store"><i class="fas fa-star"></i></span>
                                            <span v-if="store.is_coming_soon_store" class="pull-right coming_soon_store"><i class="far fa-clock"></i></span>
                                            <span v-if="store.promotions != null" class="promo_exist pull-right"><i class="fas fa-tag"></i></span>
                                        </p>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6" >
                                <div v-if="listTwo" v-for="(stores, index) in listTwo">
                                    <div class="list_header">
                                        <div class="store_initial_container">
                                            {{index}}
                                        </div>
                                    </div>
                                    <div class="store-section" v-for="store in stores">
                                        <p class="store_list_name">
                                            <router-link :to="{ name: 'storeDetails', params: { id: store.slug }}">
                                                {{store.name}}
                                            </router-link>
                                            <span v-if="store.is_new_store" class="pull-right new_store"><i class="fas fa-star"></i></span>
                                            <span v-if="store.is_coming_soon_store" class="pull-right coming_soon_store"><i class="far fa-clock"></i></span>
                                            <span v-if="store.promotions != null" class="promo_exist pull-right"><i class="fas fa-tag"></i></span>
                                        </p>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="row" v-else>
                            <div class="col-md-12">
                                <div v-for="(stores, index) in filteredStores" >
                                    <div class="list_header">
                                        <div class="store_initial_container">
                                            {{index}}
                                        </div>
                                    </div>
                                    <div class="store-section" v-for="store in stores">
                                        <p class="store_list_name">
                                            <router-link :to="{ name: 'storeDetails', params: { id: store.slug }}">
                                                {{store.name}}
                                            </router-link>
                                            <span v-if="store.is_new_store" class="pull-right new_store"><i class="fas fa-star"></i></span>
                                            <span v-if="store.is_coming_soon_store" class="pull-right coming_soon_store"><i class="far fa-clock"></i></span>
                                            <span v-if="store.promotions != null" class="promo_exist pull-right"><i class="fas fa-tag"></i></span>
                                        </p>
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
    define(["Vue", "vuex", "vue-select"], function(Vue, Vuex, VueSelect) {
        Vue.component('v-select', VueSelect.VueSelect);
        return Vue.component("stores-component", {
            template: template, // the variable template will be injected
            props:['inside_banner'],
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    sortByStores: true,
                    listOne: null,
                    listTwo: null,
                    filteredStores: null,
                    selectedCat: "Select a Category"
                }
            },
            created(){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Directory Banner');
                    if(temp_repo !== null && temp_repo !== undefined) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b9816d36e6f64281c0a0000/image/png/1531495616000/inside_banner.png"
                        }
                    }
                    this.allStores;
                    this.sortByStores = true;
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName',
                    'processedStores',
                    'processedCategories',
                    'storesByAlphaIndex',
                    'storesByCategoryName',
                    'findCategoryByName',
                    'findCategoryById'
                ]), 
                allStores() {
                    var listOne = [];
                    var listTwo = [];
                    _.forEach( this.processedStores , function( value, key ) {
                        if(value) {
                            var starter = "A";
                            var breaker = "M";
                            if(value && value.name){
                            var store_initial = _.toUpper(value.name[0]);
                                if (store_initial.charCodeAt(0) <= breaker.charCodeAt(0) && store_initial.charCodeAt(0) >= starter.charCodeAt(0)){
                                    listOne.push(value);
                                } else {
                                    listTwo.push(value);    
                                }
                            }
                        }
                    });
                    this.listOne = _.groupBy(listOne, store => (isNaN(_.upperCase(store.name.charAt(0))) ? _.upperCase(store.name.charAt(0)) : "#"));
                    this.listTwo = _.groupBy(listTwo, store => (isNaN(_.upperCase(store.name.charAt(0))) ? _.upperCase(store.name.charAt(0)) : "#"));
                },
                dropDownCats() {
                    var cats = _.filter(this.processedCategories, function(o) { return o.store_ids !== null && o.store_ids.length > 0 });
                    cats = _.map(cats, 'name');
                    cats.unshift('All');
                    return cats;
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
                },
                changeMode (mode) {
                    this.sortByStores = true;
                },
                filteredByCategory (cat_id) {
                    if(cat_id == "Select a Category" || cat_id == "All" || cat_id == null || cat_id == undefined){
                        category_id = "All";
                    } else {
                        category_id = this.findCategoryByName(cat_id).id;
                    }
                    
                    if(category_id == "All"){
                        this.sortByStores = true;
                    } else {
                        this.sortByStores = false;
                        var find = this.findCategoryById;
                        var filtered = _.filter(this.processedStores, function(o) {return _.indexOf(o.categories, _.toNumber(category_id)) > -1; });
                        _.forEach(filtered, function(value, i) {
                            value.currentCategory = find(category_id).name;
                        });
                        sortedCats = _.groupBy(filtered, store => store.currentCategory);
                        this.filteredStores = sortedCats;
                    }
                }
            }
        });
    });
</script>
