<template>
  <l-map
    ref="map"
    v-if="showMap"
    :zoom="zoom"
    :min-zoom="minZoom"
    :crs="crs"
    :options="mapOptions"
    :zoom-animation="!config.noZoomAnimation"
    :marker-zoom-animation="!config.noMarkerZoomAnimation"
    :max-bounds="bounds"
    :key="mapKey"
    @update:bounds="ready = true"
    class="oh-plan-page-lmap"
    :class="{ 'with-tabbar': context.tab,
              'oh-plan-white-background': config.backgroundColor === 'white',
              'oh-plan-black-background': config.backgroundColor === 'black',
              'oh-plan-blackwhite-background': config.backgroundColor === 'blackwhite',
              'oh-plan-dark-mode-invert': config.darkModeInvert,
    }"
    @update:center="centerUpdate"
    @update:zoom="zoomUpdate">
    <l-image-overlay
      :url="backgroundImageUrl"
      :bounds="bounds" />
    <l-feature-group ref="featureGroup" v-if="context.component.slots && ready">
      <component v-for="(marker, idx) in markers" :key="idx"
                 :is="markerComponent(marker)" :context="childContext(marker)" @update="onMarkerUpdate" />
    </l-feature-group>
    <l-control v-if="context.editmode != null" position="topright">
      <f7-menu class="padding">
        <f7-menu-item @click="context.editmode.addWidget(context.component, 'oh-plan-marker')" icon-f7="plus" text="Add Marker" />
        <f7-menu-item v-if="context.clipboardtype" @click="context.editmode.pasteWidget(context.component)" icon-f7="square_on_square" />
      </f7-menu>
    </l-control>
    <l-control v-if="context.editmode != null" position="bottomleft">
      <span>Zoom Level: {{ currentZoom.toFixed(2) }}</span>
    </l-control>
  </l-map>
</template>

<style lang="stylus">
.oh-plan-page-lmap
  position absolute
  background-color var(--f7-page-bg-color)
  top calc(var(--f7-safe-area-top) + var(--f7-navbar-height))
  height calc(100% - var(--f7-safe-area-top) - var(--f7-navbar-height)) !important
  &.with-tabbar
    height calc(100% - var(--f7-safe-area-top) - var(--f7-navbar-height) - var(--f7-tabbar-labels-height)) !important
  &.oh-plan-white-background, &.oh-plan-blackwhite-background
    background-color white
  &.oh-plan-black-background
    background-color black
.theme-dark
  .oh-plan-page-lmap
    &.oh-plan-blackwhite-background
      background-color black
    &.oh-plan-dark-mode-invert .leaflet-image-layer
      filter invert()
</style>

<script>
import mixin from '../widget-mixin'
import { CRS, Icon } from 'leaflet'
import { LMap, LImageOverlay, LFeatureGroup, LControl } from 'vue2-leaflet'
import 'leaflet/dist/leaflet.css'

import OhPlanMarker from './oh-plan-marker.vue'
import { OhPlanPageDefinition } from '@/assets/definitions/widgets/plan'

delete Icon.Default.prototype._getIconUrl
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png')
})

export default {
  mixins: [mixin],
  components: {
    LMap,
    LImageOverlay,
    LControl,
    LFeatureGroup,
    OhPlanMarker
  },
  widget: OhPlanPageDefinition,
  data () {
    return {
      ready: false,
      currentZoom: 13,
      currentCenter: null,
      minZoom: -2,
      zoom: -0.5,
      crs: CRS.Simple,
      showMap: false,
      mapKey: this.$f7.utils.id(),
      markers: []
    }
  },
  computed: {
    bounds () {
      const lat = this.config.imageHeight || 1000
      const lng = this.config.imageWidth || 1000
      return [[0, 0], [lat, lng]]
    },
    mapOptions () {
      return Object.assign({
        zoomSnap: 0.1
      }, this.config.noZoomOrDrag ? {
        dragging: false,
        touchZoom: false,
        doubleClickZoom: false,
        scrollWheelZoom: false,
        zoomControl: false
      } : {})
    }
  },
  asyncComputed: {
    backgroundImageUrl () {
      return this.$oh.media.getImage(this.config.imageUrl)
    }
  },
  watch: {
    'config.noZoomOrDrag': function (val) {
      this.refreshMap()
    },
    backgroundImageUrl (val) {
      this.showMap = true
      this.refreshMap()
    }
  },
  methods: {
    zoomUpdate (zoom) {
      this.currentZoom = zoom
      const allMarkers = this.context.component.slots.default
      const visibleMarkers = allMarkers.filter((e) => {
        const zoomVisibilityMin = parseFloat(e.config.zoomVisibilityMin)
        const zoomVisibilityMax = parseFloat(e.config.zoomVisibilityMax)
        const isVisibleMin = isNaN(zoomVisibilityMin) || zoomVisibilityMin < this.currentZoom
        const isVisibleMax = isNaN(zoomVisibilityMax) || zoomVisibilityMax > this.currentZoom
        return this.context.editmode != null || (isVisibleMin && isVisibleMax)
      })
      // only update our markers if the list has changed to avoid unessesary rendering
      if (visibleMarkers.length !== this.markers.length ||
        visibleMarkers.every((e) => this.markers.indexOf(e) < 0)) {
        this.markers = visibleMarkers
      }
    },
    centerUpdate (center) {
      this.currentCenter = center
    },
    markerComponent (marker) {
      return 'oh-plan-marker'
    },
    onMarkerUpdate () {
    },
    fitMapBounds () {
      if (this.$refs.map) this.$refs.map.mapObject.fitBounds(this.bounds)
    },
    refreshMap () {
      this.mapKey = this.$f7.utils.id()
      this.$nextTick(() => {
        this.fitMapBounds()
      })
    }
  }
}
</script>
