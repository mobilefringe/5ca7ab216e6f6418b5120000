<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Jobs</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div>
                        <div v-if="eventList.length > 0" v-for="(events, key) in eventList">
                            <div class="row event_container">
                                <div class="col-md-8">
                                    <h4 class="event_name">{{events.name}}</h4>
                                    <p class="event_type">{{events.job_type}}</p>
                                    <p class="event_dates">{{events.start_date | moment("MMM D", timezone)}} - {{events.end_date | moment("MMM D", timezone)}}</p>
                                    <p class="event_desc">{{events.description_short}}</p>
                                    <router-link :to="{ name: 'jobsDetails', params: { id: events.slug, banner: pageBanner }}">
                                        <div class="animated_btn event_link">View Job Details <i class="fas fa-angle-double-right"></i></div>
                                    </router-link>
                                    <hr class="event_seperator">
                                </div>
                            </div>
                        </div>
                        <div class="row margin_60" v-if="eventList.length === 0">
                            <div class="col-md-12">
                                <p>Sorry, there are no Jobs posted at this time. Please check back soon!</p>    
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-lazy-load", "bootstrap-vue"], function (Vue, Vuex, moment, tz, VueMoment, VueLazyload, BootstrapVue) {
        Vue.use(BootstrapVue);
        Vue.use(VueLazyload);
        return Vue.component("events-and-promotions-component", {
            template: template, // the variable template will be injected,
            props:['locale'],
            data: function () {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    toggleEvents: true,
                    promos: [],
                }
            },
            created (){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Jobs Banner').images;
                    if(temp_repo != null) {
                        this.pageBanner = temp_repo[0];
                    } else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5ca7ab216e6f6418b5120000/image/png/1554995355000/picorivera_banner.png"
                        }
                    }

                    if (_.isEmpty(this.eventList)) {
                        this.toggleEvents = false;
                    }
                    this.promos = this.eventList
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedJobs',
                    'findRepoByName'
                ]),
                eventList: function events() {
                    var vm = this;
                    var temp_promo = [];
                    var temp_job = [];
                    _.forEach(this.processedJobs, function(value, key) {
                        today = moment().tz(vm.timezone);
                        webDate = moment(value.show_on_web_date).tz(vm.timezone);
                        if (today >= webDate) {
                            value.description_short = _.truncate(value.description, {
                                'length': 150
                            });
                            value.description_short_2 = _.truncate(value.description_2, {
                                'length': 150
                            });
                            // if (value.store != null && value.store != undefined && _.includes(value.store.store_front_url_abs, 'missing')) {
                            //     value.store.store_front_url_abs = vm.property.default_logo_url;
                            // }
                            // else if (value.store == null || value.store == undefined) {
                            //     value.store = {};
                            //     value.store.store_front_url_abs =  vm.property.default_logo_url;
                            // }
                            if (value.store  && _.includes(value.store.store_front_url_abs, 'missing')) {
                                // this.currentPromo.store.store_front_url_abs = this.property.default_logo_url;
                                value.store.no_store_logo = true;
                            }
                            else if (!value.store) {
                                value.store = {};
                                value.store.store_front_url_abs = vm.property.default_logo_url;
                            }
                            temp_promo.push(value);
                        }
                    });
                    temp_promo = _.sortBy(temp_promo, ['created_at', 'start_date']).reverse();
                    console.log(temp_promo)
                    return temp_promo;
                },
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos"), this.$store.dispatch("getData", "jobs")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                }
            }
        });
    });
</script>