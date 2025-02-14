<template>
  <page-list
    show-tag-columns
    show-tag-filter
    :list="list"
    :columns="columns"
    :single-actions="singleActions"
    :group-actions="groupActions"
    :export-data-options="exportDataOptions" />
</template>

<script>
import * as R from 'ramda'
import { mapGetters } from 'vuex'
import ColumnsMixin from '../mixins/columns'
import SingleActionsMixin from '../mixins/singleActions'
import ListMixin from '@/mixins/list'
import WindowsMixin from '@/mixins/windows'
import { getDescriptionFilter } from '@/utils/common/tableFilter'

export default {
  name: 'LoadbalancerclusterList',
  mixins: [WindowsMixin, ListMixin, ColumnsMixin, SingleActionsMixin],
  props: {
    id: String,
    getParams: {
      type: [Function, Object],
    },
  },
  data () {
    return {
      list: this.$list.createList(this, {
        id: this.id,
        resource: 'loadbalancerclusters',
        getParams: this.getParam,
        filterOptions: {
          name: {
            label: this.$t('network.text_21'),
            filter: true,
            formatter: val => {
              return `name.contains(${val})`
            },
          },
          description: getDescriptionFilter(),
          region: {
            label: this.$t('dashboard.text_101'),
          },
          zone: {
            label: this.$t('compute.text_270'),
          },
        },
      }),
      exportDataOptions: {
        items: [
          { label: 'ID', key: 'id' },
          { label: this.$t('network.text_21'), key: 'name' },
          { label: this.$t('network.text_199'), key: 'region' },
          { label: this.$t('common_715'), key: 'user_tags' },
        ],
      },
      groupActions: [
        {
          label: this.$t('network.text_26'),
          permission: 'lb_loadbalancercluster_create',
          action: () => {
            this.createDialog('LoadbalancerclusterCreateDialog', {
              title: this.$t('network.text_357'),
              data: this.list.selectedItems,
              onManager: this.onManager,
              refresh: this.refresh,
            })
          },
          meta: () => {
            return {
              buttonType: 'primary',
              validate: this.$isAdminMode,
            }
          },
        },
        {
          label: this.$t('table.action.set_tag'),
          action: () => {
            this.createDialog('SetTagDialog', {
              data: this.list.selectedItems,
              columns: this.columns,
              onManager: this.onManager,
              mode: 'add',
              params: {
                resources: 'loadbalancercluster',
              },
              tipName: this.$t('network.text_19'),
            })
          },
          meta: () => {
            return {
              validate: this.list.selectedItems.length > 0,
            }
          },
        },
      ],
    }
  },
  computed: {
    ...mapGetters(['isAdminMode']),
  },
  created () {
    this.initSidePageTab('Loadbalancercluster-detail')
    this.list.fetchData()
  },
  methods: {
    getParam () {
      const ret = {
        ...(R.is(Function, this.getParams) ? this.getParams() : this.getParams),
      }
      return ret
    },
    hasService ($userInfo, service) {
      if ($userInfo && $userInfo.services && $userInfo.services.length) {
        const results = $userInfo.services.filter(item => {
          return item.type === service && item.status === true
        })
        return results && results.length > 0
      }
      return false
    },
    handleOpenSidepage (row) {
      this.sidePageTriggerHandle(this, 'LoadbalancerclusterSidePage', {
        id: row.id,
        resource: 'loadbalancerclusters',
        getParams: this.getParam,
      }, {
        list: this.list,
      })
    },
  },
}
</script>
