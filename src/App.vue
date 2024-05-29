<template>
  <div class="main_filter_wrapper">
    <filter-button :onShowFilterMenu="onToggleShowFilterMenu" :filteredValues="filteredValues" />
    <filter-component
      v-show="showFilterMenu"
      :onToggleShowFilterMenu="onToggleShowFilterMenu"
      :defaultValues="filteredValues"
      @filter-values="onFilterValuesChange"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import FilterComponent, { type ReturnData } from './components/FilterComponent.vue'
import FilterButton from './components/FilterButton.vue'

export default defineComponent({
  components: { FilterComponent, FilterButton },
  data(): { filteredValues?: ReturnData; showFilterMenu: boolean } {
    return {
      filteredValues: undefined,
      showFilterMenu: false
    }
  },
  methods: {
    onFilterValuesChange(values: ReturnData) {
      console.log(values)
      this.filteredValues = values
      this.$emit('on-filter-values-change', values)
    },
    onToggleShowFilterMenu() {
      this.showFilterMenu = !this.showFilterMenu
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
.main_filter_wrapper {
  position: relative;
}
</style>
