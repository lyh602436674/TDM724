<!--
 * @Author: 雷宇航
 * @Date: 2023-02-14 14:18:45
 * @fileName: PositionTemperatureHumidity.vue
 * @FilePath: tdm724-client\src\views\hifar\hifar-environmental-test\largeScreen\PositionTemperatureHumidity.vue
 * @Description: 地点环境温湿度
-->
<template>
  <div class="PositionTemHum">
    <div class="title">环境温湿度</div>
    <div class="content">
      <table class="table">
        <tr v-for="(item,index) in dataSource" :key="index" class="list">
          <td>{{ item.devicename + '：' }}</td>
          <td>{{ formatToFixed(item.tem) + '℃' }}</td>
          <td>{{ formatToFixed(item.hum) + '%RH' }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import {getAction} from "@api/manage";

export default {
  name: "PositionTemperatureHumidity",
  data() {
    return {
      dataSource: [],
      url: {
        list: "/LargeScreenDisplay/positionTemAndHum"
      }
    }
  },
  methods: {
    formatToFixed(value){
      return isNaN(Number(value)) ? '--' : Number(value).toFixed(1)
    },
    loadData() {
      getAction(this.url.list).then(res => {
        if (res.code === 200) {
          this.dataSource = res.data
        }
      })
    },
  },
}
</script>

<style lang="less" scoped>
.PositionTemHum {
  .title {
    width: 100%;
    text-align: center;
    color: #00f6ff;
    font-size: 0.1rem;
    padding: 0.052rem 0;
    background: url('./image/bottom_line.png') bottom no-repeat;
  }

  .content {
    width: 100%;
    height: calc(100% - 0.252rem);
    padding: 0.1rem;

    .table {
      width: 100%;
      height: 100%;
      font-size: 0.1rem;

      .list {
        width: 100%;

        td {
          width: 30%;
          text-align: center;

          &:first-child {
            width: 40%;
            text-align: right;
          }

        }
      }

    }
  }
}
</style>