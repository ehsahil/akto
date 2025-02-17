<template>
    <simple-layout title="Tests library">
        <template>
            <div class="px-8">
                <div class="py-4">
                    <search placeholder="Search categories" @changed="setSearchText" />
                </div>
                <div>                
                    <layout-with-left-pane>
                        <div class="category-tests">
                            <router-view :key="$route.fullPath"/>
                        </div>
                        <template #leftPane>
                            <v-navigation-drawer
                                v-model="drawer"
                                floating
                                width="250px"
                            >
                                <v-btn primary outlined color="var(--themeColor)" @click="showCreateTestDialog = true" class="ma-4">
                                    <div class="d-flex">
                                        <div class="my-auto"><v-icon size="14">$fas_plus</v-icon></div>
                                        <div class="my-auto">Add test</div>
                                    </div>
                                </v-btn>
                                <div class="nav-section">
                                    <tests-library
                                        :items=leftNavItems
                                    >
                                        <!-- <template #prependItem>
                                            <v-btn primary dark color="var(--themeColor)" tile style="width: -webkit-fill-available" class="mt-8 mb-8">
                                                <div style="width: 100%">
                                                    <v-icon>$fas_plus</v-icon> 
                                                    New test
                                                </div>
                                            </v-btn>
                                        </template> -->
                                    </tests-library>
                                </div>
                            </v-navigation-drawer>
                        </template>
                    </layout-with-left-pane>  
                </div>
            </div>
            <v-dialog v-model="showCreateTestDialog" width="600px">
                <a-card title="Add test" icon="$fas_plus"  class="ma-0">
                        <div class="py-4 px-6">
                            <div class="input-title">
                                <div>Raw URL of Nuclei template</div>
                                <v-text-field
                                    v-model="newTest.url"
                                    dense
                                    class="form-field-text"                             
                                />
                            </div>
                            <div class="input-title">
                                <div>Category</div>
                                <v-select
                                    class="form-field-select"
                                    :items="businessCategoryShortNames"
                                    v-model="newTest.category"
                                    attach
                                />
                            </div>
                            <div class="input-title mb-4">
                                <div>Subcategory</div>
                                <v-menu v-model="showSubcategoriesMenu">
                                    <template v-slot:activator="{ on, attrs }">
                                        <div v-bind="attrs" v-on="on" class="category-selection">
                                            {{newTest.subcategory}}
                                        </div>
                                    </template>

                                    <v-list class="form-field-select py-2">
                                        <v-list-item
                                            v-for="(item, index) in allSubcategories"
                                            :key="index"
                                            :class='["clickable", newTest.subcategory == item ? "c-item--selected" : "", "c-item--title"]'
                                            @click="newTest.subcategory = item" 
                                        >
                                            <v-list-item-title>
                                                {{ item }}
                                            </v-list-item-title>
                                        </v-list-item>
                                        <v-list-item
                                            :class='["clickable", "c-item--title"]'
                                            @click="(e) => {addNewSubcategory = true; e.stopPropagation(); e.preventDefault()}" 
                                        >
                                            <v-list-item-title class="primary--text">
                                                <div v-if="addNewSubcategory">
                                                    <simple-text-field
                                                        :readOutsideClick="true"
                                                        placeholder="Add new"
                                                        @changed="(name) => {newTest.subcategory = name; addNewSubcategory = false; showSubcategoriesMenu=false}"
                                                        @aborted="addNewSubcategory = false"
                                                    >

                                                    </simple-text-field>
                                                </div>

                                                <div v-else class="primary--text fs-14"><v-icon color="var(--themeColor)" size="12">$fas_plus</v-icon>Add new</div>
                                            </v-list-item-title>
                                        </v-list-item>
                                    </v-list>
                                </v-menu>
                            </div>
                            <div class="input-title">
                                <div>Severity</div>
                                <v-select
                                    class="form-field-select"
                                    :items="allSeverities"
                                    v-model="newTest.severity"
                                    attach
                                />
                            </div>
                            <div class="d-flex" style="justify-content: right">
                                <v-btn primary outlined color="var(--themeColorDark)" @click="showCreateTestDialog = false" class="mr-4">Cancel</v-btn>
                                <v-btn :disabled="validateUrl()" primary dark color="var(--themeColor)" @click="addTest">Add</v-btn>
                            </div>
                        </div>
                </a-card>
            </v-dialog>
        </template>
    </simple-layout>
</template>

<script>
import SimpleLayout from '@/apps/dashboard/layouts/SimpleLayout'
import LayoutWithLeftPane from '@/apps/dashboard/layouts/LayoutWithLeftPane'
import TestsLibrary from '@/apps/dashboard/shared/components/menus/TestsLibrary'
import ACard from '@/apps/dashboard/shared/components/ACard'
import SimpleTextField from '@/apps/dashboard/shared/components/SimpleTextField'
import Search from  '@/apps/dashboard/shared/components/inputs/Search'

import api from './api'
import func from '@/util/func'
import { mapState } from 'vuex'

