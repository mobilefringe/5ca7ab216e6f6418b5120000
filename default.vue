<template>
    <router-view>
        <h1>{{ property.name }}</h1>
    </router-view>
</template>

<script>
    define(["Vue", "vuex", "vue-meta"], function(Vue, Vuex, Meta) {
        return Vue.component("default-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    meta: {
                        meta_title: "",
                        meta_description: "",
                        meta_keywords: ""
                    }
                }
            },
            beforeRouteEnter (to, from, next) {
                next(vm => {
                    // access to component instance via `vm`
                    vm.meta = vm.findMetaDataByPath(to.path);
                });
            },
            beforeRouteUpdate (to, from, next) {
                this.meta = this.findMetaDataByPath(to.path);
                next();
            },
            computed: {
                ...Vuex.mapGetters([
                    'property'
                ]),
                findMetaDataByPath () {
                    return this.$store.getters.findMetaDataByPath;
                }
            },
            metaInfo () {
                return {
                    title: this.meta.meta_title,
                    meta: [
                        {name: 'description', content: this.meta.meta_description},
                        {name: 'keywords', content: this.meta.meta_keywords}
                    ]
                }
            }
        });
    });
</script>