<template>
  <g>
    <path
      :d="reversedCurve ? curveY : `M${startX} ${startY} ${curveX} L ${endX} ${endY}`"
      stroke="#cccccc"
      stroke-width="3"
      fill="transparent"
      marker-end="url(#arrowhead)"
    ></path>
  </g>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator'
import { Rect } from './SvgCanvas.vue'

@Component
export default class SvgArrow extends Vue {

  @Prop() rect!: Rect
  @Prop() rects!: Rect[]
  @Prop() index!: number
  @Prop() dimensions!: any

  get nextRect() {
    return this.rects[this.index + 1]
  }

  get startX(): number {
    return this.rect.x + this.dimensions.width + 10
  }
  get startY(): number {
    return this.rect.y + (this.dimensions.height / 2)
  }

  get endX(): number {
    return this.nextRect.x - 40
  }
  get endY(): number {
    return this.nextRect.y + (this.dimensions.height / 2)
  }

  get reversedCurve(): boolean {
    return this.rect.x + this.dimensions.width + 80 > this.nextRect.x 
  }

  get curveX(): string {
      const rangeX = this.nextRect.x - this.rect.x - this.dimensions.width
      return `H ${this.rect.x + this.dimensions.width + rangeX / 2} V ${this.endY}`
  }

  get curveY(): string {
    const rangeY = this.nextRect.y - this.rect.y - this.dimensions.height
    if(this.rect.y > this.nextRect.y) {
      return `M${this.rect.x + this.dimensions.width / 2} ${this.rect.y - 10} 
              V ${this.rect.y + this.dimensions.height +  rangeY / 2} 
              H ${this.nextRect.x + this.dimensions.height / 2}
              L ${this.nextRect.x + this.dimensions.width / 2} ${this.nextRect.y + this.dimensions.height + 40}`
    } else {
      return `M${this.rect.x + this.dimensions.width / 2} ${this.rect.y + this.dimensions.height + 10} 
              V ${this.rect.y + this.dimensions.height +  rangeY / 2} 
              H ${this.nextRect.x + this.dimensions.height / 2}
              L ${this.nextRect.x + this.dimensions.width / 2} ${this.nextRect.y - 40}`
    }
  }

}
</script>