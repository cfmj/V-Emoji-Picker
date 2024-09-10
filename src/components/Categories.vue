<template>
  <div id="Categories">
    <div
      v-for="(category, index) in categories"
      :class="['category', { active: category.name === current }]"
      :key="index"
      @click="onSelect(category)"
    >
      <CategoryItem :label="category.label" :icon="category.icon" />
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Emit } from "vue-property-decorator";
import { Category } from "@/models/Category";

import CategoryItem from "./CategoryItem.vue";

@Component({
  components: {
    CategoryItem
  }
})
export default class Categories extends Vue {
  @Prop({}) categories!: Category[];
  @Prop({}) current!: string;

  @Emit("select")
  onSelect(category: Category) {
    return category;
  }
}
</script>

<style lang="scss" scoped>
#Categories {
  box-sizing: border-box;
  display: flex;
  width: 100%;
  align-items: center;
  border-bottom: 1px solid var(--ep-color-border);
  background: var(--ep-color-bg);
  overflow-x: auto;
  height: 56px;
  padding: 16px 20px 0;
}

.category {
  box-sizing: border-box;
  height: 100%;
  margin-right: 20px;
  padding-bottom: 16px;
  text-align: center;
  cursor: pointer;

  &.active {
    border-bottom: 2px solid var(--ep-color-active);
    //   filter: saturate(3);
    padding-bottom: 14px;
  }

  & > img {
    width: 24px;
    height: 24px;
  }

  &:hover {
    filter: saturate(3);
  }
}

.category:last-of-type {
  margin-right: 0;
}
</style>