export default {
    name: "PageMarketplace",
    components: { 
        SimpleLayout,
        LayoutWithLeftPane,
        TestsLibrary,
        ACard,
        SimpleTextField,
        Search,
    },
    data() {
      
        let allSeverities = ["HIGH", "MEDIUM", "LOW"]
        
        return {
            drawer: null,
            showCreateTestDialog: false,
            allSeverities,
            addNewSubcategory: false,
            showSubcategoriesMenu: false,
            businessCategories: [],
            newTest: {
                url: "",
                category: "BOLA",
                subcategory: "path_traversal",
                severity: allSeverities[0],
                description: ""
            },
            searchText: "",
            searchItems:[],
            businessSubCategories: [],
            searchUserItems:[],
        }
    },
    methods: {
        validateUrl() {
            let url = this.newTest.url
            let isGoodUrl = url != null && (url.startsWith("http://") || url.startsWith("https://")) && (url.endsWith(".yml") || url.endsWith(".yaml"))
            return !isGoodUrl
        },
        addTest() {
            this.showCreateTestDialog = false
            api.addCustomTest(this.newTest).then(resp => {
                window._AKTO.$emit('SHOW_SNACKBAR', {
                    show: true,
                    text: `Test added successfully!`,
                    color: 'green'
                })
            })
        },
        setSearchText(searchText){
            this.searchText = searchText
            if(this.searchText && this.searchText != null){
                this.onSearch()
            }
            this.$store.commit('marketplace/searchTestResults', { 'searchText': this.searchText })
        },
        async onSearch() {
            this.searchItems = []
            this.searchUserItems = []

            await api.searchTestResults(this.searchText).then((resp) =>{
                this.businessCategories = resp.categories
                this.businessSubCategories = resp.inbuiltTests
                let urlLink = "/dashboard/library/default/"
                const objLink = new Set()
                resp.inbuiltTests.forEach(item => {
                    let obj = {
                        title:item.superCategory.shortName + "/business-logic",
                        link: urlLink + item.superCategory.name,
                        icon: "$fas_plus",
                        active:false,
                    }

                    if(!objLink.has(obj.link)){
                        objLink.add(obj.link)
                        this.searchItems.push(obj)
                    }
                });

                resp.searchResults.forEach((item)=>{
                    let name = item.id.split("/")
                    let obj = {}
                    let isUser = false
                    if(item.creator == 'default'){
                        obj = {
                            title: name[name.length - 3] + "/" + item.subcategory,
                            link: urlLink + item.subcategory,
                            icon: "$fas_plus",
                            active:false,
                        }
                    }
                    else{
                        isUser = true
                        obj = {
                            title: name[name.length - 3] + "/" + item.subcategory,
                            link: "/dashboard/library/custom/" + item.subcategory,
                            icon: "$fas_plus",
                            active:false,
                        }
                    }
                    if(!objLink.has(obj.link)){
                        objLink.add(obj.link)
                        if(isUser)
                            this.searchUserItems.push(obj)
                        else{
                            this.searchItems.push(obj)
                        }
                    }
                })
            })
            if(this.searchItems && this.searchItems.length > 0){
                if(this.$route.path !== this.searchItems[0].link){
                    this.$router.push(this.searchItems[0].link)
                }
            }
        },
        createCategoryObj(arrCategoryKv, creatorTitle, creatorType, colorType) {
            return {
                icon: "$fas_plus",
                title: creatorTitle,
                group: "/dashboard/library/"+creatorType,
                color: func.actionItemColors()[colorType],
                active: true,
                items: arrCategoryKv
            }
        },
    },
    async mounted() {
        await this.$store.dispatch('marketplace/fetchAllMarketplaceSubcategories')
        await this.onSearch()
    },
    computed: {
        ...mapState('marketplace', ['defaultSubcategories', 'userSubcategories', 'loading']),
        allSubcategories() {
            return [... new Set([...this.defaultSubcategories, ...this.userSubcategories])].map(x => x.split("/")[1])
        },
        businessCategoryNames() {

            let businessLogicCategoriesSet = new Set()
            let ret = []

            for(let index in this.businessSubCategories) {
                let category = this.businessSubCategories[index]
                let shortName = category.superCategory.shortName

                if (!businessLogicCategoriesSet.has(shortName)) {
                    businessLogicCategoriesSet.add(shortName)
                    ret.push({text: shortName+"/business-logic", value: category.superCategory.name})
                }
            }

            return ret
        },
        businessCategoryShortNames() {
            return [...new Set(this.businessCategories.map(category => {return { text: category.shortName, value: category.name}}))]
        },
        businessCategoryShortNamesMap() {
            let ret = {}
            for (let index in this.businessCategoryShortNames) {
                ret[this.businessCategoryShortNames[index].value] = this.businessCategoryShortNames[index].text
            }
            return ret
        },
        leftNavItems() {
            return [
                this.createCategoryObj(this.searchItems, "Categories", "default", "This week"),
                this.createCategoryObj(this.searchUserItems, "Your tests", "custom", "Total")
            ]
        }
    }
}
</script>

<style lang="sass" scoped>
.category-tests
    color: var(--themeColorDark)
    font-family: Poppins, sans-serif !important

.input-title
    display: flex
    color: var(--themeColorDark)
    padding-bottom: 8px
    & > div:first-child
        font-size: 14px
        font-weight: 500
        max-width: 200px
        min-width: 200px
        width: 200px

.form-field-select
    padding: 0px
    margin: 0px    
        
</style>

<style scoped>
.form-field-select >>> .v-list-item__title {
    font-size: 14px !important;
    color: var(--themeColorDark) !important;
}

.category-selection {
    font-size: 14px !important;
    color: var(--themeColorDark) !important;
    margin: 0px !important;
    width: 100%;
    padding-bottom: 8px;
    border-bottom: 1px solid var(--hexColor17);
}

.form-field-select >>> .v-select__selection {
    font-size: 14px !important;
    color: var(--themeColorDark) !important;
    margin: 0px !important;
}

.form-field-select >>> .v-list-item__content {
    padding: 0px !important;
    color: var(--themeColorDark) !important;
}

.form-field-select >>> .v-list-item {
    min-height: 30px !important;
}

.form-field-text >>> input {
    font-size: 14px !important;
    color: var(--themeColorDark) !important;    
}

.c-item--selected {
    background: var(--hexColor24);
}

.c-item--title:hover {
    background: var(--hexColor31);
}
</style>