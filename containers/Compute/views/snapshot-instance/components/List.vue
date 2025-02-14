<template>
  <page-list
    show-tag-columns
    show-tag-filter
    :list="list"
    :columns="columns"
    :group-actions="groupActions"
    :single-actions="singleActions"
    :export-data-options="exportDataOptions"
    :showSearchbox="showSearchbox"
    :showGroupActions="showGroupActions" />
</template>

<script>
import { steadyStatus } from '../constants'
import ColumnsMixin from '../mixins/columns'
import SingleActionsMixin from '../mixins/singleActions'
import WindowsMixin from '@/mixins/windows'
import GlobalSearchMixin from '@/mixins/globalSearch'
import ListMixin from '@/mixins/list'
import {
  getNameFilter,
  getTenantFilter,
  getStatusFilter,
  getDomainFilter,
  getOsArchFilter,
  getRegionFilter,
  getDescriptionFilter,
} from '@/utils/common/tableFilter'

export default {
  name: 'InstanceSnapshotList',
  mixins: [WindowsMixin, ListMixin, GlobalSearchMixin, ColumnsMixin, SingleActionsMixin],
  props: {
    id: String,
    getParams: {
      type: [Function, Object],
    },
    cloudEnv: String,
  },
  data () {
    return {
      list: this.$list.createList(this, {
        id: this.id,
        resource: 'instance_snapshots',
        getParams: this.getParam,
        steadyStatus,
        filterOptions: {
          id: {
            label: this.$t('table.title.id'),
          },
          name: getNameFilter(),
          description: getDescriptionFilter(),
          status: getStatusFilter('snapshot'),
          guest_id: {
            label: this.$t('res.server'),
          },
          projects: getTenantFilter(),
          project_domains: getDomainFilter(),
          region: getRegionFilter(),
          os_arch: getOsArchFilter(),
        },
        responseData: this.responseData,
        hiddenColumns: ['created_at'],
      }),
      exportDataOptions: {
        items: [
          { label: 'ID', key: 'id' },
          { label: this.$t('table.title.name'), key: 'name' },
          { label: this.$t('table.title.sub_snapshot'), key: 'snapshots' },
          { label: this.$t('table.title.snapshot_size'), key: 'size' },
          { label: this.$t('common.status'), key: 'status' },
          { label: this.$t('res.project'), key: 'tenant' },
          { label: this.$t('res.server'), key: 'guest' },
          { label: this.$t('table.title.user_tag'), key: 'user_tags' },
          { label: this.$t('table.title.os_arch'), key: 'os_arch' },
        ],
      },
      groupActions: [
        {
          label: this.$t('table.action.set_tag'),
          action: () => {
            this.createDialog('SetTagDialog', {
              data: this.list.selectedItems,
              columns: this.columns,
              onManager: this.onManager,
              mode: 'add',
              params: {
                resources: 'instance_snapshot',
              },
              tipName: this.$t('compute.text_462'),
            })
          },
          meta: () => {
            return {
              validate: this.list.selected.length,
              tooltip: null,
            }
          },
        },
        {
          label: this.$t('compute.perform_delete'),
          permission: 'snapshots_delete',
          action: () => {
            this.createDialog('DeleteResDialog', {
              vm: this,
              data: this.list.selectedItems,
              columns: this.columns,
              onManager: this.onManager,
              title: this.$t('compute.perform_delete'),
              name: this.$t('compute.text_462'),
            })
          },
          meta: () => {
            const ret = {
              validate: this.list.selected.length,
              tooltip: null,
            }
            if (this.list.selectedItems.some(item => !item.can_delete)) {
              ret.validate = false
              return ret
            }
            if (this.list.selectedItems.some(item => item.is_sub_snapshot)) {
              ret.validate = false
              ret.tooltip = this.$t('compute.text_1062')
              return ret
            }
            return ret
          },
        },
      ],
    }
  },
  watch: {
    cloudEnv (val) {
      this.$nextTick(() => {
        this.list.fetchData(0)
      })
    },
  },
  created () {
    this.initSidePageTab('snapshot-detail')
    this.list.fetchData()
  },
  methods: {
    getParam () {
      const ret = {
        details: true,
        ...this.getParams,
      }
      if (this.cloudEnv) ret.cloud_env = this.cloudEnv
      return ret
    },
    handleOpenSidepage (row) {
      this.sidePageTriggerHandle(this, 'SnapshotInstanceSidePage', {
        id: row.id,
        resource: 'instance_snapshots',
        getParams: this.getParam,
        steadyStatus: this.list.steadyStatus,
      }, {
        list: this.list,
        type: 'instance',
      })
    },
  },
}
</script>
