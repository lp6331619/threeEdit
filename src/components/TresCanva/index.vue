<template>
  <TresCanvas v-bind="canvasConfig" ref="TresCanvasRef"
  @context-menu="(event:any) => console.log('context-menu (right click)',event)"
  @pointer-missed="(event:any) => transformRef=null">
    <!-- 轴 -->
    <TresAxesHelper :args="[10]" />
    <!-- 控制 -->
    <OrbitControls make-default  v-bind="cameraConfig" />
    <!-- 坐标格辅助对象 -->
    <TresGridHelper :position-y="0.1" />
    <!-- 透视摄像机 -->
    <TresPerspectiveCamera ref="cameraRef" :position="[10,10,10]" :look-at="[0, 0, 0]"/>
    <!-- <CameraControls :camera="cameraRef" ref="controlsRef" v-bind="cameraConfig" make-default  /> -->
    <!-- 环境光 -->
    <TresAmbientLight :intensity="2"/>
    <!-- 灯光 -->
    <template v-for="(item,i) in lightSetting">
      <component ref="lightRef" :is="item.type" v-bind="item.config" />
    </template>
    <template v-for="(subMesh, index) in config">
      <!-- Primitives -->
      <primitive v-if="subMesh.type == 'primitive'" :object="objectBox(subMesh)" v-bind="subMesh.option" />
      <Sky v-if="subMesh.type == 'Sky'" v-bind="subMesh.option"  />
      <Stars v-if="subMesh.type == 'Stars'" v-bind="subMesh.option" />
      <!-- 添加的mesh对象 -->
      <TresMesh
        v-if="subMesh.type == 'TresMesh'"
        ref="TresMeshRef"
        v-bind="subMesh.option"
        cast-shadow 
        :key="index" 
        :name="subMesh.name + index"
        @pointer-enter="onPointerEnter"
        @pointer-leave="onPointerLeave"
        @click="clickObject(subMesh.name + index,index,$event)"
        @context-menu="clickRight($event,subMesh)"
      >
        <!-- 球缓冲几何体 --> 
        <template v-for="(item, i) in subMesh.children">
          <component :is="item.type" v-bind="item.config" />
        </template>
      </TresMesh>
    </template>
    <!-- 变换控制器 -->
    <TransformControls v-if="transformRef" :object="transformRef" v-bind="TransformControlsState"  />
    <primitive :object="nodes.sm_car" />
  </TresCanvas>
</template>

<script setup lang="ts">
import { ref, reactive, watchEffect,nextTick, onMounted,watch, onUpdated ,shallowRef} from 'vue'
import { SRGBColorSpace, BasicShadowMap, NoToneMapping } from 'three'
import { useRenderLoop,useTresContext,vLightHelper } from '@tresjs/core'
import { OrbitControls, TransformControls,CameraControls ,Stars,Sky,GLTFModel ,useGLTF ,useEnvironment ,Environment,Backdrop ,Sphere } from '@tresjs/cientos'
import { useChartEditStore } from '@/store/modules/chartEditStore/chartEditStore'
const chartEditStore = useChartEditStore()
const canvasConfig = chartEditStore.getEditCanvasConfig
const cameraConfig = chartEditStore.getCameraConfig
const lightSetting = chartEditStore.getLightSetting
const emits = defineEmits(['click'])
const props = defineProps({
  config: {
    type: Array,
    default: () => {
      return []
    }
  },
})
import imageUrl from './models/su7_car/sm_car.gltf';
const { nodes } = await useGLTF(imageUrl)
console.log(nodes,123)
const TresCanvasRef = shallowRef()
const TresMeshRef = shallowRef()
const transformRef = shallowRef()
const cameraRef = shallowRef()
const lightRef = shallowRef([])
// 灯光
watch(()=>lightSetting,(e)=>{
  if(!e) return 
  nextTick(()=>{
    e.forEach((item:any,i:number)=>{
      if(item.config.position?.length) { 
        lightRef.value[i]?.position.set(item.config.position[0],item.config.position[1],item.config.position[2])
       }
    })
  })
},{deep:true, immediate:true})
const objectBox = async(item:any) => {
  const { nodes } = await useGLTF(JSON.parse(item.meshConfig))
  console.log(nodes,123)
  return nodes
}
const { onLoop, onBeforeLoop, onAfterLoop,pause, resume } = useRenderLoop()
// 选择模型移动
const clickObject = (name: string,i:number,e:Event) => {
  for (let s in TresMeshRef.value) {
    if (TresMeshRef.value[s].name === name) {
      transformRef.value = TresMeshRef.value[s]
    } 
  }
  emits('click', {
    ref:transformRef.value,
    config:props.config[i],
    e:e
  })
}
// const clickRight = (e:Event,item:anyObj)=>{
//   console.log(e,item)
// }
// 控制器的配置
const TransformControlsState = reactive({
  mode: 'translate',
  enabled: true,
  space: 'world',
  axis: 'XYZ',
  size: 1,
  showX: true,
  showY: true,
  showZ: true
})
onLoop((res) => {
  // console.log(res,111)
  if (!TresMeshRef.value) return
  // TresMeshRef.value.map(item => {
  //     item.position.y += Math.sin(elapsed) * 0.1
  // })
})
onAfterLoop((res)=>{
})
// 鼠标移入到模型上时，改变模型颜色
function onPointerEnter(ev: any) {
  if (ev) {
    ev.object.material.color.set('#DFFF45')
    // pause()
  }
}
// 鼠标移出时，恢复模型颜色
function onPointerLeave(ev: any) {
  if (ev) {
    ev.eventObject.material.color.set('#214A68')
    // resume()
  }
}
watchEffect(() => {})
onMounted(() => {})
</script>
