<script setup>
import { RouterLink, useRoute } from 'vue-router'
import Tr from '@/i18n/translation'
import { ref, watch, onMounted } from 'vue'
import SearchBar from '@/components/search/SearchBar.vue'
import Country from '@/components/networks/Country.vue'
import { isoCountries } from '@/plugins/countryName'
import Feedback from '@/components/Feedback.vue'
import '@/styles/chart.css'

const route = useRoute()

const countryString = ref(null)

const init = () => {
  if (route.params.cc) {
    countryString.value = route.params.cc in isoCountries ? route.params.cc : null
  }
}

watch(
  () => route.params.cc,
  () => {
    init()
  }
)

onMounted(() => {
  init()
})
</script>

<template>
  <div id="IHR_as-and-ixp-container" ref="ihrAsAndIxpContainer" class="IHR_char-container">
    <div v-if="route.params.cc">
      <Country v-if="countryString" />
    </div>
    <div v-else>
      <div>
        <h1 class="text-center q-pa-xl">Country Report</h1>
        <div class="q-pa-md flex flex-center">
          <div style="width: 100%; max-width: 500px">
            <SearchBar
              bg="white"
              label="grey-8"
              input="black"
              label-txt="Enter a country name"
              :no-a-s="true"
              :no-i-x-p="true"
              :no-prefix="true"
              :no-host-name="true"
              :no-tag="true"
              :no-rank="true"
            />
          </div>
        </div>
      </div>
      <div class="q-pa-lg">
        <div class="row q-pa-lg column items-center">
          <div class="col-6">
            <h3>Examples</h3>
          </div>
        </div>
        <div class="row justify-center">
          <div class="row examples">
            <ul class="ul_styles">
              <li>
                <RouterLink
                  :to="Tr.i18nRoute({ name: 'country', params: { cc: 'JP' } })"
                  class="IHR_delikify"
                >
                  Japan
                </RouterLink>
              </li>
              <li>
                <RouterLink
                  :to="Tr.i18nRoute({ name: 'country', params: { cc: 'FR' } })"
                  class="IHR_delikify"
                >
                  France
                </RouterLink>
              </li>
              <li>
                <RouterLink
                  :to="Tr.i18nRoute({ name: 'country', params: { cc: 'US' } })"
                  class="IHR_delikify"
                >
                  United States
                </RouterLink>
              </li>
            </ul>
            <ul class="ul_styles">
              <li>
                <RouterLink
                  :to="Tr.i18nRoute({ name: 'country', params: { cc: 'BR' } })"
                  class="IHR_delikify"
                >
                  Brazil
                </RouterLink>
              </li>
              <li>
                <RouterLink
                  :to="Tr.i18nRoute({ name: 'country', params: { cc: 'DE' } })"
                  class="IHR_delikify"
                >
                  Germany
                </RouterLink>
              </li>
              <li>
                <RouterLink
                  :to="Tr.i18nRoute({ name: 'country', params: { cc: 'CN' } })"
                  class="IHR_delikify"
                >
                  China
                </RouterLink>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
  <Feedback />
</template>

<style>
.examples {
  column-gap: 30px;
}
@media screen and (max-width: 500px) {
  .examples {
    flex-direction: column;
  }
}
.ul_styles {
  padding: 0;
  margin: 0;
  list-style-position: inside;
}
</style>
