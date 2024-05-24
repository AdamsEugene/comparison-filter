<!-- <template>
  <ul class="filter_body_filters">
    <li
      v-for="item in data"
      :key="item.name"
      :data-idsegment="item.idsegment"
      :data-definition="item.definition"
      :data-title="item.title"
      @click="setActive(item.name, item.isDefinitionValueSet)"
      class="filter_body_filter"
      :class="{
        pendingClass: filterExist(item.name),
        activeClass: selectIndicators.active.includes(item.name),
        disabled_me: disableButton(item.name)
      }"
    >
      <img class="filter_image" :src="getImagePath(item.iconSrc)" alt="" />
      <p class="filter_body_filter_text">{{ item.name }}</p>
    </li>
  </ul>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  props: ['data'],
  components: {},
  data() {
    return {
      // Define your reactive data properties here
    }
  },
  methods: {
    getImagePath(filename: string) {
      return filename
    },
    setActive(filter: string, noModal?: boolean) {
      this.selectIndicators.active = filter
      if (this.selectIndicators.pendingList.length === 1) {
        this.onCompareWith()
      }
      if (!noModal) {
        this.showSelectModal = true
      }
    },
    filterExist(filter: string) {
      return selectIndicators.pendingList.map((f) => f.filter).includes(filter)
    },
    disableButton(button: string) {
      const index = selectIndicators.pendingList.map((f) => f.filter).indexOf(button)
      return index >= 0
    }
  }
})
</script>

<style scoped></style> -->
