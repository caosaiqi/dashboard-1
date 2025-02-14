<template>
  <div>
    <loading-block v-if="loading" />
    <!-- <detail
      v-else
      :data="data"
      :base-info="baseInfo"
      :on-manager="onManager"
      :showDesc="false"
      :hiddenKeys="['id', 'name', 'status', 'project_domain', 'tenant', 'created_at', 'updated_at']" /> -->
    <template v-else>
      <a-row :gutter="16">
        <a-col :md="12" :xl="8" :xxl="6" v-for="obj in baseInfo" :key="obj.field">
          <usage-quota :title="obj.title" :value="obj.value()" />
        </a-col>
      </a-row>
      <a-row :gutter="16">
        <a-col :md="12" :xl="8" :xxl="6" v-for="obj in ringInfo" :key="obj.field">
          <usage-ring :title="obj.title" :field="obj.field" :options="obj.options" />
        </a-col>
      </a-row>
    </template>
  </div>
</template>

<script>
import * as R from 'ramda'
import { USAGE_CONFIG_MAP, USAGE_RING_DATAS } from './constants'
import UsageQuota from './components/UsageQuota'
import UsageRing from './components/UsageRing'

export default {
  name: 'CloudenvUsageSidepage',
  components: {
    UsageQuota,
    UsageRing,
  },
  props: {
    onManager: {
      type: Function,
      required: true,
    },
    resId: {
      type: String,
      required: true,
    },
    resource: {
      type: String,
      required: true,
    },
  },
  data () {
    return {
      data: {},
      loading: false,
      baseInfo: [],
      ringInfo: [],
    }
  },
  created () {
    this.fetchData()
  },
  methods: {
    initBaseInfo () {
      const isAdminMode = this.$store.getters.isAdminMode
      const isDomainMode = this.$store.getters.isDomainMode
      const baseInfo = []
      Object.keys(USAGE_CONFIG_MAP).forEach((key) => {
        const item = USAGE_CONFIG_MAP[key] || {}
        if (!item.noPerfix) {
          if (isAdminMode) key = `all.${key}`
          if (isDomainMode) key = `domain.${key}`
        }
        baseInfo.push({
          field: key,
          title: item.zh_cn || key,
          value: () => {
            return R.is(Function, item.formatter) ? item.formatter(this.data[key]) : `${this.data[key] || 0} ${item.unit || ''}`
          },
        })
      })
      return baseInfo
    },
    initRingInfo () {
      const isAdminMode = this.$store.getters.isAdminMode
      const isDomainMode = this.$store.getters.isDomainMode
      const ringInfo = []
      const getUnuse = (sum = 0, use = 0) => {
        return sum - use
      }
      USAGE_RING_DATAS.forEach((obj) => {
        let key = ''
        if (!obj.noPerfix) {
          if (isAdminMode) key = 'all'
          if (isDomainMode) key = 'domain'
        }
        const sum = key ? `${key}.${obj.sum}` : obj.sum
        const use = key ? `${key}.${obj.use}` : obj.use

        ringInfo.push({
          field: sum,
          title: obj.name,
          options: {
            label: {
              sum: obj.sum_label || this.$t('common_234'),
              use: obj.use_label,
              unuse: obj.unuse_label,
            },
            value: {
              sum: this.data[sum],
              use: this.data[use],
              unuse: getUnuse(this.data[sum], this.data[use]),
            },
          },
        })
      })
      return ringInfo
    },
    async fetchData () {
      try {
        this.loading = true
        const params = { scope: this.$store.getters.scope }
        const { data } = await new this.$Manager('usages', 'v2').getSpecific({
          id: this.resource,
          spec: this.resId,
          params,
        })
        this.data = data
        this.baseInfo = this.initBaseInfo()
        this.ringInfo = this.initRingInfo()
        this.loading = false
      } catch (error) {
        this.loading = false
      }
    },
  },
}
</script>

<style lang="less" scoped>
::v-deep .detail-item {
  .detail-item-title {
    width: 220px !important;
  }
  .detail-item-value {
    margin-left: 220px;
  }
}
</style>
