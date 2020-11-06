<template>
  <div class="vue-world-map">
    <Map
      :codeLang="LangUser"
      @hoverCountry="onHoverCountry"
      @hoverLeaveCountry="onHoverLeaveCountry"
    />
    <transition name="fade">
      <div
        v-show="legend.name && notShow"
        class="vue-map-legend"
        :style="
          'left:' +
            (position.left + positionLeftTooltip) +
            'px; top: ' +
            (position.top + positionTopTooltip) +
            'px;' +
            `box-shadow:${legendBoxShadowCss}; border: ${legendBorderCss};`
        "
      >
        <div
          class="vue-map-legend-header"
          :style="
            `background: ${legendHeaderBackgroundColor}; color: ${legendFontColorHeader}`
          "
        >
          <span v-if="notShow">{{ legend.name }}</span>
        </div>
        <div
          class="vue-map-legend-content"
          :style="
            `background: ${legendContentBackgroundColor}; color: ${legendFontColorContent}`
          "
        >
          <span v-if="notShow">{{
            renderValue(countryData[legend.code]) || 0
          }}</span>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import chroma from 'chroma-js'
import Map from './Map'
import {
  getDynamicMapCss,
  getBaseCss,
  getCombinedCssString
} from './dynamic-map-css'

let legend = {
  data: null,
  code: null,
  name: null
}

let position = {
  left: 0,
  top: 0
}

export default {
  name: 'MapChart',
  components: { Map },
  watch: {
    countryData () {
      this.renderMapCSS()
    }
  },
  props: {
    lowColor: {
      type: String,
      default: '#fde2e2'
    },
    LangUser: {
      type: String,
      default: 'en'
    },
    highColor: {
      type: String,
      default: '#d83737'
    },
    countryData: {
      type: Object,
      required: true
    },
    defaultCountryFillColor: {
      type: String,
      default: '#dadada'
    },
    countryStrokeColor: {
      type: String,
      default: '#909090'
    },
    showEmptyValue: {
      type: Boolean,
      default: true
    },
    legendHeaderBackgroundColor: {
      type: String,
      default: ''
    },
    legendContentBackgroundColor: {
      type: String,
      default: ''
    },
    legendBorderCss: {
      type: String,
      default: '0px solid #acacad'
    },
    positionLeftTooltip: {
      type: Number,
      default: 30
    },
    positionTopTooltip: {
      type: Number,
      default: -20
    },
    legendBoxShadowCss: {
      type: String,
      default: '0px 0px 15px rgba(182, 182, 182, 0.407)'
    },
    legendFontColorHeader: {
      type: String,
      default: ''
    },
    legendFontColorContent: {
      type: String,
      default: ''
    },
    currencyAdd: {
      type: Boolean,
      default: true
    },
    currencyOnlySign: {
      type: Boolean,
      default: true
    },
    currencyCurrent: {
      type: String,
      default: 'USD'
    }
  },
  data () {
    return {
      legend: legend,
      position: position,
      node: document.createElement('style'),
      chromaScale: chroma.scale([this.$props.lowColor, this.$props.highColor]),
      keysLegend: [],
      notShow: true
    }
  },
  computed: {
    getKeyLegend () {
      Object.keys(this.$props.countryData).forEach(key => {
        if (key === 'unknown') return
        this.keysLegend.push(key)
      })
      return this.keysLegend
    }
  },
  methods: {
    onHoverCountry (country) {
      this.legend = country
      this.notShowLegendEmpty()
      this.position = country.position
      this.$emit('hoverCountry', country)
    },
    onHoverLeaveCountry (country) {
      this.legend = {
        data: null,
        code: null,
        name: null
      }
      this.$emit('hoverLeaveCountry', country)
    },

    notShowLegendEmpty () {
      if (!this.countryData[this.legend.code] && !this.showEmptyValue) {
        this.notShow = false
      } else {
        this.notShow = true
      }
    },
    renderMapCSS () {
      const baseCss = getBaseCss(this.$props)
      const dynamicMapCss = getDynamicMapCss(
        this.$props.countryData,
        this.chromaScale
      )
      this.$data.node.innerHTML = getCombinedCssString(baseCss, dynamicMapCss)
    },
    currencyNumber (value, currency, lang) {
      let formatter
      formatter = new Intl.NumberFormat(lang, {
        style: 'currency',
        currency: currency
      })
      if (this.currencyOnlySign) {
        return formatter
          .format(value)
          .replace('USD', '')
          .replace('US', '')
      }
      return formatter.format(value)
    },
    renderValue (value) {
      if (!this.currencyAdd) {
        return value
      }
      return this.currencyNumber(value, this.currencyCurrent, this.LangUser)
    }
  },
  mounted () {
    document.body.appendChild(this.$data.node)
    this.renderMapCSS()
    this.notShowLegendEmpty()
  },
  created () {
    this.notShowLegendEmpty()
  }
}
</script>

<style scoped>
.vue-world-map,
#map-svg {
  height: 100%;
}
.vue-world-map * {
  transition: all 0.2s ease;
}
.vue-world-map {
  position: relative;
}

.vue-map-legend {
  width: 185px;
  min-height: 50px;
  background: #fff;
  border: 0px solid;
  border-color: #acacad;
  position: absolute;
  z-index: 200;
  font-weight: 600;
  /* box-shadow: 0px 0px 15px rgba(182, 182, 182, 0.407); */
  border-radius: 4px;
}

.vue-map-legend-header {
  padding: 10px 15px;
}

.vue-map-legend-content {
  padding: 10px 15px;
  background: #fff;
  font-weight: 500;
  border-radius: 4px;
  border-top: 0px solid #acacad;
}

.fade-left-enter-active,
.fade-left-leave-active {
  transition: all 0.3s;
  opacity: 1;
  transform: translateY(0px);
}
.fade-left-enter,
.fade-left-leave-to {
  opacity: 0;
  transition: all 0.3s;
  transform: translateX(50%);
}
</style>
