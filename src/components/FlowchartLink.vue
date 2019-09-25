<template>
  <g @mouseover="handleMouseOver"
    @mouseleave="handleMouseLeave">
    <path :d="dAttr" :style="pathStyle"></path>
    <text 
      :x="labX"
      :y="labY"
      @click="handleNodeTypeClick">{{type}}</text>
    <a v-if="show.delete" @click="deleteLink">
      <text 
        text-anchor="middle" 
        :transform="arrowTransform"
        font-size="22">&times;</text>
    </a>
    <path v-else d="M -1 -1 L 0 1 L 1 -1 z"
      :style="arrowStyle"
      :transform="arrowTransform"></path>
  </g>
</template>

<script>
export default {
  name: 'FlowchartLink',
  props: {
    // start point position [x, y]
    start: {
      type: Array,
      default() {
        return [0, 0]
      }
    },
    // end point position [x, y]
    end: {
      type: Array,
      default() {
        return [0, 0]
      }
    },
    type: {
      type: String,
      default: 'Next'
    },
    id: Number,
  },
  data() {
    return {
      show: {
        delete: false,
      },
      linkTypes: ['Next', 'True', 'False'],
    }
  },
  methods: {
    handleMouseOver() {
      if (this.id) {
        this.show.delete = true;
      }
    },
    handleMouseLeave() {
      this.show.delete = false;
    },
    handleNodeTypeClick() {
      this.type = this.linkTypes[(1 + this.linkTypes.indexOf(this.type)) % this.linkTypes.length]
    },
    caculateCenterPoint() {
      // caculate arrow position: the center point between start and end
      const dx = (this.end[0] - this.start[0]) / 2;
      const dy = (this.end[1] - this.start[1]) / 2;
      return [this.start[0] + dx, this.start[1] + dy];
    },
    caculateRotation() {
      // caculate arrow rotation
      const angle = -Math.atan2(this.end[0] - this.start[0], this.end[1] - this.start[1]);
      const degree = angle * 180 / Math.PI;
      return degree < 0 ? degree + 360 : degree;
    },
    deleteLink() {
      this.$emit('deleteLink')
    },
    calculateColor() {
      var stroke;
      const green = 'rgb(65, 195, 88)'
      const red = 'rgb(255, 85, 85)'
      const def = 'rgb(72, 92, 173)'
      switch(this.type) {
        case 'True': stroke = green; break;
        case 'False': stroke = red; break;
        default: stroke = def;
      }
      return stroke;
    }
  },
  computed: {
    pathStyle() {
      return {
        stroke: this.calculateColor(),
        strokeWidth: 2.73205,
        fill: 'none',
      }
    },
    arrowStyle() {
      return {
        stroke: this.calculateColor(),
        strokeWidth: 5.73205,
        fill: 'none',
      }
    },
    arrowTransform() {
      const [arrowX, arrowY] = this.caculateCenterPoint();
      const degree = this.caculateRotation()
      return `translate(${arrowX}, ${arrowY}) rotate(${degree})`;
    },
    dAttr() {
      let cx = this.start[0], cy = this.start[1], ex = this.end[0], ey = this.end[1];
      let x1 = cx, y1 = cy + 50, x2 = ex, y2 = ey - 50;
      return `M ${cx}, ${cy} C ${x1}, ${y1}, ${x2}, ${y2}, ${ex}, ${ey}`;
    },
    labX() {
      return Math.sign(this.end[0] - this.start[0]) > 0 ? this.start[0] + 40 : this.start[0] - 70;
    },
    labY() {
      return this.start[1] + 30 * Math.sign(this.end[1] - this.start[1])
    }
  }
}
</script>

<style scoped lang="scss">
g {
  cursor: pointer;
}
</style>