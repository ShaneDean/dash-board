<template>
  <div
    v-if="isInit"
    ref="db-render-wrap"
    :key="`${pageInfo.pageConfig.w}${pageInfo.pageConfig.h}`"
    class="db-render-wrap design-drag-wrap render-theme-wrap"
    :style="{
      width: '100%',
      height: '100%',
      backgroundColor: pageInfo.pageConfig.bgColor,
      backgroundImage: `url(${pageInfo.pageConfig.bg})`
    }"
    @drop="drop($event)"
    @dragover.prevent
  >
    <grid-layout
      ref="gridlayout"
      :layout.sync="chartList"
      :col-num="24"
      :row-height="10"
      :responsive="true"
      :is-resizable="true"
      :vertical-compact="true"
      :use-css-transforms="true"
      :is-draggable="true"
      :margin="[16, 16]"
      :cols="{ lg: 24, md: 24, sm: 24, xs: 24, xxs: 12 }"
    >
      <grid-item
        v-for="(card) in chartList"
        :id="`${card.code}`"
        :ref="`layoutItem_${card.code}`"
        :key="card.code"
        :min-w="1"
        :min-h="2"
        :x="card.x"
        :y="card.y"
        :w="card.w"
        :h="card.h"
        :i="card.i"
        @resized="resizestop"
        @moved="dragstop"
      >
        <Configuration
          v-if="isInit"
          :config="card"
          @openRightPanel="openRightPanel"
        >
          <RenderCard
            :ref="'RenderCard' + card.code"
            :config="card"
          />
        </Configuration>
      </grid-item>
    </grid-layout>
  </div>
