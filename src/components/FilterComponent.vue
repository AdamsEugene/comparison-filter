<template>
  <div class="heat_custom_filter_absolute">
    <div class="heat_custom_filter">
      <filter-modal
        v-if="showSelectModal"
        :closeSelectModal="closeSelectModal"
        :data="modalData"
        @item-selected="onItemSelected"
      />
      <div class="heat_custom_filter_body">
        <header class="heat_custom_filter_header">
          <p class="heat_custom_filter_header_text">
            {{ readyToCompare() ? 'Compare' : 'All Filters' }}
          </p>
          <template
            v-for="(filter, index) in selectIndicators.pendingList.slice(0, 2)"
            :key="filter.name"
          >
            <div class="right_button new_color insideCompare">
              <p class="right_button_text">{{ filter.name }}</p>
            </div>
            <p v-if="index < 1" class="heat_custom_filter_header_text">to</p>
            <div
              v-if="index === selectIndicators.pendingList.length - 1 && index < 1"
              class="right_button awaiting"
            >
              <p class="right_button_text">Select a filter</p>
            </div>
          </template>
        </header>
        <div class="filter_body">
          <div class="filter_body_column">
            <header class="filter_body_column_head">
              <p class="filter_body_column_head_text">Session Data</p>
            </header>
            <ul class="filter_body_filters">
              <li
                v-for="data in sessionData"
                :key="data.name"
                :data-idsegment="data.idsegment"
                :data-definition="data.definition"
                :data-title="data.title"
                @click="setActive(data)"
                class="filter_body_filter"
                :class="{
                  pendingClass: filterExist(data.name),
                  activeClass: addActiveClass(data.name),
                  disabled_me: disableButton(data.name)
                }"
              >
                <img class="filter_image" :src="getImagePath(data.iconSrc)" alt="" />
                <p class="filter_body_filter_text">{{ data.name }}</p>
              </li>
            </ul>
          </div>
          <div class="filter_body_column">
            <header class="filter_body_column_head">
              <p class="filter_body_column_head_text">Ecommerce Data</p>
            </header>
            <ul class="filter_body_filters">
              <li
                v-for="data in eCommerceData"
                :key="data.name"
                :data-idsegment="data.idsegment"
                :data-definition="data.definition"
                :data-title="data.title"
                @click="setActive(data)"
                class="filter_body_filter"
                :class="{
                  pendingClass: filterExist(data.name),
                  activeClass: addActiveClass(data.name),
                  disabled_me: disableButton(data.name)
                }"
              >
                <img class="filter_image" :src="getImagePath(data.iconSrc)" alt="" />
                <p class="filter_body_filter_text">{{ data.name }}</p>
              </li>
            </ul>
          </div>
          <div class="filter_body_column">
            <header class="filter_body_column_head">
              <p class="filter_body_column_head_text">Custom Filters</p>
            </header>
            <ul class="filter_body_filters">
              <div v-if="customData.length > 0">
                <li
                  v-for="data in customData"
                  :key="data.name"
                  @click="setActive(data)"
                  class="filter_body_filter"
                  :class="{
                    pendingClass: filterExist(data.name),
                    activeClass: addActiveClass(data.name)
                  }"
                >
                  <p class="filter_body_filter_text">{{ data }}</p>
                </li>
              </div>
              <div class="add_custom_filter" @click="onAddCustomFilter()">
                <p class="add_custom_filter_text">Add Custom</p>
              </div>
            </ul>
          </div>
        </div>
      </div>
      <div class="heat_custom_filter_footer">
        <div class="left_button">
          <p class="left_button_text" @click="resetAllFilters">Reset</p>
        </div>
        <div class="right_buttons">
          <div
            v-if="selectIndicators.pendingList.length < 2"
            class="right_button"
            :class="{ disabled_me: !selectIndicators.active }"
            @click="onCompareWith()"
          >
            <p class="right_button_text">Compare to...</p>
          </div>
          <div class="right_button new_color" :class="{ disabled_me: !disableCompareButton() }">
            <p class="right_button_text" @click="onCompare">
              {{ !canApply() ? 'Apply' : 'Compare' }}
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import flightLand from '@/assets/images/flight_land.svg'
import trackChanges from '@/assets/images/track_changes.svg'
import numbers from '@/assets/images/numbers.svg'
import task from '@/assets/images/task.svg'
import group from '@/assets/images/group.svg'
import groupAdd from '@/assets/images/group_add.svg'
import sentimentExtremelyDissatisfied from '@/assets/images/sentiment_extremely_dissatisfied.svg'
import addShoppingCart from '@/assets/images/add_shopping_cart.svg'
import payments from '@/assets/images/payments.svg'

