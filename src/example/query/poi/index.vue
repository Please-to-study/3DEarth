<template>
  <mars-dialog :visible="true" right="10" top="10" width="330">
    <div>
      <a-space align="start">
        <span class="mars-pannel-item-label">服务：</span>
        <mars-select v-model:value="selectService" :options="serviceOptions" @change="changeService"> </mars-select>
      </a-space>
    </div>


    <div class="f-pt f-mb">
      <a-space align="start">
        <span class="mars-pannel-item-label">范围：</span>
        <a-radio-group v-model:value="radioFanwei">
          <a-radio value="1">指定城市</a-radio>
          <a-radio value="2">当前视域</a-radio>
          <a-radio value="3">指定范围</a-radio>
        </a-radio-group>
      </a-space>
    </div>

    <div class="f-pt" v-show="radioFanwei === '1'">
      <a-space>
        <span class="item-label">城市：</span>
        <Cascader v-model:value="value" :options="options" popupClassName="mars-select-dropdown" @change="onChange">
        </Cascader>
      </a-space>
    </div>


    <div class="f-pt f-mb">
      <a-space>
        <span class="mars-pannel-item-label">关键字：</span>
        <mars-input v-model:value="serverName" placeholder="查询名称和地址"></mars-input>
      </a-space>

    </div>


    <div v-show="radioFanwei === '3'" class="range-select f-mb">
      <a-space>
        <span class="item-label">框选：</span>
        <div>
          <a-space>
            <mars-button @click="drawRectangle">框选范围</mars-button>
            <mars-button @click="drawCircle">圆形范围</mars-button>
            <mars-button class="long-btn" @click="drawPolygon">多边形范围</mars-button>
          </a-space>
        </div>
      </a-space>

    </div>

    <div class="f-mb">
      <a-space>
        <mars-button @click="query">查询</mars-button>
        <mars-button @click="removeAll" danger>清除</mars-button>
      </a-space>
    </div>


    <div v-show="tableShow">
      <mars-table :pagination="true" :dataSource="dataSource" :columns="columns" :custom-row="customRow" size="small"
        bordered :scroll="{ y: 400 }" />
    </div>
  </mars-dialog>
</template>

<script setup lang="ts">
import { computed, onBeforeMount, ref, toRaw } from "vue"
import axios from "axios"
import * as mapWork from "./map.js"
import { $message } from "@mars/components/mars-ui/index"
import { Cascader } from "ant-design-vue"

interface DataItem {
  key: number
  name: string
  graphic: any
}

const selectService = ref("gaode")
const serviceOptions = ref([
  {
    value: "tdt",
    label: "天地图服务"
  },
  {
    value: "gaode",
    label: "高德服务"
  },
  {
    value: "baidu",
    label: "百度服务"
  }
])
const changeService = () => {
  mapWork.changeService(selectService.value)
}

let cityShi = "合肥市"
let cityShiKey = "340100"

const radioFanwei = ref("1")
const serverName = ref("")
const tableShow = computed(() => dataSource.value.length > 0)

const columns = ref([
  {
    title: "序号",
    dataIndex: "key",
    key: "key",
    width: 50
  },
  {
    title: "名称",
    dataIndex: "name",
    key: "name"
  },
  {
    title: "类型",
    dataIndex: "type",
    key: "type"
  },
  {
    title: "地址",
    dataIndex: "address",
    key: "address"
  }
])

const customRow = (record: DataItem) => {
  return {
    onClick: () => {
      if (record.graphic == null) {
        $message(record.name + " 无经纬度坐标信息！")
        return
      }
      mapWork.flyToGraphic(toRaw(record.graphic))
    }
  }
}

// 绘制范围
const drawRectangle = () => {
  mapWork.drawRectangle()
}

const drawCircle = () => {
  mapWork.drawCircle()
}

const drawPolygon = () => {
  mapWork.drawPolygon()
}

// 查询数据
const query = () => {
  mapWork.clearAll(true)
  if (selectService.value === "tdt") {
    mapWork.query(radioFanwei.value, cityShiKey, serverName.value)
  } else {
    mapWork.query(radioFanwei.value, cityShi, serverName.value)
  }
}

// 城市的数据
interface Option {
  value: string
  label: string
  children?: Option[]
}

const value = ref<string[]>(["340000", "340100"])
const options = ref<Option[]>([])

// 读取JSON数据
function fetchAttrJson() {
  return axios.get(`//data.mars3d.cn/file/geojson/areas/config.json`)
}

onBeforeMount(async () => {
  const { data }: any = await fetchAttrJson()
  options.value = data
})

// 改变选择的城市
const onChange = (_value: any, selectedOptions: Option[]) => {
  const sel = selectedOptions[selectedOptions.length - 1]
  cityShi = sel.label
  cityShiKey = sel.value
}

// 表格数据
const dataSource = ref<any>([])

mapWork.eventTarget.on("tableData", (e: any) => {
  dataSource.value = []

  e.data.forEach((item: any, index: number) => {
    dataSource.value.push({ key: index, name: item.name, type: item.type, address: item.address, graphic: item.graphic })
  })
})

// 清除数据
const removeAll = () => {
  dataSource.value = []
  mapWork.clearAll()
}
</script>
<style scoped lang="less">
.ant-input,
.ant-cascader {
  width: 230px !important;
}

.mars-button {
  width: 146px !important;
}

.ant-select {
  width: 200px;
}

.range-select {
  .mars-button {
    width: 72px !important;
  }

  .long-btn {
    padding-left: 1px;
  }
}

:deep(.ant-table-row:nth-of-type(even)) {
  background-color: transparent !important;
}

.item-label {
  margin-left: 18px;
  color: rgba(234, 242, 255, 0.8);
}
</style>