</template>
<script>
import { mapState, mapMutations } from 'vuex'
import RenderCard from './RenderCard.vue'
import Configuration from './Configuration.vue'
import { randomString } from '../js/utils'
import { compile } from 'tiny-sass-compiler/dist/tiny-sass-compiler.esm-browser.prod.js'
import plotList, { getCustomPlots } from '../G2Plots/plotList'
import VueGridLayout from 'vue-grid-layout'
import _ from 'lodash'
export default {
  name: 'DashboardRender',
  components: {
    RenderCard,
    Configuration,
    GridLayout: VueGridLayout.GridLayout,
    GridItem: VueGridLayout.GridItem
  },
  props: {
    ruleKey: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      vLine: [],
      hLine: [],
      themeCss: '',
      // 临时冻结拖拽
      freeze: false,
      plotList
    }
  },
  computed: {
    ...mapState({
      pageConfig: (state) => state.dashboard.pageInfo.pageConfig,
      pageInfo: (state) => state.dashboard.pageInfo,
      // chartList: (state) => state.dashboard.pageInfo.chartList,
      activeCode: (state) => state.dashboard.activeCode,
      hoverCode: (state) => state.dashboard.hoverCode,
      themeJson: (state) => state.dashboard.pageInfo.pageConfig.themeJson,
      isInit: (state) => !state.dashboard.pageLoading,
      scale: (state) => state.dashboard.zoom / 100
    }),
    chartList: {
      get () {
        return this.pageInfo.chartList
      },
      set (val) {}
    }
  },
  watch: {
    pageConfig: {
      handler (pageConfig) {
        this.$nextTick(() => {
          const style = document.createElement('style')
          if (
            pageConfig &&
            pageConfig.themeJson &&
            pageConfig.themeJson.themeCss
          ) {
            const themeCss = pageConfig.themeJson.themeCss
            if (themeCss) {
              const themeStr = compile(themeCss).code
              style.type = 'text/css'
              style.innerText = themeStr
              document.getElementsByTagName('head')[0].appendChild(style)
            }
          }
        })
      },
      deep: true,
      immediate: true
    }
  },
  mounted () {
    this.styleSet()
    this.plotList = [...this.plotList, ...getCustomPlots()]
  },
  methods: {
    ...mapMutations('dashboard', [
      'changeLayout',
      'changeActiveCode',
      'changeChartConfig',
      'changeActiveItemWH',
      'changeActiveItemConfig',
      'addItem',
      'delItem',
      'resetPresetLine',
      'changeGridShow',
      'setPresetLine',
      'saveTimeLine'
    ]),
    // 获取到后端传来的主题样式并进行修改
    styleSet () {
      const style = document.createElement('style')
      if (this.themeJson && this.themeJson.themeCss) {
        const styleStr = this.themeJson.themeCss
        const themeCss = compile(styleStr).code
        style.type = 'text/css'
        style.innerText = themeCss
        document.getElementsByTagName('head')[0].appendChild(style)
      } else {
        style.remove()
      }
    },
    // 点击当前组件时打开右侧面板
    openRightPanel (config) {
      this.$emit('openRightPanel', config)
    },
    drop (e) {
      e.preventDefault()
      // 解决：火狐拖放后，总会默认打开百度搜索，如果是图片，则会打开图片的问题。
      e.stopPropagation()
      const transferData = e.dataTransfer.getData('dragComponent')
      if (transferData) {
        this.addChart(transferData, { x: e?.x, y: e?.y })
      }
    },
    resizestop (i, height, width) {
      const chart = this.chartList?.find((item) => item.code === i)
      const newChart = {
        ...chart,
        w: width,
        h: height
      }
      this.changeChartConfig({ code: chart.code, w: width, h: height })
      if (i === this.activeCode) {
        this.changeActiveItemWH({ code: chart.code, w: width, h: height })
      }
      this.saveTimeLine(`改变${chart?.title}大小`)
    },
    dragstop (i, left, top) {
      const chart = this.chartList?.find((item) => item.code === i)
      const newChart = {
        ...chart,
        x: left,
        y: top
      }
      if (!this.freeze) {
        this.changeChartConfig({ code: chart.code, x: left, y: top })
        if (i === this.activeCode) {
          this.changeActiveItemWH({ code: chart.code, x: left, y: top })
        }
      } else {
        const index = this.chartList.findIndex(
          (_chart) => _chart.code === chart.code
        )
        this.$set(this.chartList, index, chart)
        this.changeChartConfig({
          ...chart,
          updateKey: new Date().getTime()
        })
      }
      this.changeGridShow(false)
      this.freeze = false
      this.saveTimeLine(`拖拽${chart?.title}`)
    },
    // 新增元素
    addChart (chart, position) {
      const _chart = (typeof chart === 'string') ? JSON.parse(chart) : chart
      let option = _chart.option
      if (_chart.type === 'customComponent') {
        option = {
          ...this.plotList?.find((plot) => plot.name === _chart.name)?.option,
          theme: this.pageConfig.customTheme
        }
      }
      const code = !chart.code ? randomString(8) : chart.code
      const config = {
        ..._chart,
        x: 0,
        y: 0,
        code,
        i: code,
        option
      }
      config.key = config.code
      const index = this.chartList.length
      this.addChatInLayout(config, index, this.chartList)
    },
    addSourceChart (chart, position) {
      const _chart = JSON.parse(chart)
      let option = _chart.option
      if (_chart.type === 'customComponent') {
        option = {
          ...this.plotList?.find((plot) => plot.name === _chart.name)?.option,
          theme: this.pageConfig.customTheme
        }
      }
      const code = !chart.code ? randomString(8) : chart.code
      const config = {
        ..._chart,
        w: 12,
        h: 10,
        code,
        i: code,
        option
      }
      config.key = config.code
      const index = this.chartList.length
      this.addChatInLayout(config, index, this.chartList)
    },

    addChatInLayout (item, index, layout) {
      // 初始化元素
      const newItem = _.cloneDeep(item)
      // 确定边界
      const Ys = []
      let maxY = 0
      let edgeX = 0
      let edgeY = 0
      // eslint-disable-next-line array-callback-return
      layout.map(l => {
        Ys.push(l.y || 0 + l.h || 10)
      })
      // eslint-disable-next-line no-mixed-operators
      maxY = Ys.length && Math.max.apply(null, Ys) || 1
      edgeX = 24
      edgeY = maxY
      // 使用二维数组生成地图
      const gridMap = []

      for (let x = 0; x < edgeX; x++) {
        gridMap[x] = []
        for (let y = 0; y < edgeY; y++) {
          gridMap[x][y] = 0
        }
      }
      // 标记占位
      // eslint-disable-next-line array-callback-return
      layout.map(l => {
        // 将layout中卡片所占区域标记为1
        for (let { x } = l; x < (l.x + l.w); x++) {
          for (let { y } = l; y < (l.y + l.h); y++) {
            gridMap[x][y] = 1
          }
        }
      })
      newItem.x = 0
      newItem.y = edgeY++
      this.$set(layout, index, newItem)
      return layout
    }
  }
}
</script>
<style lang="scss" scoped>
.db-render-wrap {
  position: relative;
  background-size: cover;
  overflow-y: auto;
  overflow-x: hidden;

  .drag-item {
    cursor: move;
  }

  /deep/ .vdr {
    border: none;
  }
  .h-line {
    border-bottom: 1px dashed #0089d0;
  }
  .v-line {
    border-left: 1px dashed #0089d0;
  }
  .ref-line {
    background-color: transparent;
  }
}
.design-drag-wrap {
  // box-shadow: 0 0 30px 0 rgba(0, 0, 0, 0.5);
  height: 100% !important;
}

/deep/.vue-grid-placeholder {
  background: #C6C8CA !important;
}

</style>
