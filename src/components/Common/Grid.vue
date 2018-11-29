<template>
  <div v-if="grid.length" v-bind:class="{ 'mobile': mobilePreview }" class="builder-grid">
    <grid-layout
      :layout="grid"
      @layout-updated="handleLayoutUpdate"
      :col-num="12"
      :row-height="90"
      :vertical-compact="true"
      :is-resizable="!!editable"
      :is-draggable="!!editable"
      :use-css-transforms="true"
    >
      <grid-item
        v-for="(item, index) in grid"
        :key="item.i"
        :x="item.x"
        :y="item.y"
        :w="item.w"
        :h="item.h"
        :i="item.i"
      >
        <slot v-bind:block="item.block" v-bind:index="index"></slot>
      </grid-item>
    </grid-layout>
  </div>
  <div v-else class="builder-grid">
    <h4>No block added yet.</h4>
  </div>
</template>

<script>
import VueGridLayout from 'vue-grid-layout'

const blocksToGrid = blocks => {
  return blocks.map((block, i) => {
    return {
      i,

      x: block.x,
      y: block.y,
      w: block.width,
      h: block.height,

      block,
    }
  })
}

export default {
  props: {
    editable: {
      type: Boolean,
    },

    blocks: {
      type: Array,
      required: true,
    },
  },

  data () {
    return {
      // all blocks in vue-grid friendly structure
      grid: blocksToGrid(this.blocks),

      mobilePreview: false,
    }
  },

  watch: {
    blocks: {
      handler (blocks) {
        this.grid = blocksToGrid(blocks)
      },

      deep: true,
    },
  },

  methods: {
    handleLayoutUpdate (grid) {
      // Emit change back with update: prefix for .sync modifier to kick in.
      this.$emit(
        'update:blocks',
        grid.map(({ x, y, w, h, block }) => {
          block.merge({ x, y, width: w, height: h })
          return block
        })
      )
    },
  },

  components: {
    GridLayout: VueGridLayout.GridLayout,
    GridItem: VueGridLayout.GridItem,
  },
}
</script>

<style lang="scss" scoped>
.builder-grid {
  width: 100%;

  &.mobile {
    width: 320px;
    margin: 0 auto;
  }
}

.vue-grid-item {
  border: 1px solid #ccc;
  padding: 20px 10px 10px 10px;
}
</style>