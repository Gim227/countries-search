<template>
  <div class="container">
    <div id="toolDiv">
        <div class="mr-2" style="float: left; display: inline-block; margin-top: 1rem; font-size: .875rem; font-weight: bold; ">
            {{ '共計 : ' + CountriesList.length +' 筆' }}
        </div>
        <div style="float: left; display: inline-block; margin-top: 1rem; font-size: .875rem;">
            <span class="mr-2 tipText" style="white-space: nowrap; display: inline-flex; align-items: center;">
                點擊國家可查看該國家的詳細資訊。
            </span>
        </div>
        <div style="float: right; display: inline-flex; vertical-align: middle;">
            <div style="display: inline-flex;">
                <span class="mr-2" style="white-space: nowrap; display: inline-flex; align-items: center;">搜尋</span>
                <input type="search" class="form-control rounded searchInput" placeholder="國家名稱" aria-label="Search" aria-describedby="search-addon" v-model="searchInput" />
            </div>
        </div>
    </div>
    <table class="table" style=" font-size: .875rem;">
        <thead>
            <tr>
                <th style="text-align: center;">國旗</th>
                <th  v-bind:class="sortedClass('official')" @click="sortBy('official')">國家名稱</th>
                <th style="text-align: center;">2位國家代碼</th>
                <th style="text-align: center;">3位國家代碼</th>
                <th style="text-align: center;">母語名稱</th>
                <th style="text-align: center; width:30%">替代國家名稱</th>
                <th style="text-align: center;">國際電話區號</th>
            </tr>
        </thead>
        <tbody v-cloak>
            <tr v-if="filterCountriesList.length === 0">
                <td colspan="7" style="font-weight: bold; text-align: center; ">查無資料</td>
            </tr>
            <tr v-else v-for="cy in filterCountriesList" :key="cy.cca3 + '-' + cy.ccn3">
                <td><img :src="cy.flags.png"></td>
                <td class="icon-link" @click="showCyDetail(cy)">{{ cy.name.official }}</td>
                <td style="text-align: center;">{{ cy.cca2 }}</td>
                <td style="text-align: center;">{{ cy.cca3 }}</td>
                <template v-if="Object.keys(cy.name).includes('nativeName') && Object.keys(cy.name.nativeName).length > 0">
                    <td v-if="Object.keys(cy.name.nativeName).includes('zho')" style="text-align: center;">{{ cy.name.nativeName.zho.official }}</td>
                    <td v-else style="text-align: center;">
                        {{ cy.name.nativeName[Object.keys(cy.name.nativeName)[0]].official }}
                    </td>
                </template>
                <td v-else style="text-align: center;"></td>
                <td style="text-align: center; width:30%">{{ cy.altSpellings.join(', ') }}</td>
                <td style="text-align: center;">{{ cy.idd.root }}</td>
            </tr>
        </tbody>
    </table>
    <!-- pagination -->
    <el-footer>
        <el-pagination layout="prev, pager, next" :total="totalNum" :currentPage="currentPage"
                       :page-size="pageSize" background @current-change="setPage">
        </el-pagination>
    </el-footer>
    <!-- Modal: Country Detail -->
    <div class="modal fade bd-example-modal-xl" id="CyDetailModal" tabindex="-1" role="dialog" aria-labelledby="CyDetailModalTitle" aria-hidden="true" style="z-index: 1048;">
        <div class="modal-dialog modal-dialog-centered modal-xl" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="CyDetailModalModalTitle">{{ '詳細資訊-'+ CyDetail.name.official }}</h5>
                </div>
                <div class="modal-body sidebar-content">
                    <table class="tableDetail" style="font-size: .875rem;">
                        <thead>
                            <tr>
                                <th style="text-align: center;">國旗</th>
                                <th>國家名稱</th>
                                <th style="text-align: center;">2位國家代碼</th>
                                <th style="text-align: center;">3位國家代碼</th>
                                <th style="text-align: center;">母語名稱</th>
                                <th style="text-align: center; width:30%">替代國家名稱</th>
                                <th style="text-align: center;">國際電話區號</th>
                            </tr>
                        </thead>
                        <tbody v-cloak>
                            <tr v-if="Object.keys(CyDetail).length === 0">
                                <td colspan="7" style="font-weight: bold; text-align: center; ">查無資料</td>
                            </tr>
                            <tr v-else>
                                <td><img :src="CyDetail.flags.png"></td>
                                <td>{{ CyDetail.name.official }}</td>
                                <td style="text-align: center;">{{ CyDetail.cca2 }}</td>
                                <td style="text-align: center;">{{ CyDetail.cca3 }}</td>
                                <template v-if="Object.keys(CyDetail.name).includes('nativeName') && Object.keys(CyDetail.name.nativeName).length > 0">
                                    <td v-if="Object.keys(CyDetail.name.nativeName).includes('zho')" style="text-align: center;">{{ CyDetail.name.nativeName.zho.official }}</td>
                                    <td v-else style="text-align: center;">
                                        {{ CyDetail.name.nativeName[Object.keys(CyDetail.name.nativeName)[0]].official }}
                                    </td>
                                </template>
                                <td v-else style="text-align: center;"></td>
                                <td style="text-align: center; width:30%">{{ CyDetail.altSpellings.join(', ') }}</td>
                                <td style="text-align: center;">{{ CyDetail.idd.root }}</td>
                            </tr>
                        </tbody>
                    </table>
                    <table class="tableOther mt-3" style="font-size: .875rem;">
                        <thead>
                            <tr>
                                <th style="text-align: center;">Capital</th>
                                <th style="text-align: center;">Car(side)</th>
                                <th style="text-align: center;">CCN3</th>
                                <th style="text-align: center;">Continents</th>
                                <th style="text-align: center;">Currencies</th>
                                <th style="text-align: center;">Independent</th>
                                <th style="text-align: center;">Landlocked</th>
                                <th style="text-align: center;">Languages</th>
                            </tr>
                        </thead>
                        <tbody v-cloak>
                            <tr v-if="Object.keys(CyDetail).length === 0">
                                <td colspan="8" style="font-weight: bold; text-align: center; ">查無資料</td>
                            </tr>
                            <tr v-else>
                                <td style="text-align: center;">{{ CyDetail.capital[0] }}</td>
                                <td style="text-align: center;">{{ CyDetail.car.side }}</td>
                                <td style="text-align: center;">{{ CyDetail.ccn3 }}</td>
                                <td style="text-align: center;">{{ CyDetail.continents[0] }}</td>
                                <td style="text-align: center;">{{ filterCurrencies }}</td>
                                <td style="text-align: center;">{{ filterIndependent }}</td>
                                <td style="text-align: center;">{{ filterLandlocked }}</td>
                                <td style="text-align: center;">{{ filterLanguages }}</td>                                
                            </tr>
                        </tbody>
                    </table>
                    <table class="tableOther mt-3 mb-3" style="font-size: .875rem;">
                        <thead>
                            <tr>
                                <th style="text-align: center;">Population</th>
                                <th style="text-align: center;">Region</th>
                                <th style="text-align: center;">Start Of Week</th>
                                <th style="text-align: center;">Status</th>
                                <th style="text-align: center;">SubRegion</th>
                                <th style="text-align: center;">Timezones</th>
                                <th style="text-align: center;">Translations</th>
                                <th style="text-align: center;">UnMember</th>
                            </tr>
                        </thead>
                        <tbody v-cloak>
                            <tr v-if="Object.keys(CyDetail).length === 0">
                                <td colspan="8" style="font-weight: bold; text-align: center; ">查無資料</td>
                            </tr>
                            <tr v-else>
                                <td style="text-align: center;">{{ CyDetail.population }}</td>
                                <td style="text-align: center;">{{ CyDetail.region }}</td>
                                <td style="text-align: center;">{{ CyDetail.startOfWeek }}</td>
                                <td style="text-align: center;">{{ CyDetail.status }}</td>
                                <td style="text-align: center;">{{ CyDetail.subregion }}</td>
                                <td style="text-align: center;">{{ CyDetail.timezones.join(', ') }}</td>
                                <template v-if="Object.keys(CyDetail).includes('translations') && Object.keys(CyDetail.translations).length > 0">
                                    <td v-if="Object.keys(CyDetail.translations).includes('zho')" style="text-align: center;">{{ CyDetail.translations.zho.official }}</td>
                                    <td v-else style="text-align: center;">
                                        {{ CyDetail.translations[Object.keys(CyDetail.translations)[0]].official }}
                                    </td>
                                </template>
                                <td v-else style="text-align: center;"></td>
                                <td style="text-align: center;">{{ filterUnMember }}</td>                                
                            </tr>
                        </tbody>
                    </table>
                    <iframe 
                        width="100%" 
                        height="400" 
                        frameborder="0" 
                        style="border:0" 
                        :src="filterGoogleMaps" 
                        allowfullscreen>
                    </iframe>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-info btn-sm" data-dismiss="modal">取消</button>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */  // 禁用ESLint
