<template>
  <div class="filter_modal" @click="handleClickOutside">
    <div class="filter_wrapper" @click.stop>
      <header class="filter_header">
        <div class="filter_header_left">
          <img v-if="data.iconSrc" class="filter_image" :src="getImagePath(data.iconSrc)" alt="" />
          <p class="filter_header_text">{{ data.name }}</p>
        </div>
        <close-svg :on-click="closeSelectModal" />
      </header>
      <div class="filter_content">
        <div class="filter_content_wrapper">
          <div v-if="data.name === 'Create Custom Filter'" class="filter_content_dropdown">
            <p class="dropdown_title">Filter Name</p>
            <div class="dropdown_body_wrapper">
              <input
                class="dropdown_body"
                :class="{ second_one: data.name === 'Average Order Value' }"
                :type="'number'"
                :placeholder="SecondPlaceholderMap(data?.name)"
                v-model="inputValue"
                @input="validateInput"
              />
              <div v-if="data.name === 'Average Order Value'" class="absolute_placehopder">$</div>
              <p v-if="secondInputError" class="error_message">
                Invalid input: Please enter a numeric value
              </p>
            </div>
          </div>
          <template v-for="(item, index) in actionsData" :key="item">
            <div v-if="index" class="condition_indicator">
              <p>{{ item.condition }}</p>
            </div>
            <div v-if="data.name === 'Create Custom Filter'" class="filter_content_dropdown">
              <div class="remove_action_wrapper">
                <p class="dropdown_title">Action</p>
                <close-svg v-if="index" :on-click="() => removeCustomFilter(item.index)" />
              </div>
              <div class="dropdown_body_wrapper">
                <div
                  v-if="noDropdown(data.name)"
                  class="arrow_button_wrapper"
                  @click="toggleDropdown('action')"
                >
                  <arrow-svg :isDropdownOpen="isDropdownOpen.action" />
                </div>
                <input
                  class="dropdown_body"
                  :type="'text'"
                  :placeholder="'Select'"
                  @blur="handleBlur('action')"
                  @focus="openDropdown('action')"
                  @input="filterItems('action')"
                  v-model="selectedItem.action"
                />
                <p v-if="secondInputError" class="error_message">
                  Invalid input: Please enter a numeric value
                </p>
                <transition name="dropdown">
                  <ul
                    v-show="isDropdownOpen.action && noDropdown(data.name)"
                    class="dropdown_menu_wrapper"
                  >
                    <li
                      v-for="(items, category) in actionItems"
                      :key="category"
                      class="dropdown_menu_item"
                      :class="{ activeClass: category === selectedItem['action'] }"
                      @click="selectItem(`${category}`, 'action')"
                    >
                      {{ category }}
                    </li>
                  </ul>
                </transition>
              </div>
            </div>
            <div class="filter_content_dropdown">
              <p class="dropdown_title">{{ labelMap(data?.name) }}</p>
              <div class="dropdown_body_wrapper">
                <div
                  v-if="noDropdown(data.name)"
                  class="arrow_button_wrapper"
                  @click="toggleDropdown('default')"
                >
                  <arrow-svg :isDropdownOpen="isDropdownOpen.default" />
                </div>
                <input
                  class="dropdown_body"
                  :type="numberInput(data.name) ? 'number' : 'text'"
                  :placeholder="placeholderMap(data?.name)"
                  @blur="handleBlur('default')"
                  @focus="openDropdown('default')"
                  @input="filterItems('default')"
                  v-model="selectedItem.default"
                />
                <p v-if="secondInputError" class="error_message">
                  Invalid input: Please enter a numeric value
                </p>
                <transition name="dropdown">
                  <ul
                    v-show="isDropdownOpen.default && noDropdown(data.name)"
                    class="dropdown_menu_wrapper"
                  >
                    <li
                      v-for="item in dropdownItems"
                      :key="item"
                      class="dropdown_menu_item"
                      :class="{ activeClass: item === selectedItem['default'] }"
                      @click="selectItem(item, 'default')"
                    >
                      {{ item }}
                    </li>
                  </ul>
                </transition>
              </div>
            </div>
            <div v-if="data.showSign" class="filter_content_dropdown">
              <p class="dropdown_title">{{ SecondLabelMap(data?.name) }}</p>
              <div class="dropdown_body_wrapper">
                <input
                  class="dropdown_body"
                  :class="{ second_one: data.name === 'Average Order Value' }"
                  :type="'number'"
                  :placeholder="SecondPlaceholderMap(data?.name)"
                  v-model="inputValue"
                  @input="validateInput"
                />
                <div v-if="data.name === 'Average Order Value'" class="absolute_placehopder">$</div>
                <p v-if="secondInputError" class="error_message">
                  Invalid input: Please enter a numeric value
                </p>
              </div>
            </div>
          </template>
        </div>
      </div>
      <footer class="filter_footer">
        <div class="footer_buttons">
          <div class="footer_button" @click="cancel">
            <p class="footer_button_text">Cancel</p>
          </div>
          <div v-if="data.name === 'Create Custom Filter'" class="button-group">
            <div class="footer_button" @click="addCustomFilter('and')">
              <p class="footer_button_text">+ AND Condition</p>
            </div>
            <!-- <div class="footer_button" @click="addCustomFilter('or')">
              <p class="footer_button_text">OR</p>
            </div> -->
          </div>
          <div
            class="footer_button primary_button"
            :class="{ disabled_me: Boolean(!selectedItem.default) }"
            @click="next"
          >
            <p class="footer_button_text">Next</p>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, onBeforeUnmount, watch } from 'vue'
