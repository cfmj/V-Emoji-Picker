<template>
  <span
    :class="['emoji-c', { border: withBorder }]"
    :style="styleSize"
    v-html="uemoji(emoji.data)"
  />
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { IEmoji } from "@/models/Emoji";
import uEmojiParser from "universal-emoji-parser";

@Component({})
export default class EmojiItem extends Vue {
  @Prop({}) emoji!: IEmoji;
  @Prop({}) size!: number;
  @Prop({}) withBorder!: boolean;

  get styleSize() {
    return {
      fontSize: `${this.size - 5}px`,
      lineHeight: `${this.size}px`,
      height: `${this.size}px`,
      width: `${this.size}px`
    };
  }

  uemoji(data) {
    let tmp = uEmojiParser.parse(data);
    // window.__twemoji_base_url__ = 'https://static.wildfirechat.net/twemoji/assets/';
    if (window.hasOwnProperty("__twemoji_base_url__")) {
      tmp = tmp.replace(
        /src="https:\/\/twemoji\.maxcdn\.com\/v\/[0-9.]+\//g,
        'src="' + window.__twemoji_base_url__
      );
    }
    return tmp;
  }
}
</script>

<style lang="scss" scoped>
.emoji-c {
  text-align: center;
  padding: 3px;
  box-sizing: content-box;
  overflow: hidden;
  transition: transform 0.2s;

  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;

  // &:hover {
  //   transform: scale(1.15);
  // }

  ::v-deep img {
    width: 28px;
    height: 28px;
  }
}

.border:hover {
  background: rgba(0, 22, 47, 0.07);
  border-radius: 4px;
}
</style>