export default {
  name: "Home",
  data() {
    return {
      CountriesList: [],
      searchInput: '',
      currentPage: 1,
      pageSize: 25,
      totalNum: 0,
      sort: {
          key: 'official',
          isAsc: true
      },
      CyDetail: {
          'name' : {
              'official' : '',
              'nativeName' : {},
          },
          'flags' : {
              'png' : ''
          },
          'cca2' :'',
          'cca3' :'',
          'ccn3' :'', 
          'altSpellings' : [],
           'idd' : {
               'root' : ''
           },
           'capital' : [],
           'car' : {
               'side' : ''
           },
           'continents' : [],
           'currencies' : {},
           'independent' : '',
           'landlocked' : '',
           'languages' : {},
           'population' : '',
           'region' : '',
           'startOfWeek' : '',
           'status' : '',
           'subregion' : '',
           'timezones' : [],
           'translations' : {},
           'unMember' : '',
           'maps' : {
               'googleMaps' : '',
           }
      },
    };
  },
  components: {
  },
  computed: {
    filterCountriesList: {
        get() {
            let result = this.CountriesList.filter((res) => {
                return res.name.official.replace(/\s/g, "").toLowerCase().includes(this.searchInput.replace(/\s/g, "").toLowerCase());
            });
            
            result = result.sort((a, b) => {
                a = a.name[this.sort.key] === null ? "" : a.name[this.sort.key].replace(/\s/g, "").toLowerCase()
                b = b.name[this.sort.key] === null ? "" : b.name[this.sort.key].replace(/\s/g, "").toLowerCase()

                return (a === b ? 0 : a > b ? 1 : -1) * (this.sort.isAsc ? 1 : -1)
            });

            this.totalNum = result.length;

            let begin = this.pageSize * this.currentPage - this.pageSize;
            let end = this.pageSize * this.currentPage;
            result = result.slice(begin, end);

            return result
        }
    },
    filterCurrencies: {
        get() {
            return Object.values(this.CyDetail.currencies).reduce((acc, cv) => {
                        acc.push(cv.name+'('+cv.symbol+')')
                        return acc
                    }, []).join(', ')
        }
    },
    filterIndependent: {
        get() {
            return this.CyDetail.independent ? 'Y' : 'N'             
        }
    },
    filterLandlocked: {
        get() {
            return this.CyDetail.landlocked ? 'Y' : 'N'
        }        
    },
    filterLanguages: {
        get() {
            return Object.values(this.CyDetail.languages).reduce((acc, cv) => {
                        acc.push(cv)
                        return acc
                    }, []).join(', ')
        }
    },
    filterUnMember: {
        get() {
            return this.CyDetail.unMember ? 'Y' : 'N'
        }        
    },
    filterGoogleMaps: {
        get() {
            // return 'https://www.google.com/maps/embed/v1/place?key=AIzaSyCjT4wt-UGuYEOUga4yB_ZA6MUDVEcQ_bI&q=' + this.CyDetail.maps.googleMaps.split('maps/')[1];
            return 'https://www.google.com/maps/embed/v1/place?key=AIzaSyCjT4wt-UGuYEOUga4yB_ZA6MUDVEcQ_bI&q=' + this.CyDetail.name.official +'&zoom=10'
        }
    }
  },
  created() {
    this.getCountriesList();
  },
  methods: {
    sortedClass(key) {
        return this.sort.key === key ? `sorted ${this.sort.isAsc ? 'asc-char' : 'desc-char'}` : '';
    },
    sortBy(key) {
        this.sort.isAsc = this.sort.key === key ? !this.sort.isAsc : false;
        this.sort.key = key;
    },
    async getCountriesList() {
        $('#Loading_dialog').modal('show');
        await this.$http
        .get("https://restcountries.com/v3.1/all")
        .then(function(data) {
            $('#Loading_dialog').modal('hide');
            if (data.status === 200) {
                this.CountriesList = data.data;
            } else {
                this.showMsg(['Error : ' + data.status]);
            }
        })
        .catch(function (err) {
            $('#Loading_dialog').modal('hide');
            alert(err);
        });
    },
    showCyDetail(cy) {
        this.CyDetail = cy;        
        $('#CyDetailModal').modal('show');
    },
    setPage(val) {
        this.currentPage = val;
    },
    showMsg(Msg) {
        let msgHtml = "";
        Msg.forEach((item) => {
            msgHtml += "<p>" + item + "</p>";
        });
        $("#messageDiv").html(msgHtml)
        $('#messageModal').modal('show');
    },
  }
};
</script>