import ArrowSvg from './ArrowSvg.vue'
import CloseSvg from './CloseSvg.vue'

type DataItem = {
  type: string
  category: string
  name: string
  segment: string | null
  sqlSegment: string | null
  needsMostFrequentValues: boolean
  sqlFilterValue?: any
  acceptedValues?: string
  suggestedValuesCallback?: any
  unionOfSegments?: string[]
  sqlFilter?: any
  suggestedValuesApi?: string
}

type GroupedData = {
  [category: string]: DataItem[]
}

const data: DataItem[] = [
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Actions In Visit',
    segment: 'actions',
    sqlSegment: 'log_visit.visit_total_actions',
    needsMostFrequentValues: true
  },
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Days since first visit',
    segment: 'daysSinceFirstVisit',
    sqlSegment: 'log_visit.visitor_seconds_since_first',
    needsMostFrequentValues: true,
    sqlFilterValue: {}
  },
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Days since last visit',
    segment: 'daysSinceLastVisit',
    sqlSegment: 'FLOOR(log_visit.visitor_seconds_since_last / 86400)',
    needsMostFrequentValues: true
  },
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Events',
    segment: 'events',
    sqlSegment: 'log_visit.visit_total_events',
    needsMostFrequentValues: true,
    acceptedValues: 'To select all visits who triggered an Event, use: &segment=events>0'
  },
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Number of Interactions',
    segment: 'interactions',
    sqlSegment: 'log_visit.visit_total_interactions',
    needsMostFrequentValues: true,
    acceptedValues: 'Any positive integer',
    suggestedValuesCallback: {}
  },
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Number of Internal Searches',
    segment: 'searches',
    sqlSegment: 'log_visit.visit_total_searches',
    needsMostFrequentValues: true,
    acceptedValues: 'To select all visits who used internal Site Search, use: &segment=searches>0'
  },
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Number of visits',
    segment: 'visitCount',
    sqlSegment: 'log_visit.visitor_count_visits',
    needsMostFrequentValues: true
  },
  {
    type: 'metric',
    category: 'Visitors',
    name: 'Visit Duration (in seconds)',
    segment: 'visitDuration',
    sqlSegment: 'log_visit.visit_total_time',
    needsMostFrequentValues: true
  },
  {
    type: 'dimension',
    category: 'Visitors',
    name: 'Browser',
    segment: 'browserName',
    sqlSegment: 'log_visit.config_browser_name',
    needsMostFrequentValues: false,
    sqlFilterValue: {},
    acceptedValues: 'FireFox, Internet Explorer, Chrome, Safari, Opera etc.',
    suggestedValuesCallback: {}
  },
  {
    type: 'dimension',
    category: 'Visitors',
    name: 'Device model',
    segment: 'deviceModel',
    sqlSegment: 'log_visit.config_device_model',
    needsMostFrequentValues: true,
    acceptedValues: 'iPad, Nexus 5, Galaxy S5, Fire TV, etc.'
  },
  {
    type: 'dimension',
    category: 'Visitors',
    name: 'Device type',
    segment: 'deviceType',
    sqlSegment: 'log_visit.config_device_type',
    needsMostFrequentValues: true,
    sqlFilter: {},
    acceptedValues:
      'desktop, smartphone, tablet, feature phone, console, tv, car browser, smart display, camera, portable media player, phablet, smart speaker, wearable, peripheral',
    suggestedValuesCallback: {}
  },
  {
    type: 'dimension',
    category: 'Visitors',
    name: 'Operating system',
    segment: 'operatingSystemName',
    sqlSegment: 'log_visit.config_os',
    needsMostFrequentValues: false,
    sqlFilterValue: {},
    acceptedValues: 'Windows, Linux, Mac, Android, iOS etc.',
    suggestedValuesCallback: {}
  },
  {
    type: 'dimension',
    category: 'Visitors',
    name: 'Resolution',
    segment: 'resolution',
    sqlSegment: 'log_visit.config_resolution',
    needsMostFrequentValues: true,
    acceptedValues: '1280x1024, 800x600, etc.'
  },
  {
    type: 'dimension',
    category: 'Visitors',
    name: 'Site time — hour (time of last action)',
    segment: 'visitServerHour',
    sqlSegment: 'HOUR(log_visit.visit_last_action_time)',
    needsMostFrequentValues: true,
    acceptedValues: '0, 1, 2, 3, ..., 20, 21, 22, 23'
  },
  {
    type: 'dimension',
    category: 'Visitors',
    name: 'Visit type',
    segment: 'visitorType',
    sqlSegment: 'log_visit.visitor_returning',
    needsMostFrequentValues: true,
    sqlFilterValue: {},
    acceptedValues:
      'new, returning, returningCustomer. For example, to select all visitors who have returned to the website, including those who have bought something in their previous visits, the API request would contain "&segment=visitorType==returning,visitorType==returningCustomer"',
    suggestedValuesCallback: {}
  },
  {
    type: 'dimension',
    category: 'Visitor location',
    name: 'City',
    segment: 'city',
    sqlSegment: 'log_visit.location_city',
    needsMostFrequentValues: true,
    acceptedValues: 'Sydney, Sao Paolo, Rome, etc.'
  },
  {
    type: 'dimension',
    category: 'Visitor location',
    name: 'Continent',
    segment: 'continentCode',
    sqlSegment: 'log_visit.location_country',
    needsMostFrequentValues: true,
    sqlFilter: 'Piwik\\Plugins\\UserCountry\\UserCountry::getCountriesForContinent',
    acceptedValues: 'eur, asi, amc, amn, ams, afr, ant, oce'
  },
  {
    type: 'dimension',
    category: 'Visitor location',
    name: 'Country',
    segment: 'countryName',
    sqlSegment: 'log_visit.location_country',
    needsMostFrequentValues: false,
    sqlFilterValue: {},
    acceptedValues: 'Germany, France, Spain, ...'
  },
  {
    type: 'dimension',
    category: 'Behaviour',
    name: 'Action Type',
    segment: 'actionType',
    sqlSegment: 'log_action.type',
    needsMostFrequentValues: true,
    sqlFilter: {},
    acceptedValues:
      'A type of action, such as: pageviews, contents, sitesearches, events, outlinks, downloads',
    suggestedValuesCallback: {}
  },
  {
    type: 'dimension',
    category: 'Behaviour',
    name: 'Action URL',
    segment: 'actionUrl',
    sqlSegment: null,
    needsMostFrequentValues: true,
    unionOfSegments: ['pageUrl', 'downloadUrl', 'outlinkUrl', 'eventUrl'],
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment'
  },
  {
    type: 'dimension',
    category: 'Behaviour',
    name: 'Category (Site Search)',
    segment: null,
    sqlSegment: 'log_link_visit_action.search_cat',
    needsMostFrequentValues: true
  },
  {
    type: 'dimension',
    category: 'Behaviour',
    name: 'Entry Page URL',
    segment: 'entryPageUrl',
    sqlSegment: 'log_visit.visit_entry_idaction_url',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment',
    suggestedValuesApi: 'Actions.getEntryPageUrls'
  },
  {
    type: 'dimension',
    category: 'Behaviour',
    name: 'Exit Page URL',
    segment: 'exitPageUrl',
    sqlSegment: 'log_visit.visit_exit_idaction_url',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment',
    suggestedValuesApi: 'Actions.getExitPageUrls'
  },
  {
    type: 'dimension',
    category: 'Behaviour',
    name: 'Page URL',
    segment: 'pageUrl',
    sqlSegment: 'log_link_visit_action.idaction_url',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment',
    suggestedValuesApi: 'Actions.getPageUrls'
  },
  {
    type: 'metric',
    category: 'Events',
    name: 'Event value',
    segment: 'eventValue',
    sqlSegment: 'log_link_visit_action.custom_float',
    needsMostFrequentValues: true
  },
  {
    type: 'dimension',
    category: 'Events',
    name: 'Event Action',
    segment: 'eventAction',
    sqlSegment: 'log_link_visit_action.idaction_event_action',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment',
    suggestedValuesApi: 'Events.getAction'
  },
  {
    type: 'dimension',
    category: 'Events',
    name: 'Event Category',
    segment: 'eventCategory',
    sqlSegment: 'log_link_visit_action.idaction_event_category',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment',
    suggestedValuesApi: 'Events.getCategory'
  },
  {
    type: 'dimension',
    category: 'Events',
    name: 'Event Name',
    segment: 'eventName',
    sqlSegment: 'log_link_visit_action.idaction_name',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment',
    suggestedValuesApi: 'Events.getName'
  },
  {
    type: 'dimension',
    category: 'Events',
    name: 'Event URL',
    segment: 'eventUrl',
    sqlSegment: 'log_link_visit_action.idaction_url',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment',
    acceptedValues:
      'The URL must be URL encoded, for example: http%3A%2F%2Fexample.com%2Fpath%2Fpage%3Fquery'
  },
  {
    type: 'dimension',
    category: 'Acquisition',
    name: 'Channel Type',
    segment: 'referrerType',
    sqlSegment: 'log_visit.referer_type',
    needsMostFrequentValues: true,
    sqlFilterValue: 'Piwik\\Plugins\\Referrers\\getReferrerTypeFromShortName',
    acceptedValues: 'direct, search, website, campaign',
    suggestedValuesCallback: {}
  },
  {
    type: 'dimension',
    category: 'Acquisition',
    name: 'Keyword',
    segment: 'referrerKeyword',
    sqlSegment: 'log_visit.referer_keyword',
    needsMostFrequentValues: true,
    acceptedValues: 'Encoded%20Keyword, keyword'
  },
  {
    type: 'dimension',
    category: 'Acquisition',
    name: 'Referrer Name',
    segment: 'referrerName',
    sqlSegment: 'log_visit.referer_name',
    needsMostFrequentValues: true,
    acceptedValues: 'twitter.com, www.facebook.com, Bing, Google, Yahoo, CampaignName'
  },
  {
    type: 'dimension',
    category: 'Acquisition',
    name: 'Referrer URL',
    segment: 'referrerUrl',
    sqlSegment: 'log_visit.referer_url',
    needsMostFrequentValues: true,
    acceptedValues: 'http%3A%2F%2Fwww.example.org%2Freferer-page.htm'
  },
  {
    type: 'metric',
    category: 'Ecommerce',
    name: 'Order Revenue',
    segment: 'revenueOrder',
    sqlSegment: 'log_conversion.idvisit',
    needsMostFrequentValues: true,
    sqlFilter: {}
  },
  {
    type: 'metric',
    category: 'Ecommerce',
    name: 'Revenue Left In Cart',
    segment: 'revenueAbandonedCart',
    sqlSegment: 'log_conversion.idvisit',
    needsMostFrequentValues: true,
    sqlFilter: {}
  },
  {
    type: 'dimension',
    category: 'Ecommerce',
    name: 'Product Name',
    segment: 'productName',
    sqlSegment: 'log_conversion_item.idaction_name',
    needsMostFrequentValues: true,
    sqlFilter: '\\Piwik\\Tracker\\TableLogAction::getIdActionFromSegment'
  },
  {
    type: 'metric',
    category: 'Session Tags',
    name: 'Session Tag',
    segment: 'sessionTag',
    sqlSegment: 'log_hsr.session_tag',
    needsMostFrequentValues: false
  }
]

