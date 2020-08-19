<template>
  <div class="canvas__wrapper">
    <div class="d-flex">
      <v-slider v-model="svgWidth" label="Width" min="10" step="10" max="100"></v-slider>
      <v-slider class="pl-10" v-model="svgHeight" label="Height" min="200" step="100" max="10000"></v-slider>
    </div>
    <div class="mb-5">
      <v-btn @click="addNewBlock" color="primary" block>Добавить</v-btn>
    </div>
    <svg 
      class="bordered" 
      ref="draggable" 
      @mouseup="handleDrop"
      :width="`${svgWidth}%`" 
      :height="`${svgHeight}px`" 
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <filter id="shadow" x="0" y="0" width="200%" height="200%">
          <feOffset result="offOut" in="SourceGraphic" dx="10" dy="10" />
          <feColorMatrix 
            result="matrixOut" 
            in="offOut" 
            type="matrix"
            values="0.6 0 0 0 0 0 0.6 0 0 0 0 0 0.6 0 0 0 0 0 1 0" 
          />
          <feGaussianBlur result="blurOut" in="matrixOut" stdDeviation="10" />
          <feBlend in="SourceGraphic" in2="blurOut" mode="normal" />
        </filter>
        <marker id="arrowhead" markerWidth="10" markerHeight="7" 
          refX="0" refY="3.5" orient="auto">
          <polygon points="0 0, 10 3.5, 0 7" fill="#cccccc" />
        </marker>
      </defs>
      <g v-for="(rect, index) in rects" :key="rect.id">
        <rect
          class="svg__block"
          @mousedown="handleDrag(rect)"  
          :x="rect.x" 
          :y="rect.y" 
          rx="10"
          ry="10"
          :width="dimensions.width" 
          :height="dimensions.height" 
          fill="white" 
          :id="rect.id"
          filter="url(#shadow)"
          stroke="#cccccc"
        ></rect>
        <!-- <path
          v-if="rects[index + 1]"
          :d="`M${rect.x + dimensions.width + 10} ${rect.y + (dimensions.height / 2)} 
              L ${rects[index + 1].x - 40} ${rects[index + 1].y + (dimensions.height / 2)}`"
          stroke="#cccccc"
          stroke-width="3"
          fill="transparent"
          marker-end="url(#arrowhead)"
        ></path> -->
        <SvgArrow 
          v-if="rects[index + 1]"
          :rect="rect"
          :rects="rects"
          :index="index"
          :dimensions="dimensions"
        />
        
        <circle r="5" :cx="rect.x" :cy="rect.y + (dimensions.height / 2)" fill="red" class="svg__block-pointer"></circle>
        <circle r="5" :cx="rect.x + dimensions.width" :cy="rect.y + (dimensions.height / 2)" fill="red" class="svg__block-pointer"></circle>
        <text :x="rect.x + 10" :y="rect.y + 30" class="title">{{ rect.name }}</text>
        <text :x="rect.x + 10" :y="rect.y + 60" class="text">{{ rect.description }}</text>
      </g>
      <path 
        v-if="lines.x.visible" 
        :d="`M0 ${lines.y.coord} L 1761 ${lines.y.coord}`" 
        stroke="orange"
        stroke-dasharray="4"
      ></path>
      <path 
        v-if="lines.y.visible" 
        :d="`M${lines.x.coord} 0 L ${lines.x.coord} ${svgHeight}`" 
        stroke="orange"
        stroke-dasharray="4"
      ></path>
    </svg>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Ref } from 'vue-property-decorator'
import SvgArrow from './SvgArrow.vue'

export interface Rect {
  id: number
  name: string
  description: string
  x: number
  y: number
}

@Component({ 
  components: { SvgArrow }
})
export default class SvgCanvas extends Vue {

  @Ref() draggable!: any
  
  svgWidth: number = 100
  svgHeight: number = 500

  lines: any = {
    x: { coord: 0, visible: false },
    y: { coord: 0, visible: false }
  }

  dimensions: any = {
    width: 150,
    height: 150
  }

  rects: Rect[] = [
    {id: 1, x: 0, y: 0, name: 'Окисление', description: 'Описание'},
    {id: 2, x: 400, y: 50, name: 'Диффузия', description: 'Описание'},
    {id: 3, x: 800, y: 0, name: 'Штамповка', description: 'Описание'},
    {id: 4, x: 1200, y: 0, name: 'Гальваника', description: 'Описание'},
    {id: 5, x: 1600, y: 0, name: 'Упаковка', description: 'Описание'},
  ]

  dragClientX: number | null = null
  dragClientY: number | null = null

  addNewBlock() {
    const newBlock: any = {};
    const last = this.rects[this.rects.length - 1]

    newBlock.id = last.id + 1
    newBlock.name = this.rects[Math.floor(Math.random() * 4)].name
    newBlock.description = this.rects[Math.floor(Math.random() * 4)].description

    if(this.rects.length % 5 !== 0) {
      newBlock.x = last.x + 400
      newBlock.y = last.y
    } else {
      newBlock.x = 0
      newBlock.y = last.y + 300
      this.svgHeight += 300
    }

    this.rects.push(newBlock)
  }

  handleDrag(rect: Rect) {

    this.dragClientX = (<MouseEvent>event).clientX - rect.x
    this.dragClientY = (<MouseEvent>event).clientY - rect.y

    this.move = this.move.bind(this, rect)

    this.draggable.addEventListener('mousemove', this.move)
  }

  handleDrop() {
    this.dragClientX = this.dragClientY = null;
    this.draggable.removeEventListener('mousemove', this.move)
    this.draggable.classList.remove('not-selectable')

    this.lines.x.visible = this.lines.y.visible = false

    this.move = this.defaultMove
  }

  move(rect: Rect) {
    this.draggable.classList.add('not-selectable')
    this.lines.x.visible = this.lines.y.visible = false
    let mouseX = (<MouseEvent>event).clientX - this.dragClientX!
    let matchesX = this.rects.filter(r => (mouseX - r.x < 15 && mouseX - r.x > -15 || r.x - mouseX < 15 && r.x - mouseX > -15) && rect.id !== r.id)
    let mouseY = (<MouseEvent>event).clientY - this.dragClientY!
    let matchesY = this.rects.filter(r => (mouseY - r.y < 15 && mouseY - r.y > -15 || r.y - mouseY < 15 && r.y - mouseY > -15) && rect.id !== r.id)
    if(matchesY.length && matchesX.length) {
      this.lines.x.visible = this.lines.y.visible = true
      this.lines.x.coord = rect.x + this.dimensions.height / 2
      this.lines.y.coord = rect.y + this.dimensions.width / 2
      rect.x = matchesX[0].x
      rect.y = matchesY[0].y 
    } else if(matchesY.length) {
      this.lines.x.visible = true
      this.lines.y.coord = rect.y + this.dimensions.width / 2
      rect.x = (<MouseEvent>event).clientX - this.dragClientX!
      rect.y = matchesY[0].y 
    } else if(matchesX.length) {
      this.lines.y.visible = true
      this.lines.x.coord = rect.x + this.dimensions.height / 2
      rect.y = (<MouseEvent>event).clientY - this.dragClientY!
      rect.x = matchesX[0].x
    } else {
      rect.x = (<MouseEvent>event).clientX - this.dragClientX!
      rect.y = (<MouseEvent>event).clientY - this.dragClientY!
    }
  }

  defaultMove(rect: Rect) {
    this.draggable.classList.add('not-selectable')
    this.lines.x.visible = this.lines.y.visible = false
    let mouseX = (<MouseEvent>event).clientX - this.dragClientX!
    let matchesX = this.rects.filter(r => (mouseX - r.x < 15 && mouseX - r.x > -15 || r.x - mouseX < 15 && r.x - mouseX > -15) && rect.id !== r.id)
    let mouseY = (<MouseEvent>event).clientY - this.dragClientY!
    let matchesY = this.rects.filter(r => (mouseY - r.y < 15 && mouseY - r.y > -15 || r.y - mouseY < 15 && r.y - mouseY > -15) && rect.id !== r.id)
    if(matchesY.length && matchesX.length) {
      this.lines.x.visible = this.lines.y.visible = true
      this.lines.x.coord = rect.x + this.dimensions.height / 2
      this.lines.y.coord = rect.y + this.dimensions.width / 2
      rect.x = matchesX[0].x
      rect.y = matchesY[0].y 
    } else if(matchesY.length) {
      this.lines.x.visible = true
      this.lines.y.coord = rect.y + this.dimensions.width / 2
      rect.x = (<MouseEvent>event).clientX - this.dragClientX!
      rect.y = matchesY[0].y 
    } else if(matchesX.length) {
      this.lines.y.visible = true
      this.lines.x.coord = rect.x + this.dimensions.height / 2
      rect.y = (<MouseEvent>event).clientY - this.dragClientY!
      rect.x = matchesX[0].x
    } else {
      rect.x = (<MouseEvent>event).clientX - this.dragClientX!
      rect.y = (<MouseEvent>event).clientY - this.dragClientY!
    }
  }
}
</script>

<style lang="scss" scoped>

.bordered {
  border: 1px dashed rgba(0, 0, 0, 0.15);
  &.not-selectable {
    user-select: none;
  }
}
.title {
  font-size: 20px;
  color: black;
}
.text {
  font-size: 14px;
  color: black;
}
.svg__block {
  cursor: grab;
}
.svg__block-pointer {
  fill: transparent;
  &:hover {
    fill: red;
  }
}
</style>