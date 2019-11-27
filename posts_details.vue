<template>
    <div> <!-- Without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h1 v-if="currentPromo">{{currentPromo.title}}</h1>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row hidden-xs">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div v-if="currentPromo">
                        <div class="row">
                            <div class="col-md-12">
                                <div class="text-center">
                                    <img v-lazy="currentPromo.image_url" :alt="'Promotion: ' + currentPromo.name" class="margin_20 img_max"/>   
                                </div> 
                                <!--<p v-if="currentPromo.promotionable_type == 'Property'" class="event_store_name">{{ property.name }}</p>-->
                                <!--<p v-else class="event_store_name">{{ currentPromo.store.name }}</p>-->
                                <h4 class="event_name">{{ currentPromo.title }}</h4>
                                <p class="event_dates">
                                    <span>{{ currentPromo.publish_date | moment("MMMM D", timezone)}}</span>
                                </p>
                                <div class="event_desc event_details" v-html="currentPromo.html_body"></div>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-md-12">
                                <div class="row margin_30">
                                    <div class="col-md-12">
                                        <router-link to="/posts">
                    		                <div class="animated_btn pull-left">Back to Blog</div>    
                    		            </router-link>    
                                    </div>
                                </div>
                                <social-sharing v-if="currentPromo" :url="shareURL(currentPromo.slug)" :title="currentPromo.name" :description="currentPromo.description" :quote="truncate(currentPromo.description)" :twitter-user="siteInfo.twitterHandle" :media="currentPromo.image_url" inline-template>
                                    <div class="social_share margin_60">
                                        <network network="facebook">
                                            <i class="fab fa-facebook"></i>
                                        </network>
                                        <network network="twitter">
                                            <i class="fab fa-twitter"></i>
                                        </network>
                                        <network network="email">
                                            <i class="fas fa-envelope"></i>
                                        </network>
                                    </div>
                                </social-sharing>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "lightbox", "vue-lazy-load",  "vue-social-sharing", "json!site.json"], function(Vue, Vuex, moment, tz, VueMoment, Lightbox, VueLazyload, SocialSharing, site) {
        Vue.use(VueLazyload);
        Vue.component('social-sharing', SocialSharing);
        return Vue.component("posts-details-component", {
            template: template, // the variable template will be injected,
            props: ['id'],
            data: function() {
                return {
                    dataLoaded: false,
                    currentPromo: null,
                    siteInfo: site
                }
            },
            created() {
				this.$store.dispatch("getData", "blogs").then(response => {
				    var temp_repo = this.findRepoByName('Blog Banner');
				    if(temp_repo !== null && temp_repo !== undefined) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5c93d5496e6f642f32010000/image/png/1553624485505/creekside_banner.png"
                        }
                    }
					this.currentPromo = this.findBlogPostBySlug2('district-main',this.id);
					if (this.currentPromo === null || this.currentPromo === undefined) {
						this.$router.replace({ path: '/posts' });
					}
			    	this.$breadcrumbs[1].meta.breadcrumb = this.currentPromo.title
					this.dataLoaded = true;
				}, error => {
					console.error("Could not retrieve data from server. Please check internet connection and try again.");
				});
			},
			watch: {
                currentPromo : function (){
                    if(this.currentPromo != null) {
                        console.log("currentPromo", this.currentPromo);
                        if (this.currentPromo.promotionable_type === "Store"){
                            if  (_.includes(this.currentPromo.promo_image_url_abs, 'missing')) {
                                this.currentPromo.image_url = this.currentPromo.store.store_front_url_abs; 
                            }
                        } else {
                            if  (_.includes(this.currentPromo.promo_image_url_abs, 'missing')) {
                                this.currentPromo.image_url = "//codecloud.cdn.speedyrails.net/sites/5c93d5496e6f642f32010000/image/png/1553624484143/creekside_placeholder.png";    
                            }
                        }
                    }
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findBlogPostBySlug2',
                    'findRepoByName'
                ])
            },
            methods: {
				isMultiDay(currentPromo) {
					var timezone = this.timezone
					var start_date = moment(currentPromo.start_date).tz(timezone).format("MM-DD-YYYY")
					var end_date = moment(currentPromo.end_date).tz(timezone).format("MM-DD-YYYY")
					if (start_date === end_date) {
						return false
					} else {
						return true
					}
				},
				truncate(val_body) {
                    var truncate = _.truncate(val_body, { 'length': 99, 'separator': ' ' });
                    return truncate;
                },
				shareURL(slug) {
                    var share_url = window.location.href
                    return share_url
                }
			}
        });
    });
</script>
