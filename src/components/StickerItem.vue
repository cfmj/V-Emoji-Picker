
<template>
  <img :class="['sticker', { 'border': withBorder } ]" :style="styleSize" :src="emoji.data"  alt=""/>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { IEmoji } from "@/models/Emoji";

@Component({})
export default class StickerItem extends Vue {
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
}
</script>

<style lang="scss" scoped>
.sticker {
  display: inline-block;
  text-align: center;
  padding: 3px;
  box-sizing: content-box;
  overflow: hidden;
  transition: transform 0.2s;

  cursor: pointer;

  &:hover {
    transform: scale(1.05);
  }
}

.border:hover {
  background: #00000010;
  border-radius: 8px;
}
</style>
