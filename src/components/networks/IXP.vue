<script setup>
import { QCard, QTabs, QTab, QSeparator, QTabPanels, QTabPanel } from 'quasar'
import { useRoute, useRouter } from 'vue-router'
import Tr from '@/i18n/translation'
import { ref, watch, computed, onMounted, inject } from 'vue'
import report from '@/plugins/report'
import { useI18n } from 'vue-i18n'
import IXPOverview from '@/components/networks/ixp/IXPOverview.vue'
import IXPMonitoring from '@/components/networks/ixp/IXPMonitoring.vue'
import IXPRouting from '@/components/networks/ixp/IXPRouting.vue'
import IXPPeering from '@/components/networks/ixp/IXPPeering.vue'
import IXPCustom from '@/components/networks/ixp/IXPCustom.vue'
import DateTimePicker from '@/components/DateTimePicker.vue'
import { AS_FAMILY } from '@/plugins/IhrApi'

const { t } = useI18n()

const iyp_api = inject('iyp_api')

const route = useRoute()
const router = useRouter()

const timeRange = route.query.last ? route.query.last : 3

let {
  interval,
  utcString,
  fetch,
  reportDateFmt,
  minDate,
  maxDate,
  setReportDate,
  startTime,
  endTime
} = report(timeRange)

if (route.query.date && route.query.date != utcString(maxDate.value).split('T')[0]) {
  setReportDate(new Date(route.query.date))
}

const activeMenu = route.query.active ? route.query.active : 'overview'

const routeHash = ref(route.hash)
const loadingStatus = ref(false)
const ixpNumber = ref(Number(route.params.id.replace('IXP', '')))
const ixpName = ref(null)
const menu = ref(activeMenu)
const caidaId = ref(null)
const addressFamily = ref(route.query.af == undefined ? 4 : route.query.af)

const getInfo = () => {
  const query = `MATCH (:PeeringdbIXID {id: $ixp})<-[:EXTERNAL_ID]-(i:IXP)
    OPTIONAL MATCH (i)-[:EXTERNAL_ID]-(c:CaidaIXID)
    RETURN i.name AS name, c.id AS caida_id`
  return [{ statement: query, parameters: { ixp: ixpNumber.value } }]
}

const fetchData = async () => {
  let queries = getInfo()

  loadingStatus.value = true

  try {
    let res = await iyp_api.run(queries)
    ixpName.value = res[0][0].name
    caidaId.value = res[0][0].caida_id + 1
    loadingStatus.value = false
  } catch (e) {
    loadingStatus.value = false
    return
  }
}

const pushRoute = () => {
  router.push(
    Tr.i18nRoute({
      replace: true,
      query: Object.assign({}, route.query, {
        af: family.value,
        last: interval.value.dayDiff(),
        date: utcString(interval.value.end).split('T')[0],
        active: menu.value ? menu.value : activeMenu
      })
    })
  )
}

const family = computed(() => {
  return addressFamily.value == 6 ? AS_FAMILY.v6 : AS_FAMILY.v4
})

const pageTitle = computed(() => {
  return `IXP${ixpNumber.value} - ${ixpName.value}`
})

watch(addressFamily, () => {
  pushRoute()
})
watch(
  () => route.params.id,
  (ixp) => {
    const newIXP = Number(ixp.replace('IXP', ''))
    if (newIXP != ixpNumber.value) {
      ixpNumber.value = newIXP
      if (ixpNumber.value) {
        pushRoute()
        fetchData()
      }
    }
  }
)
watch(interval, () => {
  pushRoute()
})
watch(
  () => route.query.active,
  (active) => {
    if (active != menu.value) {
      menu.value = active
    }
  }
)
watch(menu, () => {
  if ('display' in route.query && !route.hash.includes('#')) {
    delete route.query.display
  }
  pushRoute()
})
onMounted(() => {
  if (ixpNumber.value) {
    pushRoute()
    fetchData()
  } else {
    router.push(
      Tr.i18nRoute({
        name: 'network'
      })
    )
  }
})
</script>

<template>
  <div id="IHR_as-and-ixp-container" ref="ihrAsAndIxpContainer" class="IHR_char-container">
    <h1 class="text-center">
      {{ pageTitle }}
    </h1>
    <h3 class="text-center">
      <div v-if="['monitoring', 'custom'].includes(menu)">
        {{ interval.dayDiff() }}-day report ending on {{ reportDateFmt }}
        <DateTimePicker
          :min="minDate"
          :max="maxDate"
          :value="endTime"
          hide-time
          @input="setReportDate"
        />
      </div>
      <div v-else>Weekly report</div>
    </h3>
    <QCard flat>
      <QTabs
        v-model="menu"
        dense
        indicator-color="secondary"
        active-color="primary"
        align="justify"
        narrow-indicator
      >
        <QTab name="overview"> Overview </QTab>
        <QTab name="monitoring"> Monitoring </QTab>
        <QTab name="routing"> Routing </QTab>
        <QTab name="peering"> Peering </QTab>
        <QTab name="custom"> Custom </QTab>
      </QTabs>
      <QSeparator />
      <QTabPanels v-if="pageTitle" v-model="menu">
        <QTabPanel name="overview">
          <IXPOverview :ixp-number="ixpNumber" />
        </QTabPanel>
        <QTabPanel name="monitoring">
          <IXPMonitoring
            :start-time="startTime"
            :end-time="endTime"
            :caida-id="caidaId"
            :family="family"
            :page-title="pageTitle"
            :interval="interval"
            :peeringdb-id="ixpNumber"
          />
        </QTabPanel>
        <QTabPanel name="routing">
          <IXPRouting :ixp-number="ixpNumber" :page-title="pageTitle" />
        </QTabPanel>
        <QTabPanel name="peering">
          <IXPPeering :ixp-number="ixpNumber" :page-title="pageTitle" />
        </QTabPanel>
        <QTabPanel name="custom">
          <IXPCustom
            :start-time="startTime"
            :end-time="endTime"
            :caida-id="caidaId"
            :family="family"
            :page-title="pageTitle"
            :peeringdb-id="ixpNumber"
            :interval="interval"
            :hash="routeHash"
          />
        </QTabPanel>
      </QTabPanels>
    </QCard>
  </div>
</template>

<style>
.cards {
  display: inline-block;
}
</style>