const items = [
  'menu item 1',
  'menu item 2',
  'menu item 3',
  'menu item 4',
  'menu item 5',
  'menu item 6',
  'menu item 7',
  'menu item 8'
]

export default defineComponent({
  props: ['closeSelectModal', 'data'],
  components: { ArrowSvg, CloseSvg },
  setup(props, { emit }) {
    const isDropdownOpen = ref({ default: false, action: false })
    const dropdownItems = ref(items)
    const actionsData = ref<{ condition: 'and' | 'or'; index: number }[]>([
      { condition: 'and', index: 0 }
    ])
    const selectedItem = ref({ default: '', action: '' })
    const inputValue = ref<number>()
    const secondInputError = ref(false)

    const toggleDropdown = (what: 'default' | 'action') => {
      isDropdownOpen.value[what] = !isDropdownOpen.value[what]
    }

    const openDropdown = (what: 'default' | 'action') => {
      isDropdownOpen.value[what] = true
    }

    const closeDropdown = (what: 'default' | 'action') => {
      isDropdownOpen.value[what] = false
    }

    const handleClickOutside = () => {
      closeDropdown('action')
      closeDropdown('default')
    }

    const handleBlur = (what: 'default' | 'action') => {
      setTimeout(() => closeDropdown(what), 100) // Delay to allow item selection
    }

    const selectItem = (item: string, what: 'default' | 'action') => {
      if (inputValue.value) {
        selectedItem.value[what] = `${inputValue.value}`
      } else {
        selectedItem.value[what] = item
      }
      closeDropdown(what)
    }

    const cancel = () => {
      props.closeSelectModal()
    }

    const getImagePath = (filename: string) => {
      return filename
    }

    // console.log(props.data)

    switch (props.data.name) {
      case 'Average Order Value':
      case 'Create Custom Filter':
        dropdownItems.value = [
          'Equal ( = )',
          'Less Than ( < )',
          'Greater Than ( > )',
          'Not Equal ( != )'
        ]
        break
      case 'custom':
        break
      default:
        break
    }

    const next = () => {
      if (selectedItem.value) {
        if (inputValue.value) {
          selectedItem.value.default = `${inputValue.value}`
          selectedItem.value.action = `${inputValue.value}`
        }
        emit('item-selected', selectedItem.value.default)
        props.closeSelectModal()
      }
    }

    const handleDocumentClick = (event: MouseEvent) => {
      if (!event.composedPath().includes(document.querySelector('.filter_wrapper') as Node)) {
        closeDropdown('action')
        closeDropdown('default')
      }
    }

    const filterItems = (what: 'default' | 'action') => {
      const searchText = selectedItem.value[what].toLowerCase()
      dropdownItems.value = items.filter((item) => item.toLowerCase().includes(searchText))
      openDropdown(what)
    }

    const noDropdown = (filter: string) => {
      return filter !== 'Total Pages Visited'
    }

    const numberInput = (filter: string) => {
      return filter === 'Total Pages Visited'
    }

    const validateInput = () => {
      secondInputError.value = isNaN(Number(inputValue.value))
    }

    const addCustomFilter = (condition: 'and' | 'or') => {
      const index = actionsData.value.length
      actionsData.value = [...actionsData.value, { condition, index }]
      console.log(actionsData.value)
    }

    const removeCustomFilter = (index: number) => {
      actionsData.value = actionsData.value.filter((d) => d.index !== index)
    }

    const groupDataByCategory = (data: DataItem[]): GroupedData => {
      return data.reduce((acc, item) => {
        if (!acc[item.category]) {
          acc[item.category] = []
        }
        acc[item.category].push(item)
        return acc
      }, {} as GroupedData)
    }

    const groupedData = groupDataByCategory(data)
    const actionItems = ref(groupedData)

    // console.log(groupedData)

    const labelMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Referrer URL',
        'Traffic Source': 'Referrer URL',
        'Total Pages Visited': 'Number of visits',
        'Viewed Page': 'Action URL',
        'Average Order Value': 'Condition',
        'Create Custom Filter': 'Condition'
      }
      return map[inputType]
    }

    const placeholderMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Select',
        'Traffic Source': 'Select',
        'Total Pages Visited': 'Enter value',
        'Viewed Page': 'Select',
        'Average Order Value': 'Equals',
        'Create Custom Filter': 'Equals'
      }
      return map[inputType]
    }

    const SecondLabelMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Value',
        'Traffic Source': '',
        'Total Pages Visited': '',
        'Average Order Value': 'Value',
        'Create Custom Filter': 'Value'
      }
      return map[inputType]
    }

    const SecondPlaceholderMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Select',
        'Traffic Source': '',
        'Total Pages Visited': '',
        'Average Order Value': '0.00',
        'Create Custom Filter': 'Enter value'
      }
      return map[inputType]
    }

    onMounted(() => {
      document.addEventListener('click', handleDocumentClick)
    })

    onBeforeUnmount(() => {
      document.removeEventListener('click', handleDocumentClick)
    })

    watch(inputValue, () => {
      validateInput()
    })

    return {
      isDropdownOpen,
      toggleDropdown,
      openDropdown,
      closeDropdown,
      handleClickOutside,
      handleBlur,
      selectItem,
      cancel,
      next,
      dropdownItems,
      selectedItem,
      filterItems,
      getImagePath,
      labelMap,
      placeholderMap,
      SecondLabelMap,
      SecondPlaceholderMap,
      noDropdown,
      numberInput,
      inputValue,
      validateInput,
      secondInputError,
      actionsData,
      addCustomFilter,
      removeCustomFilter,
      actionItems
    }
  }
})
</script>

