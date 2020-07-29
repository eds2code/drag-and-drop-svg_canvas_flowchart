<template>
  <g>
    <rect
      @mousedown="handleDrag($event, rect)"  
      :x="rect.x" 
      :y="rect.y" 
      :width="dimensions.width" 
      :height="dimensions.height" 
      fill="red" 
      :id="rect.id"
    ></rect>
    <line v-if="rects[index + 1]" :x1="rect.x + 100" :y1="rect.y + 50" :x2="rects[index + 1].x" :y2="rects[index + 1].y + 50" stroke="black"></line>
    <text :x="rect.x" :y="rect.y + 30" class="text">blabla</text>
  </g>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class SvgBlock extends Vue {

  @Prop() private rect!: any
  @Prop() private index!: number
  @Prop() private rects!: any[]

  dimensions: any = {
    width: 100,
    height: 100
  }

  private get x1() {
    return this.rect.x + this.dimensions.width
  }
  private get y1() {
    return this.rect.y + (this.dimensions.height / 2)
  }
  private get x2() {
    return this.rects[this.index + 1].x
  }
  private get y2() {
    return this.rects[this.index + 1].y + (this.dimensions.height / 2)
  }

}
</script>