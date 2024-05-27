<template>
  <div class="filter_modal" @click="handleClickOutside">
    <div class="filter_wrapper" @click.stop>
      <header class="filter_header">
        <div class="filter_header_left">
          <img class="filter_image" :src="getImagePath(data.iconSrc)" alt="" />
          <p class="filter_header_text">{{ data.name }}</p>
        </div>
        <svg
          id="close_button"
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 20 20"
          fill="none"
          @click="closeSelectModal()"
        >
          <path
            d="M15 5L5 15"
            stroke="#677078"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
          <path
            d="M5 5L15 15"
            stroke="#677078"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </header>
      <div class="filter_content">
        <div class="filter_content_wrapper">
          <div class="filter_content_dropdown">
            <p class="dropdown_title">{{ labelMap(data?.name) }}</p>
            <div class="dropdown_body_wrapper">
              <div
                v-if="noDropdown(data.name)"
                class="arrow_button_wrapper"
                @click="toggleDropdown"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="15"
                  height="8"
                  viewBox="0 0 15 8"
                  fill="none"
                  :class="{ rotated: isDropdownOpen }"
                >
                  <path
                    d="M1.89062 1L7.7519 6.82487C7.8494 6.92177 8.00685 6.92177 8.10435 6.82487L13.9656 1"
                    stroke="#34404B"
                    stroke-width="2"
                    stroke-linecap="round"
                  />
                </svg>
              </div>
              <input
                class="dropdown_body"
                :type="numberInput(data.name) ? 'number' : 'text'"
                :placeholder="placeholderMap(data?.name)"
                @blur="handleBlur"
                @focus="openDropdown"
                @input="filterItems"
                v-model="selectedItem"
              />
              <ul v-if="isDropdownOpen && noDropdown(data.name)" class="dropdown_menu_wrapper">
                <li
                  v-for="item in dropdownItems"
                  :key="item"
                  class="dropdown_menu_item"
                  :class="{ activeClass: item === selectedItem }"
                  @click="selectItem(item)"
                >
                  {{ item }}
                </li>
              </ul>
            </div>
          </div>
          <div v-if="data.showSign" class="filter_content_dropdown">
            <p class="dropdown_title">{{ SecondLabelMap(data?.name) }}</p>
            <div class="dropdown_body_wrapper">
              <!-- <div class="arrow_button_wrapper" @click="toggleDropdown">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="15"
                  height="8"
                  viewBox="0 0 15 8"
                  fill="none"
                  :class="{ rotated: isDropdownOpen }"
                >
                  <path
                    d="M1.89062 1L7.7519 6.82487C7.8494 6.92177 8.00685 6.92177 8.10435 6.82487L13.9656 1"
                    stroke="#34404B"
                    stroke-width="2"
                    stroke-linecap="round"
                  />
                </svg>
              </div> -->
              <input
                class="dropdown_body second_one"
                :type="numberInput(data.name) ? 'number' : 'text'"
                :placeholder="SecondPlaceholderMap(data?.name)"
              />
              <div v-if="data.name === 'Average Order Value'" class="absolute_placehopder">$</div>
              <!-- <ul v-if="isDropdownOpen" class="dropdown_menu_wrapper">
                <li
                  v-for="item in dropdownItems"
                  :key="item"
                  class="dropdown_menu_item"
                  @click="selectItem(item)"
                >
                  {{ item }}
                </li>
              </ul> -->
            </div>
          </div>
        </div>
      </div>
      <footer class="filter_footer">
        <div class="footer_buttons">
          <div class="footer_button" @click="cancel">
            <p class="footer_button_text">Cancel</p>
          </div>
          <div
            class="footer_button primary_button"
            :class="{ disabled_me: Boolean(!selectedItem) }"
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
import { defineComponent, ref, onMounted, onBeforeUnmount } from 'vue'

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
  setup(props, { emit }) {
    const isDropdownOpen = ref(false)
    const dropdownItems = ref(items)
    const selectedItem = ref('')

    const toggleDropdown = () => {
      isDropdownOpen.value = !isDropdownOpen.value
    }

    const openDropdown = () => {
      isDropdownOpen.value = true
    }

    const closeDropdown = () => {
      isDropdownOpen.value = false
    }

    const handleClickOutside = () => {
      closeDropdown()
    }

    const handleBlur = () => {
      setTimeout(closeDropdown, 100) // Delay to allow item selection
    }

    const selectItem = (item: string) => {
      selectedItem.value = item
      closeDropdown()
    }

    const cancel = () => {
      props.closeSelectModal()
    }

    const getImagePath = (filename: string) => {
      return filename
    }

    const next = () => {
      if (selectedItem.value) {
        emit('item-selected', selectedItem.value)
        props.closeSelectModal()
      }
    }

    const handleDocumentClick = (event: MouseEvent) => {
      if (!event.composedPath().includes(document.querySelector('.filter_wrapper') as Node)) {
        closeDropdown()
      }
    }

    const filterItems = () => {
      const searchText = selectedItem.value.toLowerCase()
      dropdownItems.value = items.filter((item) => item.toLowerCase().includes(searchText))
      openDropdown()
    }

    const noDropdown = (filter: string) => {
      return filter !== 'Total Pages Visited'
    }

    const numberInput = (filter: string) => {
      return filter === 'Total Pages Visited'
    }

    const labelMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Referrer URL',
        'Traffic Source': 'Referrer URL',
        'Total Pages Visited': 'Number of visits',
        'Viewed Page': 'Action URL',
        'Average Order Value': 'Condition'
      }
      return map[inputType]
    }

    const placeholderMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Select',
        'Traffic Source': 'Select',
        'Total Pages Visited': 'Enter value',
        'Viewed Page': 'Select',
        'Average Order Value': 'Equals'
      }
      return map[inputType]
    }

    const SecondLabelMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Value',
        'Traffic Source': '',
        'Total Pages Visited': '',
        'Average Order Value': 'Value'
      }
      return map[inputType]
    }

    const SecondPlaceholderMap = (inputType: string) => {
      const map: { [x: string]: string } = {
        'Entry Page': 'Select',
        'Traffic Source': '',
        'Total Pages Visited': '',
        'Average Order Value': '0.00'
      }
      return map[inputType]
    }

    onMounted(() => {
      document.addEventListener('click', handleDocumentClick)
    })

    onBeforeUnmount(() => {
      document.removeEventListener('click', handleDocumentClick)
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
      numberInput
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

    .filter_header {
      display: flex;
      padding: var(--vertical-padding-lg, 24px) var(--vertical-padding-med, 20px) 16px
        var(--vertical-padding-med, 20px);
      align-items: flex-start;
      justify-content: space-between;
      gap: 10px;
      align-self: stretch;
      border: 1px solid var(--Grey-200, #e6e7e8);

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
            top: 13px;
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
            padding: var(--horizontal-padding-lg, 12px);
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
            z-index: 2;

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
  display: flex;
  padding: var(--horizontal-padding-lg, 12px) var(--vertical-padding-med, 20px);
  justify-content: flex-end;
  align-items: center;
  align-self: stretch;
  border-radius: 0px 0px var(--horizontal-padding-lg, 12px) var(--horizontal-padding-lg, 12px);
  background: var(--Grey-White, #fff);

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
        line-height: 32px; /* 200% */
      }
    }

    .footer_button.primary_button .footer_button_text {
      color: var(--Grey-White, #fff);
    }

    .footer_button.primary_button {
      background: var(--Primary-03-Main, #00936f);
    }
  }
}
</style>
