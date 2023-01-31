<template>
  <div class="hello" style="border: 1px solid #999">
    <zzl-map ref="zzlMap"
             @returnNodeOnDownHandle="returnNodeOnDownHandle"
             @returnNodeOnUpHandle="returnNodeOnUpHandle"
             width="700px" minZoom="6" height="550px"
    ></zzl-map>
  </div>
</template>

<script>
import zzlMap from "./zzlMap";
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return {
      organizationList:[],//地区集合
    }
  },
  components: {zzlMap},
  mounted() {
    this.$refs.zzlMap.selectHandle(350000,350000,0) //初始化地图
  },
  methods:{
    returnNodeOnDownHandle(item){ //下砖
      this.organizationList.some(itemm=>{
        let level = item.level==='province'?0:1
        if(itemm.areaCode == item.adcode&&level==itemm.type){
          this.distrct = itemm.id
          this.organizationType = itemm.type
          // this.searchHandle()
        }else if(itemm.distCode == item.adcode){
          this.distrct = itemm.id
          this.organizationType = itemm.type
          // this.searchHandle()
        }
      })
    },
    returnNodeOnUpHandle(item){//上砖
      this.organizationList.some(itemm=>{
        let level = item.level==='province'?0:1
        if(itemm.areaCode == item.adcode&&level==itemm.type){
          this.distrct = itemm.id
          this.organizationType = itemm.type
          // this.searchHandle()
        }else if(itemm.distCode == item.adcode){
          this.distrct = itemm.id
          this.organizationType = itemm.type
          // this.searchHandle()
        }
      })
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
