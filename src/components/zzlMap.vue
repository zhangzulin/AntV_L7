<template>
  <div class="index">
    <div id="map" class="map" :style="{width: width,height:height}"></div>
  </div>
</template>

<script>
/**
 *
 * @Author zzl
 * @Date 2022/5/23 16:04.
 */
import {Scene, Mapbox} from '@antv/l7';
import {Choropleth} from '@antv/l7plot';
import area from './area-list.json'
// const {Scene, Mapbox} = L7
// const {Choropleth} = L7Plot
// var scene=null
var choropleth=null
var cityData=null
var data=null
var districtData=null
var t = null
export default {
  mounted() {
    // this.mapLoaded1()
  },
  components: {},
  props: {
    width:{
      type:String
    },
    height:{
      type:String
    },
    minZoom:{
      type:String,
      default:'3'
    }
  },
  data() {
    return {
      show: true,
      areaJson: area,
    }
  },
  computed: {},
  methods: {
    selectHandle(pid,areaCode,type) {//type：0和2:省级，其他：市级
      console.log('choropleth',choropleth)
      if (choropleth) {
        if (t == 0 &&type == 1) {
          choropleth.drillDown({
            level: type == 0?'province':'city',
            adcode: type == 0?Number(areaCode):Number(pid),
            granularity: type == 0?'city':'district'
          },{
            source: {
              data: type == 0? cityData : districtData,
            }
          })
        } else if (t == 1 && type == 1) {
          choropleth.drillUp()
          choropleth.drillDown({
            level: type == 0?'province':'city',
            adcode: type == 0?Number(areaCode):Number(pid),
            granularity: type == 0?'city':'district'
          },{
            source: {
              data: type == 0? cityData : districtData,
            }
          })
        } else if (t == 1 && type == 0) {
          choropleth.drillUp()
        }
      } else {
        console.log(pid,areaCode,type)
        this.show = false
        this.$nextTick(()=>{
          this.show = true
          setTimeout(()=>{
            this.mapUpdate(pid,areaCode,type)
          },200)
        })
      }
      t = type
    },
    mapUpdate(pid,areaCode,type) {
      const scene = new Scene({
        id: 'map',
        map: new Mapbox({
          style: 'blank',
          center: [118.28, 26.08],
          zoom: Number(this.minZoom),
          minZoom: Number(this.minZoom),
          pitch: 0,
        }),
      });
      scene.setMapStatus({
        dragEnable: false, // 是否允许地图拖拽
        keyboardEnable: false, // 是否允许形键盘事件
        doubleClickZoom: false, // 双击放大
        zoomEnable: false, // 滚动缩放
        rotateEnable: false // 旋转
      })

      data = this.areaJson
        .filter(({ level, parent }) => level === 'city' && parent === Number(areaCode))
        .map((item) =>
          Object.assign({}, item, { value: Math.random() * 5000 }),
        );
      cityData = this.areaJson
        .filter(({ level }) => level === 'city')
        .map((item) => Object.assign({}, item, { value: Math.random() * 2000 }));
      districtData = this.areaJson
        .filter(({ level }) => level === 'district')
        .map((item) => Object.assign({}, item, { value: Math.random() * 1000 }));
      choropleth = new Choropleth({
        source: {
          data: type == 0?data:districtData,
          joinBy: {
            sourceField: 'adcode',
            geoField: 'adcode',
          },
        },
        viewLevel: {
          level: type == 0?'province':'city',
          adcode: type == 0?Number(areaCode):Number(pid),
        },
        autoFit: true,
        drill: {
          steps: ['city','district'],
          triggerUp: 'unclick',
          triggerDown: 'click',
          onDown: (from, to, callback) => {
            t = 1
            this.$emit('returnNodeOnDownHandle',to)
            console.log('222', from)
            console.log('333', to)
            if(to.level === "district"&&to.granularity==="district"){
              return
            }
            const { level, granularity } = to;
            if (granularity === 'city') {
              callback({
                source: { data: cityData, joinBy: { sourceField: 'adcode' } },
              });
            } else if (granularity === 'district') {
              callback({
                source: { data: districtData, joinBy: { sourceField: 'adcode' } },
              });
            }
          },
          onUp: (from, to, callback) => {
            console.log('onUp',to)
            this.$emit('returnNodeOnUpHandle',to)
            callback();
          },
        },
        chinaBorder:{
          national:{opacity:0},
          dispute:{opacity:0},
          coast:{opacity:0},
          hkm:{opacity:0},
        },
        color: {
          field: 'value',
          value: ['#A1C9FB', '#82BDFF', '#57A5FC', '#5BA3FF', '#338DFF'],
          scale: { type: 'quantize' },
        },
        style: {
          opacity: 1,
          stroke: '#9e9e9e',
          lineWidth: 0.6,
          lineOpacity: 1,
        },
        label: {
          visible: true,
          field: 'name',
          style: {
            fill: '#000',
            opacity: 1,
            fontSize: 14,
            stroke: '#fff',
            strokeWidth: 1.5,
            textAllowOverlap: true,
            padding: [0, 0],
            textAnchor:'center'
          },
        },
        state: {
          active: {
            fill: false,
            stroke:  'rgb(97,217,171)',
            lineWidth: 1.5,
            lineOpacity: 0.8
          },
          select: {
            fill: 'rgb(97,217,171)',
            stroke: false,
            lineWidth: 1.5,
            lineOpacity: 0.8,
          }
        },
        // tooltip: {
        //   items: ['name', 'adcode', 'value'],
        // },
        // zoom: {
        //   position: 'bottomright',
        // },
        // legend: {
        //   position: 'bottomleft',
        // },
      });
      if (scene.loaded) {
        choropleth.addToScene(scene);
      } else {
        scene.on('loaded', () => {
          choropleth.addToScene(scene);
        });
      }
    },
  },
  watch: {
  },
  filters: {},
  beforeDestroy() {
    choropleth = null
  }
}
</script>

<style lang="less" scoped>
.index {
  width: 100%;
  .map {
    position: relative;
    margin: 0 auto
  }
}
/dee/.l7-ctrl-logo{
  display: none!important;
}
/deep/.l7-control-logo .l7-control{
  display: none!important;
}
</style>