import FilterModal from './FilterModal.vue'

interface SessionDataItem {
  definition: string
  iconSrc: string // Assuming iconSrc is a URL string
  idsegment: number
  name: string
  title: string
  notDone?: boolean // Optional property
  isDefinitionValueSet?: boolean // Optional property
}

interface ECommerceDataItem {
  definition: string
  iconSrc: string // Assuming iconSrc is a URL string or a string identifier for an icon
  idsegment: number
  name: string
  title: string
  isDefinitionValueSet?: boolean // Optional property
  showSign?: boolean // Optional property
}

export type CombinedFilter = SessionDataItem | ECommerceDataItem
type SelectIndicators = { active?: CombinedFilter; pendingList: CombinedFilter[] }
export type ReturnData = { definition: string; name: string }

type Data = {
  sessionData: SessionDataItem[]
  eCommerceData: ECommerceDataItem[]
  customData: CombinedFilter[]
  selectIndicators: SelectIndicators
  defaultSelections?: ReturnData[]
  showSelectModal: boolean
  modalData?: CombinedFilter
}

const sessionData: SessionDataItem[] = [
  {
    definition: 'entryPageUrl==',
    iconSrc: flightLand,
    idsegment: 0,
    name: 'Entry Page',
    title: 'QuickFilter-EntryPage'
  },
  {
    definition: 'referrerUrl==',
    iconSrc: trackChanges,
    idsegment: 0,
    name: 'Traffic Source',
    title: 'QuickFilter-TrafficSource',
    notDone: true
  },
  {
    definition: 'visitCount==',
    iconSrc: numbers,
    idsegment: 0,
    name: 'Total Pages Visited',
    title: 'QuickFilter-TotalPagesVisited'
  },
  {
    definition: 'actionUrl==',
    iconSrc: task,
    idsegment: 0,
    name: 'Viewed Page',
    title: 'QuickFilter-ViewedPage'
  },
  {
    definition: 'visitorType==returning',
    iconSrc: group,
    idsegment: 0,
    name: 'Returning Users',
    title: 'QuickFilter-ReturningUsers',
    isDefinitionValueSet: true
  },
  {
    definition: 'visitorType==new',
    iconSrc: groupAdd,
    idsegment: 0,
    name: 'New Users',
    title: 'QuickFilter-NewUsers',
    isDefinitionValueSet: true
  },
  {
    definition: 'heatmapType==rage',
    iconSrc: sentimentExtremelyDissatisfied,
    idsegment: 0,
    name: 'Rage Clicks',
    title: 'rage click events',
    isDefinitionValueSet: true
  }
]

const eCommerceData: ECommerceDataItem[] = [
  {
    definition: 'revenueOrder>1',
    iconSrc: addShoppingCart,
    idsegment: 0,
    name: 'Purchasers',
    title: 'QuickFilter-Purchasers',
    isDefinitionValueSet: true
  },
  {
    definition: 'revenueOrder==',
    iconSrc: payments,
    idsegment: 0,
    name: 'Average Order Value',
    title: 'QuickFilter-OrderValue',
    showSign: true
  }
]

const customData: CombinedFilter[] = []

const selectIndicators: SelectIndicators = {
  active: undefined,
  pendingList: []
}