<style>
    #map {
        height: 100%;
    }
    /* searchInput */
    .searchInput {
        max-width: calc(12em + 10px);
        min-height: calc(1.5em + .5rem + 2px);
        height: calc(1.5em + .5rem + 2px);
        line-height: 1.5;
    }
    #toolDiv {
        display: flow-root;
        padding: 0rem 0rem 0.5rem 0rem;
    }
    
    .tipText {
        font-size: .75rem;
        color: #dc3545;
    }

    .el-footer {
        margin-top: 20px;
        text-align: center;
    }
    
    .tableDetail, .tableOther {
        white-space: nowrap;
        width: 100%;
    }
    /* img */
    .table tbody td:first-child,
    .tableDetail tbody td:first-child {
        /* width: 80px;
        vertical-align: middle; */
        text-align: center;
    }
    .table tbody td img,
    .tableDetail tbody td img {
        max-width: 64px;
        max-height: 64px;
    }

    /* table sort */
    table th.sorted.desc::after {
        font-family: bootstrap-icons !important;
        font-weight: normal !important;
        vertical-align: text-bottom;
        line-height: 1;
        content: '\f514';
        font-size: 1.125rem;
    }

    table th.sorted.asc-char::after {
        font-family: bootstrap-icons !important;
        font-weight: normal !important;
        vertical-align: text-bottom;
        line-height: 1;
        content: '\f510';
        font-size: 1.125rem;
    }

    table th.sorted.desc-char::after {
        font-family: bootstrap-icons !important;
        font-weight: normal !important;
        vertical-align: text-bottom;
        line-height: 1;
        content: '\f50f';
        font-size: 1.125rem;
    }
    /* icon-link */
    td.icon-link {
        cursor: pointer;
    }
    td.icon-link:hover {
        color: hsla(220, 85%, 60%, 1);
    }
    /*  */
    #CyDetailModal .modal-dialog {
        overflow-y: initial !important
    }
    #CyDetailModal .modal-body {
        height: 70vh;
        overflow-y: auto;
    }
</style>