<style scoped>
* {
  font-family: Montserrat;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.dropdown-enter-active,
.dropdown-leave-active {
  transition: all 0.3s ease-in-out;
  /* transform: translateY(-10px); */
}

.dropdown-enter, .dropdown-leave-to /* .dropdown-leave-active in <2.1.8 */ {
  opacity: 0;
  /* transform: translateY(-10px); */
}

input:focus,
input:visited,
input:focus-visible,
input:focus-within,
input:target {
  border-radius: var(--corner-med, 8px);
  border: 1px solid var(--Info-03-Main, #449ff4) !important;
  background: var(--Grey-White, #fff);
  box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);
}

.filter_modal {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  top: 0;
  left: 0;
  z-index: 1;
  width: 100%;
  height: 100%;
  border-radius: var(--horizontal-padding-lg, 12px);
  background: rgba(52, 64, 75, 0.1);

  p,
  ul,
  li {
    margin: 0;
    padding: 0;
  }

  .filter_wrapper {
    display: flex;
    width: 400px;
    flex-direction: column;
    align-items: flex-start;
    border-radius: var(--horizontal-padding-lg, 12px);
    border: 1px solid var(--Grey-200, #e6e7e8);
    background: var(--Grey-White, #fff);
    box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);
    /* overflow: hidden; */

    .filter_header {
      position: sticky;
      top: 0;
      background: #fff;
      z-index: 9;
      display: flex;
      padding: var(--vertical-padding-lg, 24px) var(--vertical-padding-med, 20px) 16px
        var(--vertical-padding-med, 20px);
      align-items: flex-start;
      justify-content: space-between;
      gap: 10px;
      align-self: stretch;
      border-bottom: 1px solid var(--Grey-200, #e6e7e8);

      .filter_header_left {
        display: flex;
        align-items: center;
        gap: var(--corner-med, 8px);
      }

      img {
        width: var(--vertical-padding-med, 20px);
        height: var(--vertical-padding-med, 20px);
      }

      #close_button {
        cursor: pointer;
      }

      .filter_header_text {
        flex: 1 0 0;
        color: var(--Grey-800, #34404b);
        font-size: 16px;
        font-style: normal;
        font-weight: 700;
        line-height: 24px; /* 150% */
      }
    }

    .filter_content {
      display: flex;
      padding: var(--vertical-padding-lg, 24px);
      flex-direction: column;
      align-items: flex-start;
      gap: 32px;
      align-self: stretch;
      /* max-height: 300px; */
      /* overflow-y: auto; */

      .filter_content_wrapper {
        display: flex;
        /* padding: var(--vertical-padding-lg, 24px); */
        flex-direction: column;
        align-items: flex-start;
        gap: 16px;
        align-self: stretch;
      }

      .filter_content_dropdown {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        gap: 4px;
        align-self: stretch;

        .dropdown_title {
          align-self: stretch;
          color: var(--Grey-800, #34404b);
          font-size: 14px;
          font-style: normal;
          font-weight: 500;
          line-height: 20px; /* 142.857% */
        }

        .dropdown_body_wrapper {
          position: relative;
          width: 100%;

          .absolute_placehopder {
            position: absolute;
            left: 13px;
            top: 10px;
            color: var(--Grey-600, #677078);
            font-size: 16px;
            font-style: normal;
            font-weight: 400;
            line-height: 24px; /* 150% */
          }

          .arrow_button_wrapper {
            position: absolute;
            display: flex;
            justify-content: center;
            align-items: center;
            right: 0%;
            top: 50%;
            transform: translate(-50%, -50%);
            width: 24px;
            height: 24px;
            flex-shrink: 0;
            cursor: pointer;
          }

          .arrow_button_wrapper svg {
            transition: transform 0.3s ease;
          }

          .arrow_button_wrapper svg.rotated {
            transform: rotate(180deg);
          }

          .dropdown_body {
            display: flex;
            width: calc(100%);
            padding: 8px 12px;
            justify-content: space-between;
            align-items: center;
            align-self: stretch;
            border-radius: var(--corner-med, 8px);
            border: 1px solid var(--Grey-200, #e6e7e8);
            background: var(--Grey-White, #fff);
            box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);

            color: var(--Grey-800, #34404b);
            font-size: 16px;
            font-style: normal;
            font-weight: 400;
            line-height: 24px; /* 150% */

            &.second_one {
              text-align: right;
            }
          }

          .dropdown_body::placeholder {
            color: var(--Grey-800, #999fa5);
            font-size: 16px;
            font-style: normal;
            font-weight: 400;
            line-height: 24px; /* 150% */
          }

          .dropdown_menu_wrapper {
            position: absolute;
            width: 100%;
            border-radius: var(--corner-med, 10px);
            background: var(--Grey-White, #fff);
            border: 1px solid var(--Grey-200, #e6e7e8);
            box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);
            list-style: none;
            padding: 0;
            margin-top: 4px;
            max-height: 300px;
            overflow-y: auto;
            z-index: 10;
            /* transition: all 3s ease-in-out; */

            .dropdown_menu_item {
              display: flex;
              padding: var(--corner-med, 8px) var(--horizontal-padding-lg, 12px);
              align-items: flex-start;
              align-self: stretch;
              cursor: pointer;

              color: var(--Grey-800, #34404b);
              font-size: 14px;
              font-style: normal;
              font-weight: 400;
              line-height: 20px; /* 142.857% */
              &:not(:last-child) {
                border-bottom: 1px solid var(--Grey-200, #e6e7e8);
              }
              &.activeClass {
                color: var(--Grey-800, #fff) !important;
                background: #03c191 !important;
              }
            }

            .dropdown_menu_item:hover {
              background: #e6e7e8;
            }
          }
        }
      }
    }
  }
}

.filter_footer {
  position: sticky;
  bottom: 0;
  background: #fff;
  z-index: 9;
  display: flex;
  padding: var(--horizontal-padding-lg, 12px) var(--vertical-padding-med, 20px);
  justify-content: flex-end;
  align-items: center;
  align-self: stretch;
  border-radius: 0px 0px var(--horizontal-padding-lg, 12px) var(--horizontal-padding-lg, 12px);
  background: var(--Grey-White, #fff);
  border-top: 1px solid #e6e7e8;

  .footer_buttons {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex: 1 0 0;

    .footer_button {
      display: flex;
      padding: var(--Padding-Horizontal-padding, 6px) var(--Padding-Vertical-padding, 16px);
      align-items: flex-start;
      gap: 10px;
      border-radius: var(--Padding-Corner, 6px);
      background: var(--Grey-200, #e6e7e8);
      box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);
      cursor: pointer;

      &.disabled_me {
        opacity: 0.5;
        cursor: not-allowed;
      }

      .footer_button_text {
        color: var(--Grey-600, #677078);
        font-size: 16px;
        font-style: normal;
        font-weight: 600;
        line-height: 26px; /* 200% */
      }
    }

    .footer_button.primary_button .footer_button_text {
      color: var(--Grey-White, #fff);
    }

    .footer_button.primary_button {
      background: var(--Primary-03-Main, #00936f);
    }
  }

  .button-group {
    display: flex;
    border-radius: 8px;
    overflow: hidden;
    border: 1px solid #e6e7e8;

    .footer_button {
      border-radius: 0;
      background: transparent;

      .footer_button_text {
        color: #34404b;
        font-size: 14px;
      }

      &:last-child {
        /* border-left: 1px solid #34404b; */
      }
    }
  }
}
.remove_action_wrapper {
  display: flex;
  width: 100%;
  /* margin-top: 8px; */
  align-items: center;
  justify-content: space-between;
}

.condition_indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  margin-top: 8px;

  p {
    color: #00936f;
    font-weight: 900;
  }
}

.error_message {
  color: var(--Error-04-Dark, #b71e2d);
  font-size: 14px;
  font-style: normal;
  font-weight: 500;
  line-height: 20px; /* 142.857% */
  margin-top: 8px !important;
}
</style>
