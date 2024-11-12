<template>
  <collapse-item :name="'灯光设置'">
      <n-collapse :title="'灯光'">
        <n-collapse-item v-for="(item,i) in config" :title="`灯光${i+1}`"  :name="i" >
          <setting-item-box name="灯光类型" >
            <setting-item >
              <Select v-model:value="item.type" :disabled="item.type=='TresAmbientLight'" :index="i" :list="lightTypeOptions"  @change="onChange"></Select>
            </setting-item>
            <setting-item >
              <n-button strong secondary :disabled="item.type=='TresAmbientLight'" type="error" size="small" @click="delItem(i)">
                删除灯光
              </n-button>
            </setting-item>
          </setting-item-box> 
          <setting-item-box name="位置" v-if="item.type!=='TresAmbientLight'">
            <setting-item name="X轴">
              <n-input-number v-model:value="item.config.position[0]" :step="1" size="small"></n-input-number>
            </setting-item>
            <setting-item name="Y轴">
              <n-input-number v-model:value="item.config.position[1]" :step="1" size="small"></n-input-number>
            </setting-item>
            <setting-item name="Z轴">
              <n-input-number v-model:value="item.config.position[2]" :step="1" size="small"></n-input-number>
            </setting-item>
          </setting-item-box>
          <!-- <setting-item-box name="照射位置" v-if="item.type!=='TresAmbientLight'">
            <setting-item name="X轴">
              <n-input-number v-model:value="item.config.targets[0]" :step="1" size="small"></n-input-number>
            </setting-item>
            <setting-item name="Y轴">
              <n-input-number v-model:value="item.config.targets[1]" :step="1" size="small"></n-input-number>
            </setting-item>
            <setting-item name="Z轴">
              <n-input-number v-model:value="item.config.targets[2]" :step="1" size="small"></n-input-number>
            </setting-item>
          </setting-item-box> -->
          <setting-item-box name="是否开启辅助" >
            <setting-item ><n-switch v-model:value="item.config.isShow" size="small"></n-switch></setting-item>
          </setting-item-box>
          <setting-item-box name="灯光" >
            <setting-item name="颜色">
              <n-color-picker
                size="small"
                v-model:value="item.config.color"
                :showPreview="true"
                :swatchesColors="swatchesColors"
              ></n-color-picker>
            </setting-item>
            <setting-item name="强度">
              <n-input-number v-model:value="item.config.intensity" :step="0.5" :min="0" :max="100" size="small"></n-input-number>
            </setting-item>
          </setting-item-box> 
          
          <setting-item-box v-if="item.type!='TresAmbientLight'" name="开启阴影" >
            <setting-item ><n-switch v-model:value="item.config.castShadow" size="small"></n-switch></setting-item>
            <setting-item name="旋转速度" >
              <n-input-number v-model:value="config.autoRotateSpeed" :step="1" :min="1" :max="100" size="small"></n-input-number>
            </setting-item>  
          </setting-item-box> 
        </n-collapse-item>
      </n-collapse>
      <div>
      <div class="flex">
      <n-button class="mt10" @click="addConfig">添加灯光</n-button>
    </div>
    </div>
  </collapse-item>
</template>

<script setup lang="ts">
import { ref, PropType,defineAsyncComponent } from 'vue'
import { swatchesColors } from '@/settings/chartThemes/index'
import { PickCreateComponentType, BlendModeEnumList } from '@/packages/index.d'
import { SettingItemBox, SettingItem, CollapseItem } from '@/components/Pages/ChartItemSetting'
import { icon } from '@/plugins'
import logoImg from '@/assets/logo.png'
import { useDesignStore } from '@/store/modules/designStore/designStore'
const Select = defineAsyncComponent(() => import('@/components/Select/index.vue'));
const props = defineProps({
  isGroup: {
    type: Boolean,
    required: false
  },
  config: {
    type: Object as PropType<Omit<PickCreateComponentType<'styles'>, 'animations'>>,
    required: true
  }
})
const onChange = ({index,value})=>{
  const i = lightTypeOptions.findIndex((e:any)=>e.value==value)
  props.config[index].type = value
  props.config[index].helper = lightTypeOptions[i].helper
}
const designStore = useDesignStore()
const addConfig = () => {
  props.config?.push({
    type: 'TresDirectionalLight',
    helper:'TresDirectionalLightHelper',
    config:{
      color: '#ffffff',
      intensity: 1,
      position: {
        x: 0,
        y: 5,
        z: 0
      }
    }
  })
}
// 删除
const delItem = (i:number)=>{
  props.config.length!=1 && props.config.splice(i,1)
}
// 灯光类型
const lightTypeOptions = [
  {
    label: '环境光',
    value: 'TresAmbientLight',
    disabled:true
  },
  {
    label: '平行光',
    value: 'TresDirectionalLight',
    helper:'TresDirectionalLightHelper'
  },
  {
    label: '点光源',
    value: 'TresPointLight',
    helper:'TresPointLightHelper'
  },
  {
    label: '半球光',
    value: 'TresHemisphereLight',
    helper:'TresHemisphereLightHelper'
  },
  {
    label: '聚光灯',
    value: 'TresSpotLight',
    helper:'TresSpotLightHelper'
  }
]


</script>

<style lang="scss" scoped>
// 预设滤镜
.flex{
  display: flex;
  justify-content: center;
  .mt10{
    margin-top:10px
  }
}
</style>
