<template>
  <m-list-construction :title="$t('UDF Resources')">
    <template slot="conditions">
      <m-conditions @on-conditions="_onConditions">
        <template slot="button-group">
          <x-button type="ghost" size="small"  @click="_uploading" v-ps="['GENERAL_USER']">{{$t('Upload UDF Resources')}}</x-button>
        </template>
      </m-conditions>
    </template>
    <template slot="content">
      <template v-if="udfResourcesList.length">
        <m-list :udf-resources-list="udfResourcesList" :page-no="searchParams.pageNo" :page-size="searchParams.pageSize">
        </m-list>
        <div class="page-box">
          <x-page :current="parseInt(searchParams.pageNo)" :total="total" :page-size="searchParams.pageSize" show-elevator @on-change="_page"></x-page>
        </div>
      </template>
      <template v-if="!udfResourcesList.length">
        <m-no-data></m-no-data>
      </template>
      <m-spin :is-spin="isLoading">
      </m-spin>
    </template>
  </m-list-construction>
</template>
<script>
  import _ from 'lodash'
  import { mapActions } from 'vuex'
  import mList from './_source/list'
  import mSpin from '@/module/components/spin/spin'
  import { findComponentDownward } from '@/module/util/'
  import mNoData from '@/module/components/noData/noData'
  import listUrlParamHandle from '@/module/mixin/listUrlParamHandle'
  import mConditions from '@/module/components/conditions/conditions'
  import mListConstruction from '@/module/components/listConstruction/listConstruction'

  export default {
    name: 'resource-list-index-UDF',
    data () {
      return {
        total: null,
        isLoading: false,
        udfResourcesList: [],
        searchParams: {
          pageSize: 10,
          pageNo: 1,
          searchVal: '',
          type: 'UDF'
        }
      }
    },
    mixins: [listUrlParamHandle],
    props: {},
    methods: {
      ...mapActions('resource', ['getResourcesListP']),
      /**
       * File Upload
       */
      _uploading () {
        findComponentDownward(this.$root, 'roof-nav')._fileUpdate('UDF')
      },
      _onConditions (o) {
        this.searchParams = _.assign(this.searchParams, o)
        this.searchParams.pageNo = 1
      },
      _page (val) {
        this.searchParams.pageNo = val
      },
      _updateList () {
        this.searchParams.pageNo = 1
        this.searchParams.searchVal = ''
        this._debounceGET()
      },
      _getList (flag) {
        this.isLoading = !flag
        this.getResourcesListP(this.searchParams).then(res => {
          this.udfResourcesList = []
          this.udfResourcesList = res.totalList
          this.total = res.total
          this.isLoading = false
        }).catch(e => {
          this.isLoading = false
        })
      }
    },
    watch: {
      // router
      '$route' (a) {
        // url no params get instance list
        this.searchParams.pageNo = _.isEmpty(a.query) ? 1 : a.query.pageNo
      }
    },
    created () {
    },
    mounted () {
    },
    components: { mListConstruction, mConditions, mList, mSpin, mNoData }
  }
</script>