export default defineComponent({
  props: ['onToggleShowFilterMenu', 'defaultValues'],
  components: { FilterModal },
  data(): Data {
    return {
      sessionData,
      eCommerceData,
      customData,
      selectIndicators,
      defaultSelections: undefined,
      showSelectModal: false,
      modalData: undefined
    }
  },
  methods: {
    getImagePath(filename: string) {
      return filename
    },
    canApply() {
      return this.selectIndicators.pendingList.length === 2
    },
    setActive(filter: CombinedFilter) {
      this.defaultSelections = []
      if (!filter.isDefinitionValueSet && !this.filterExist(filter.name)) {
        this.showSelectModal = true
        this.modalData = filter
      } else {
        this.selectIndicators.active = filter
        if (this.selectIndicators.pendingList.length === 1) {
          this.onCompareWith()
        }
      }
    },
    addActiveClass(filter: string) {
      return (
        this.defaultSelections?.map((d) => d.name)?.includes(filter) ||
        selectIndicators.active?.name === filter
      )
    },
    onAddCustomFilter() {
      this.modalData = { name: 'Create Custom Filter', definition: '', iconSrc: '', idsegment: 0, title: '',showSign:true }
      this.showSelectModal = true
    },
    onCompareWith(fromModal?: Boolean) {
      if (this.selectIndicators.active) {
        const filter = fromModal ? this.modalData! : this.selectIndicators.active!
        this.selectIndicators.pendingList = [...this.selectIndicators.pendingList, filter]
        this.selectIndicators.active = undefined
      }
    },
    resetAllFilters() {
      this.selectIndicators.pendingList = []
      this.selectIndicators.active = undefined
      this.selectIndicators.pendingList = []
      this.defaultSelections = []
    },
    disableButton(button: string) {
      const index = selectIndicators.pendingList.map((f) => f.name).indexOf(button)
      return index >= 0
    },
    readyToCompare() {
      return selectIndicators.pendingList.length > 0
    },
    filterExist(filter: string) {
      return selectIndicators.pendingList.map((f) => f.name).includes(filter)
    },
    closeSelectModal() {
      this.showSelectModal = false
    },
    disableCompareButton() {
      return (
        this.selectIndicators.pendingList.length > 0 || Boolean(this.selectIndicators.active?.name)
      )
    },
    onItemSelected(item: string) {
      if (this.modalData) {
        const definition = this.modalData.definition
        this.modalData = { ...this.modalData, definition: `${definition}${item}` }
        this.selectIndicators.active = this.modalData
        if (this.selectIndicators.pendingList.length === 1) {
          this.onCompareWith(true)
        } else {
          // this.selectIndicators.pendingList = [...this.selectIndicators.pendingList, this.modalData]
        }
      }
    },
    onCompare() {
      if (this.selectIndicators.pendingList.length === 0 && this.selectIndicators.active?.name) {
        this.selectIndicators.pendingList = [this.selectIndicators.active]
      }
      const returnData = this.selectIndicators.pendingList.map((d) => ({
        definition: d.definition,
        name: d.name
      }))
      this.$emit('filter-values', returnData)
      this.resetAllFilters()
      this.onToggleShowFilterMenu()
    }
  },
  computed: {},
  watch: {
    defaultValues: {
      immediate: true,
      handler(newVal: ReturnData[]) {
        this.defaultSelections = newVal
      }
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

.heat_custom_filter_absolute {
  position: absolute;
  top: 58px;
  left: 0;
  z-index: 9;
}

.heat_custom_filter {
  position: relative;
  display: flex;
  width: 612px;
  flex-direction: column;
  align-items: flex-start;
  border-radius: var(--horizontal-padding-lg, 12px);
  border: 1px solid var(--Grey-200, #e6e7e8);
  background: var(--Grey-White, #fff);
  box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);

  p,
  ul,
  li {
    margin: 0;
    padding: 0;
  }

  .heat_custom_filter_body {
    display: flex;
    padding: var(--vertical-padding-lg, 24px) var(--vertical-padding-med, 20px) 16px
      var(--vertical-padding-med, 20px);
    flex-direction: column;
    align-items: flex-start;
    gap: var(--vertical-padding-lg, 24px);
    align-self: stretch;

    .heat_custom_filter_header {
      display: flex;
      padding: 0px 6px;
      align-items: center;
      gap: var(--corner-med, 8px);
      align-self: stretch;
      border-radius: 4px;

      .heat_custom_filter_header_text {
        color: var(--Grey-800, #34404b);
        font-size: 16px;
        font-style: normal;
        font-weight: 700;
        line-height: 24px; /* 150% */
        padding: 8px 0px;
      }
    }

    .filter_body {
      display: flex;
      align-items: flex-start;
      gap: var(--vertical-padding-lg, 24px);
      align-self: stretch;

      .filter_body_column {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        gap: var(--corner-med, 8px);
        flex: 1 0 0;

        .filter_body_column_head {
          display: flex;
          padding: 0px 6px;
          align-items: center;
          gap: var(--corner-med, 8px);
          align-self: stretch;
          border-radius: 4px;

          .filter_body_column_head_text {
            flex: 1 0 0;
            color: var(--Primary-04-Dark, #016f52);
            font-size: 14px;
            font-style: normal;
            font-weight: 600;
            line-height: 18px; /* 128.571% */
          }
        }

        .filter_body_filters {
          display: flex;
          flex-direction: column;
          align-items: flex-start;
          align-self: stretch;

          .filter_body_filter {
            display: flex;
            padding: 10px 6px;
            align-items: center;
            gap: 8px;
            align-self: stretch;
            border-radius: 4px;
            list-style: none;
            cursor: pointer;
            transition: all 0.3s ease-in-out;

            .filter_image {
              transition: all 0.3s ease-in-out;
            }

            &:hover {
              background: #e6e7e8;
              /* .filter_body_filter_text {
                color: #fff;
              } */

              /* .filter_image {
                filter: invert(100%);
              } */
            }

            .filter_body_filter_text {
              flex: 1 0 0;
              color: var(--Grey-800, #34404b);
              font-size: 12px;
              font-style: normal;
              font-weight: 500;
              line-height: 12px; /* 100% */
              transition: all 0.3s ease-in-out;
            }

            &.pendingClass {
              background-color: #000000bc;
              .filter_body_filter_text {
                color: #fff;
              }
              .filter_image {
                filter: invert(100%);
              }
            }
            &.activeClass {
              background: #03c191 !important;
              .filter_body_filter_text {
                color: #fff !important;
              }
              .filter_image {
                filter: invert(100%);
              }
            }

            &.disabled_me {
              background-color: transparent;
              .filter_body_filter_text {
                color: var(--Grey-500, #999fa5);
              }
            }
          }

          .add_custom_filter {
            display: flex;
            padding: 7px 6px;
            align-items: center;
            gap: 8px;
            align-self: stretch;
            border-radius: 4px;
            cursor: pointer;
            transition: all 0.3s ease-in-out;

            &:hover {
              background: #e6e7e8;
              /* .add_custom_filter_text {
                color: #fff;
              } */
            }

            .add_custom_filter_text {
              color: var(--Grey-800, #34404b);
              font-size: 13px;
              font-style: normal;
              font-weight: 600;
              line-height: 18px; /* 138.462% */
              transition: all 0.3s ease-in-out;
            }
          }
        }
      }
    }
  }

  .heat_custom_filter_footer {
    display: flex;
    padding: 12px 20px;
    justify-content: space-between;
    align-items: center;
    align-self: stretch;
    border-radius: 0px 0px 12px 12px;
    border: 1px solid var(--Grey-200, #e6e7e8);
    background: var(--Grey-White, #fff);

    .left_button {
      display: flex;
      padding: var(--Padding-Horizontal-padding, 6px) var(--Padding-Vertical-padding, 16px);
      align-items: flex-start;
      gap: 10px;
      border-radius: var(--Padding-Corner, 6px);
      background: var(--Grey-100, #f6f6f6);
      box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);
      cursor: pointer;

      .left_button_text {
        color: var(--Grey-600, #677078);
        font-size: 16px;
        font-style: normal;
        font-weight: 600;
        line-height: 32px; /* 200% */
      }
    }

    .right_buttons {
      display: flex;
      align-items: center;
      gap: var(--horizontal-padding-lg, 12px);
    }
  }

  .right_button {
    display: flex;
    padding: var(--Padding-Horizontal-padding, 6px) var(--Padding-Vertical-padding, 16px);
    align-items: flex-start;
    gap: 10px;
    border-radius: var(--Padding-Corner, 6px);
    border: 1.5px solid var(--Primary-03-Main, #00936f);
    box-shadow: 0px 1px 2px 0px rgba(26, 40, 53, 0.09);
    cursor: pointer;

    &.new_color {
      background: var(--Primary-03-Main, #00936f);
      .right_button_text {
        color: var(--Grey-White, #fff);
      }
    }

    &.awaiting {
      background: var(--Grey-100, #f6f6f6);
      border: 1.5px solid var(--Primary-03-Main, #f6f6f6);
      padding: 2px 10px;
      .right_button_text {
        color: var(--Grey-White, #b3b7bc);
        font-size: 12px;
      }
    }

    &.insideCompare {
      padding: 2px 10px;
      .right_button_text {
        font-size: 12px;
      }
    }

    .right_button_text {
      color: var(--Primary-03-Main, #00936f);
      font-size: 16px;
      font-style: normal;
      font-weight: 600;
      line-height: 32px; /* 200% */
    }

    &.disabled_me {
      opacity: 0.5;
      cursor: not-allowed;
    }
  }

  @keyframes radiating {
    0% {
      transform: scale(0.8);
    }
    50% {
      transform: scale(1);
    }
    100% {
      transform: scale(0.8);
    }
  }

  @keyframes glowing-green {
    0% {
      box-shadow:
        0 0 10px #00ff00,
        0 0 20px #00ff00,
        0 0 30px #00ff00;
    }
    50% {
      box-shadow:
        0 0 20px #00ff00,
        0 0 30px #00ff00,
        0 0 40px #00ff00;
    }
    100% {
      box-shadow:
        0 0 10px #00ff00,
        0 0 20px #00ff00,
        0 0 30px #00ff00;
    }
  }
}
</style>
