<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h1>#MyDistrict</h1>
                    </div>
                </div>
                <div class="main_container blog_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="text-center blog_desc">
                        <h3>Your insider's guide to everything THE DISTRICT, from where to Shop, Dine and Play -- to uncovering the hidden gems and discovering what’s new and fresh!</h3>
                    </div>
                    <div class="row margin_40">
        		        <div class="col-md-12 clearfix text-center">
        		            <button class="animated_btn stores_btn" @click="filterList('all')">All</button>
        		            <button class="animated_btn stores_btn" @click="filterList(tag)" v-for="tag in tags">{{tag}}</button>
        		        </div>
        		    </div>
                    <transition-group name="list" tag="div">
                        <div v-if="paginatedBlogs" v-for="(event,index) in paginatedBlogs" :key="event.id">
                            <div v-if="index==0"  class="row event_container">
                                <div class="col-md-12 text-center">
                                <img :src="event.image_url" :alt="'Event: ' + event.title" class="event_img img_max" /> 
                                <h4 class="event_name">{{ event.title }}</h4>
                                <p class="event_dates"><span v-if="event.tags && event.tags.length >0">{{event.tags[0]}} | </span><span>{{ event.publish_date | moment("MMMM D", timezone)}}</span></p>
                                <div class="event_desc" v-html="event.description_short"></div>
                                <router-link :to="{ name: 'postsDetails', params: { id: event.slug, banner: pageBanner }}">
                                    <p class="event_link">Post Details <i class="fas fa-angle-double-right"></i></p>
                                </router-link>
                                </div>
                            </div>
                            <div class="row event_container" v-else>
                                <div class="col-md-4">
                                    <img :src="event.image_url" :alt="'Event: ' + event.title" class="event_img img_max" />   
                                </div>
                                <div class="col-md-8">
                                    <h4 class="event_name">{{ event.title }}</h4>
                                    <p class="event_dates"><span v-if="event.tags && event.tags.length >0">{{event.tags[0]}} | </span><span>{{ event.publish_date | moment("MMMM D", timezone)}}</span></p>
                                    <div class="event_desc" v-html="event.description_short"></div>
                                    <router-link :to="{ name: 'postsDetails', params: { id: event.slug, banner: pageBanner }}">
                                        <p class="event_link">Post Details <i class="fas fa-angle-double-right"></i></p>
                                    </router-link>
                                </div>
                            </div>
                        </div>
                        <div v-else>
                            <div class="row">
                                <div class="col-md-12">
                                    <p>Sorry, there are no Promotions posted at this time. Please check back soon!</p>    
                                </div>
                            </div>
                        </div>
                    </transition-group>
                    <div class="row margin_60">
                        <div class="col-md-12">
                            <button class="animated_btn event_load_more" v-if="!noMoreBlogs" @click="handleButton">Load More</button>
                            <p v-if="noBlogs">No More Blogs</p>
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
        return Vue.component("posts", {
            template: template, // the variable template will be injected,
            data: function () {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    toggleEvents: true,
                    togglePromos: false,
                    paginatedBlogs: [],
                    moreBlogs: [],
                    moreBlogsFetched: false,
                    noMoreBlogs: false,
                    noBlogs: false,
                    filteredBlogList: null
                }
            },
            created (){
                this.loadData().then(response => {
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
                    this.dataLoaded = true;
                    var temp_blogs = this.blogList();
                    this.filteredBlogList = temp_blogs;
                    this.handleButton();
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedEvents',
                    'processedPromos',
                    'findRepoByName',
                    'findBlogBySlug'
                ]),
                
                tags(){
                    var tags = this.uniqueTags([].concat(_.map(this.blogList(), 'tag')));
                    return _.uniq(_.pullAll(tags, [null]));
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos"), this.$store.dispatch("getData", "blogs"), this.$store.dispatch("getData","promotions")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                toggleView(item) {
                    if(this.promos.length == 0) {
                        this.handleButton();
                    }
                    
                    if(this.toggleEvents) { 
                        this.toggleEvents = false
                    } else {
                        this.toggleEvents = true
                    }
                    
                    if(this.togglePromos) {
                        this.togglePromos = false
                    } else {
                        this.togglePromos = true
                    }
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
                },
                handleButton () {
                    if(!this.moreBlogsFetched){
                        this.moreBlogs = this.filteredBlogList;
                        this.paginatedBlogs = this.moreBlogs.splice(0, 3);
                        this.moreBlogsFetched = true;
                    } else {
                        var nextBlogs = this.moreBlogs.splice(0, 3);
                        // Add 3 more posts to posts array
                        var vm = this;
                        _.forEach(nextBlogs, function(value, key) {
                            vm.paginatedBlogs.push(value);
                        });
                    }
                    if(this.filteredBlogList.length === 0){
                        this.noMoreBlogs = true
                        this.noBlogs = true
                    }
                },
                uniqueTags(currentTags) {
                    var allUniqueTags = [];
                    _.forEach(currentTags, function(val, key){
                        if(val !== null && val !== undefined && val.length > 0) {
                            _.forEach(val, function(tag, key){
                                if(!_.includes(allUniqueTags,tag )){
                                    allUniqueTags.push(tag);
                                }
                            });
                        }
                    });
                    return allUniqueTags;
                },
                filterList(tag){
                    
                    var temp_blogs = this.blogList();
                    this.filteredBlogList = temp_blogs
                    if(tag == 'all'){
                       this.filteredBlogList =temp_blogs;
                    }
                    else {
                        this.filteredBlogList =  _.filter(temp_blogs,function(o){return _.includes(o.tag, tag);});
                    }
                    this.moreBlogsFetched = false;
                    this.noMoreBlogs = false;
                    this.noBlogs = false;
                    this.handleButton();
                },
                blogList() {
                    var blog = this.findBlogBySlug('district-main');
                    if(blog !== null && blog !== undefined) {
                        
                        blog = blog.posts;
                        var blog = _.orderBy(blog, ['publish_date'], ['desc']);
                        var showBlogs = [];
                        var month_heading = "";
                        _.forEach(blog, function (value, key) {
                            var today = moment.tz(this.timezone).format();
                            var showOnWebDate = moment.tz(value.publish_date, this.timezone).format();
                            var today_month = moment.tz(this.timezone).format("MM-YYYY");
                            if (today >= showOnWebDate) {
                                var start_month = moment.tz(value.start_date, this.timezone).format("MM-YYYY");
                                if (start_month <= today_month) {
                                    value.month = moment.tz(this.timezone).format("MMMM YYYY");
                                    month_heading = today_month;
                                } else {
                                    value.month = moment.tz(value.start_date, this.timezone).format("MMMM YYYY");
                                    month_heading = start_month;
                                }
                                
                                if (_.includes(value.image_url, 'missing')) {
                                    value.image_url = "//codecloud.cdn.speedyrails.net/sites/5c93d5496e6f642f32010000/image/png/1553624484143/creekside_placeholder.png";
                                }
                                
                                value.description_short = _.truncate(value.body, { 'length': 250, 'separator': ' ' });
                                
                                showBlogs.push(value);
                            }
                        });
                    }
                    return showBlogs;
                }
            }
        });
    });
</script>
