<template>
    <div> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Location</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="row" v-if="main">
                        <div class="col-md-12">
                            <div  v-html="main.body"></div>
                        </div>
                    </div>
                </div>
                <div class="main_container">
                    <div class="location_map">
                        <iframe :src="propertyAddress()" width="100%" height="490" frameborder="0" style="border:0" allowfullscreen></iframe>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row location_content">
                        <div class="col-md-6" v-if="address">
                            <div v-html="address.body"></div>
                        </div>
                        <div class="col-md-6" v-if="office">
                            <div v-html="office.body"></div>
                        </div>
                    </div>
                </div>
                <div class="location_image_container">
                    <div class="location_image" v-if="pageImages" v-for="item in pageImages">
                        <img :src="item.image_url" alt="item.id" class="img_max" />   
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
	define(["Vue", "vuex", "vue!google_map", "json!site.json"], function(Vue, Vuex, google_map, Site ) {
		return Vue.component("location-component", {
            template: template, // the variable template will be injected
            data: function () {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    main: null,
                    address: null,
                    directions: null,
                    pageImages: null
                }
            },
            created() {
                this.loadData().then(response => {
                    var repo = this.findRepoByName('Location Banner');
                    if(repo !== null && repo !== undefined) {
                       repo = repo.images;
                       this.pageBanner = repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5ca7ab216e6f6418b5120000/image/png/1554995355000/picorivera_banner.png"
                        }
                    }
                    
                    var temp_repo = this.findRepoByName('Location Images');
                    if(temp_repo) {
                        this.pageImages = temp_repo.images;
                    }
                    if(response && response[1]){
                        this.main = response[1].data;
                        if(response[1].data && response[1].data.subpages){
                           this.address = response[1].data.subpages[0];
                           this.office = response[1].data.subpages[1]
                        }
                    }
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'repos',
                    'findRepoByName'
                ]),
                getPropertyAddress() {
                    return this.property.name + ' ' + this.property.address1 + ' ' + this.property.city + ' ' + this.property.country + ' ' +this.property.province_state + ' ' + this.property.province_state
                }
            },
            methods: {
                loadData: async function () {
                    this.property.mm_host = this.property.mm_host.replace("http:", "");
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos"), this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/"+Site.subdomain+"-location.json"})]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                propertyAddress() {
                    var address = this.property.name + "+" + this.property.address1 + "+" + this.property.city + "+" + this.property.province_state + "+" + this.property.country + this.property.postal_code
                    var key ="AIzaSyCukCjH3fsuDYBdI44hZKL43m60jEToJjY"
                    var src = "https://www.google.com/maps/embed/v1/place?q=" + address + "&key="+ key
                    return src
                }
            }
        });
	});
</script>