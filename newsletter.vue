<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-if="pageBanner" v-bind:style="{ background: 'linear-gradient(0deg, rgba(0,0,0,0.2), rgba(0,0,0,0.2)), #000 url(' + pageBanner.image_url + ') center center' }">
                    <div class="main_container position_relative">
                        <h2>Newsletter</h2>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                            <div v-if="pageContent" v-html="pageContent.body"></div>
                            <form class="newsletter_form form-horizontal" action="//mobilefringe.createsend.com/t/d/s/uliklt/" method="post">
                                <div class="row">
                                    <div class="col-sm-6" >
                                        <label for="fielddjdylh" class="visuallyhidden">First Name</label>
                                        <input v-model="form_data.first_name" required class="margin_20 form-control" id="fielddjdylh" name="cm-f-djdylk" type="text" placeholder="First Name">
                                    </div>
                                    <div class="col-sm-6" >
                                        <label for="fielddjdylk" class="visuallyhidden">Last Name</label>
                                        <input v-model="form_data.last_name" required class="margin_20 form-control" id="fielddjdylk" name="cm-f-gvuhu" type="text" placeholder="Last Name">
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-sm-12">
                                        <label for="newsletter_email" class="visuallyhidden">Email</label>
                                        <input v-model="form_data.email" required class="margin_20 form-control" name="cm-uliklt-uliklt" type="email" placeholder="Email" id="newsletter_email">
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-sm-12">
                                        <div style="margin-left: 20px">
                                            <label class="checkbox">
                                                <input name="agree_newsletter" required  type="checkbox">
                                                    I agree to receive communications from {{ property.name }}.
                                            </label>
                                        </div>
            					    </div>
            					</div>
        					    <div class="margin_40 clearfix"></div>
        					    <div class="row">
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
            data: function() {
                return {
                    dataLoaded: true,
                    pageContent: null,
                    siteInfo: site,
                    form_data : {},
                    formSuccess : false,
                    formError: false,
                    pageBanner: null
                }
            },
            mounted () {
                this.form_data.first_name = this.$route.query.name;
                $("#fieldgvuhk").val(this.form_data.first_name);
                this.form_data.email = this.$route.query.email;
                $("#newsletter_email").val(this.form_data.email);
            },
            watch : {
                $route () {
                    this.form_data.first_name = this.$route.query.name;
                    $("#fieldgvuhk").val(this.form_data.first_name);
                    this.form_data.email = this.$route.query.email;
                    $("#newsletter_email").val(this.form_data.email);
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Newsletter Banner');
                    if (temp_repo != null && temp_repo != undefined) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5dddc9b76e6f642ebd020000/image/png/1554995355000/picorivera_banner.png"
                        }
                    }
                    
                    if (response) {
                        this.pageContent = response[0].data;
                    }
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function () {
                    this.property.mm_host = this.property.mm_host.replace("http:", "");
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch('LOAD_PAGE_DATA', { url: this.property.mm_host + "/pages/" + site.subdomain + "-newsletter.json" })
                        ]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                validateBeforeSubmit(form) {
                    this.$validator.validateAll().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            
                            if (errors.length > 0) {
                                console.log("Error");
                                this.formError = true;
                            } else {
                                form.preventDefault();
                                console.log("No Error", form);
                                var vm = this;
                                $.getJSON(
                                form.target.action + "?callback=?",
                                $(form.target).serialize(),
                                function (data) {
                                    if (data.Status === 400) {
                                       vm.formError = true;
                                    } else { // 200
                                        vm.formSuccess = true;
                                    }
                                });
                                
                            }
                        }
                    })
                }
            }
        });
    });
</script>