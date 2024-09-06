<template>
  <div id="EmojiPicker" :class="['emoji-picker', { dark }]">
    <Categories
      v-if="showCategories"
      :categories="categoriesFiltered"
      :current="currentCategory"
      @select="changeCategory"
    />
    <InputSearch v-if="showSearch" @update="onSearch" />
    <EmojiList
      :data="mapEmojis"
      :category="currentCategory"
      :filter="filterEmoji"
      :emojiWithBorder="emojiWithBorder"
      :emojiSize="emojiSize"
      :stickerSize="stickerSize"
      :emojisByRow="emojisByRow"
      :stickersByRow="stickersByRow"
      :continuousList="continuousList"
      :hasSearch="showSearch"
      @select="onSelectEmoji"
    />
  </div>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue, Watch } from "vue-property-decorator";

import { Emoji, IEmoji } from "./models/Emoji";
import { ICategory } from "./models/Category";
import { MapEmojis } from "./models/MapEmojis";

import { emojisDefault } from "./utils/emojis";
import { categoriesDefault } from "./utils/categories";

import Categories from "./components/Categories.vue";
import EmojiList from "./components/EmojiList.vue";
import InputSearch from "./components/InputSearch.vue";

import locale from "./locale";

@Component({
  components: {
    Categories,
    EmojiList,
    InputSearch
  }
})
export default class VEmojiPicker extends Vue {
  @Prop({ default: () => emojisDefault }) customEmojis!: IEmoji[];
  @Prop({ default: () => categoriesDefault }) customCategories!: ICategory[];
  @Prop({ default: 15 }) limitFrequently!: number;
  @Prop({ default: 7 }) emojisByRow!: number;
  @Prop({ default: 4 }) stickersByRow!: number;
  @Prop({ default: false }) continuousList!: boolean;
  @Prop({ default: 32 }) emojiSize!: number;
  @Prop({ default: 64 }) stickerSize!: number;
  @Prop({ default: true }) emojiWithBorder!: boolean;
  @Prop({ default: false }) showSearch!: boolean;
  @Prop({ default: true }) showCategories!: boolean;
  @Prop({ default: false }) dark!: boolean;
  @Prop({ default: "Peoples" }) initialCategory!: string;
  @Prop({ default: () => [] as ICategory[] }) exceptCategories!: ICategory[];
  @Prop({ default: () => [] as Emoji[] }) exceptEmojis!: IEmoji[];
  @Prop({}) i18n!: Object;

  mapEmojis: MapEmojis = {};

  currentCategory = this.initialCategory;
  filterEmoji = "";

  created() {
    const categoriesNames = this.customCategories.map(c => c.name);
    if (!categoriesNames.includes(this.initialCategory)) {
      this.initialCategory = categoriesNames[0];
    }

    // Create map
    this.mapperEmojisCategory(this.customEmojis);
    this.restoreFrequentlyEmojis();

    // Configure i18n
    if (this.i18n) {
      locale.i18n(this.i18n);
    }
  }

  beforeDestroy() {
    this.mapEmojis = {};
  }

  async onSearch(term: string) {
    this.filterEmoji = term;
  }

  async changeCategory(category: ICategory) {
    const hasEmojis = this.mapEmojis[category.name].length;
    this.currentCategory = category.name;

    if (hasEmojis) {
      await this.onChangeCategory(category);
    }
  }

  async updateFrequently(emoji: IEmoji) {
    if (emoji.data.startsWith("http")) {
      return;
    }
    const oldEmojis = this.mapEmojis["Frequently"];
    const emojis = [...new Set([emoji, ...oldEmojis])];

    this.mapEmojis["Frequently"] = emojis.slice(0, this.limitFrequently);

    await this.saveFrequentlyEmojis(emojis);
  }

  async mapperEmojisCategory(emojis: IEmoji[]) {
    this.$set(this.mapEmojis, "Frequently", []);

    emojis
      .filter(emoji => !this.exceptEmojis.includes(emoji))
      .forEach(emoji => {
        const _category = emoji.category;

        if (!this.mapEmojis[_category]) {
          this.$set(this.mapEmojis, _category, []);
        }

        this.mapEmojis[_category].push(emoji);
      });
  }

  async restoreFrequentlyEmojis() {
    const jsonMapIndexEmojis = localStorage.getItem("frequentlyEmojis");

    const mapIndexEmojis = JSON.parse(jsonMapIndexEmojis!!) || [];
    this.mapEmojis["Frequently"] = mapIndexEmojis.map(
      index => this.customEmojis[index]
    );
  }

  async saveFrequentlyEmojis(emojis: IEmoji[]) {
    const mapIndexEmojis = emojis.map(emoji => {
      return this.customEmojis.indexOf(emoji);
    });

    localStorage.setItem("frequentlyEmojis", JSON.stringify(mapIndexEmojis));
  }

  get categoriesFiltered() {
    //  !this.exceptCategories.includes(category)
    return this.customCategories.filter(category =>
      category.name.startsWith("Peoples")
    );
  }

  @Emit("select")
  async onSelectEmoji(emoji: IEmoji) {
    await this.updateFrequently(emoji);

    return emoji;
  }

  @Emit("changeCategory")
  async onChangeCategory(category: ICategory) {
    return category;
  }

  @Watch("customEmojis")
  onChangeCustomEmojis(newEmojis: IEmoji[]) {
    if (newEmojis && newEmojis.length) {
      this.mapEmojis = {};
      this.mapperEmojisCategory(newEmojis);
    }
  }
}
</script>

<style lang="scss" scoped>
.emoji-picker {
  --ep-color-bg: #fff;
  --ep-color-sbg: #f6f6f6;
  --ep-color-border: #d8dde5;
  --ep-color-text: #4a4a4a;
  --ep-color-active: #0ad47e;

  display: inline-flex;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeSpeed;
  flex-direction: column;
  align-items: center;
  background-color: var(--ep-color-bg);
  border-radius: 8px;
  // border: 1px solid var(--ep-color-border);
  box-shadow: 0px 0px 8px 0px rgba(0, 22, 47, 0.15);
  overflow: hidden;
  width: 372px;
  user-select: none;

  @media screen and (max-width: 372px) {
    width: 100%;
  }
}

.dark {
  --ep-color-bg: #191b1a;
  --ep-color-sbg: #212221;
  --ep-color-border: #3e3d42;
  --ep-color-text: #f0f0f0;
  --ep-color-active: #009688;
}
</style>
