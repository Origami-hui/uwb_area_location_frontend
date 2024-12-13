<template>
  <div id="app">
    <div style="display: flex; align-items: center; text-align: center; justify-content: center;">
      <el-icon style="font-size: 24px"><LocationInformation /></el-icon>
      <h2 style="margin: 0">UWB行人与车辆定位系统</h2>
    </div>

    <div class="container">
      <!-- 左侧区域：配置项 -->
      
      <div class="left">
        <el-scrollbar>
          <el-form
            label-position="left"
            label-width="auto"
            style="margin-right: 20px"
            @input="setConfig"
          >
          <el-button type="primary" size="large" @click="startLocationCalculation" :disabled="isCalculating">启动定位计算</el-button>
          <el-button type="primary" size="large" @click="stopLocationCalculation" :disabled="!isCalculating">停止定位计算</el-button>
          
          <el-form-item label="定位模式" style="margin-top: 20px">
            <el-switch
              v-model="SAVE_DATA_FLAG"
              class="mb-2"
              style="--el-switch-off-color: #13ce66; "
              active-text="实时定位"
              inactive-text="使用历史数据"
              @change="setConfig"
            />
          </el-form-item>

          <el-form-item label="数据库持久化"  >
            <el-switch v-model="INSERT_DATABASE_FLAG" @change="setConfig"/>
          </el-form-item>

          <el-form-item label="记录数据表名称"  >
            <el-input v-model="W_DATA_FILE_NAME" :disabled="!configuration.SAVE_DATA_FLAG"/>
          </el-form-item>

          <el-form-item label="读取的历史数据表"  >
            <el-input v-model="R_DATA_FILE_NAME" :disabled="configuration.SAVE_DATA_FLAG"/>
          </el-form-item>

          <el-form-item label="主基站连接串口">
            <el-input-number v-model="RX_COM" @input="setConfig">
              <template #prefix>
                <span>COM</span>
              </template>
            </el-input-number>
          </el-form-item>

          <el-form-item label="每个标签的定位频率"  >
            <el-input-number v-model="LOCATION_FREQ" @input="setConfig">
              <template #suffix>
                <span>Hz</span>
              </template>
            </el-input-number>
          </el-form-item>

          <el-form-item label="车辆长"  >
            <el-input-number v-model="CAR_LENGTH" @input="setConfig">
              <template #suffix>
                <span>m</span>
              </template>
            </el-input-number>
          </el-form-item>

          <el-form-item label="车辆宽"  >
            <el-input-number v-model="CAR_WIDTH" @input="setConfig">
              <template #suffix>
                <span>m</span>
              </template>
            </el-input-number>
          </el-form-item>
          
          <el-form-item label="基站数量"  >
            <el-input-number v-model="RX_NUM" @input="setConfig"/>
          </el-form-item>
          
          <el-form-item label="标签数量"  >
            <el-input-number v-model="TX_NUM" @input="setConfig"/>
          </el-form-item>

          <el-form-item label="车辆标签起始编号"  >
            <el-input-number v-model="TX_STR_NUM" @input="setConfig"/>
          </el-form-item>

          <el-form-item label="存在人员标签"  >
            <el-switch v-model="HAVE_HUM" @change="setConfig"/>
          </el-form-item>

          <el-form-item label="人员数量"  >
            <el-input-number v-model="HUM_NUM" @input="setConfig"/>
          </el-form-item>
          
          <el-form-item label="构建与校正区域"  >
            <el-switch v-model="BUILD_RECT_FLAG" @change="setConfig"/>
          </el-form-item>

          <el-form-item label="启用NLoS误差补偿"  >
            <el-switch v-model="NLOS_FIX_FLAG" @change="setConfig"/>
          </el-form-item>

          <el-form-item label="安全阈值距离"  >
            <el-input-number v-model="THRES_DIS" @input="setConfig">
              <template #suffix>
                <span>m</span>
              </template>
            </el-input-number>
          </el-form-item>

          <el-form-item label="开启卡尔曼滤波"  >
            <el-switch v-model="FILTER_FLAG" @change="setConfig"/>
          </el-form-item>

          <el-form-item label="开启转向平滑矫正"  >
            <el-switch v-model="TOWARDS_COR_FLAG" @change="setConfig"/>
          </el-form-item>

          <el-form-item label="RX1坐标"  >
            <el-input v-model="rx1" />
          </el-form-item>
          <el-form-item label="RX2坐标"  >
            <el-input v-model="rx2" />
          </el-form-item>
          <el-form-item label="RX3坐标"  >
            <el-input v-model="rx3" />
          </el-form-item>

          <el-button size="large" type="success" @click="setConfig">保存配置</el-button>
          </el-form>
        </el-scrollbar>
      </div>
      

      <!-- 中间区域：散点图 -->
      <div class="middle">
        <div id="scatterChart"></div>
      </div>

      <!-- 右侧区域：输出数据 -->
      <div class="right">
         <div class="card-container">
          <el-card style="width: 300px">
            <template #header>
              <div class="card-header">
                <span><el-text tag="b">当前状态</el-text></span>
              </div>
            </template>
            <p class="text item">
            <el-text>{{ statusMessage }}</el-text>
            </p>
          </el-card>

          <el-card style="width: 300px">
            <template #header>
              <div class="card-header">
                <el-text tag="b">实时输出</el-text>
              </div>
            </template>
            <p class="text item">
              <el-table v-if="outputMessage" :data="outputMessageData" stripe style="width: 100%">
                <el-table-column prop="key" label="字段" />
                <el-table-column prop="value" label="值" />
              </el-table>
              <el-empty v-else description="无输出" :image-size="50"/></p>
          </el-card>
        </div>
      </div>
    </div>
    
  </div>
</template>

<script>
import * as echarts from 'echarts';
import axios from 'axios';


export default {
  name: 'location-system',
  data() {
    return {
      statusMessage: '',
      outputMessage: '',
      isCalculating: false, // 标识是否正在计算
      coordinates: null,  // 用来存储接收到的坐标数据
      outputMessageData: null,
      configuration:{
        LOCATION_FREQ: 2,
        CAR_WIDTH: 1.5,
        CAR_LENGTH: 3.0,
        RX_COM: 4,
        RX_NUM: 3,
        TX_NUM: 1,
        TX_STR_NUM: 5,
        HAVE_HUM: true,
        HUM_NUM: 1,
        SAVE_DATA_FLAG: false,
        W_DATA_FILE_NAME: "data/data_nlos_imu_1204-7.xls",
        R_DATA_FILE_NAME: "data/data_nlos_imu_1204-5.xls",
        NLOS_DATA_NAME: "nlos dataset/nlos case1204-2.csv",
        IN_NLOS_FLAG: true,
        NLOS_MODEL_NAME: "random_forest_model_test.starry",
        BUILD_RECT_FLAG: true,
        NLOS_FIX_FLAG: false,
        THRES_DIS: 2,
        WARN_MOD_FLAG: false,
        FILTER_FLAG: true,
        TOWARDS_COR_FLAG: true,
        CAR_TX_RENDER_FLAG: true,
        TDOA_FLAG: false,
        INSERT_DATABASE_FLAG: true,
        rx1: [7.4, 17.9, 0],
        rx2: [0, 0, 0],
        rx3: [15, 0, 0]
      }
    };
  },

  computed: {
    RX_NUM: {
      get() { return this.configuration.RX_NUM; },
      set(value) { this.configuration.RX_NUM = value; }
    },
    RX_COM: {
      get() { return this.configuration.RX_COM; },
      set(value) { this.configuration.RX_COM = value; }
    },

    LOCATION_FREQ: {
      get() { return this.configuration.LOCATION_FREQ; },
      set(value) { this.configuration.LOCATION_FREQ = value; }
    },
    CAR_WIDTH: {
      get() { return this.configuration.CAR_WIDTH; },
      set(value) { this.configuration.CAR_WIDTH = value; }
    },
    CAR_LENGTH: {
      get() { return this.configuration.CAR_LENGTH; },
      set(value) { this.configuration.CAR_LENGTH = value; }
    },
    TX_NUM: {
      get() { return this.configuration.TX_NUM; },
      set(value) { this.configuration.TX_NUM = value; }
    },
    TX_STR_NUM: {
      get() { return this.configuration.TX_STR_NUM; },
      set(value) { this.configuration.TX_STR_NUM = value; }
    },
    HAVE_HUM: {
      get() { return this.configuration.HAVE_HUM; },
      set(value) { this.configuration.HAVE_HUM = value; }
    },
    HUM_NUM: {
      get() { return this.configuration.HUM_NUM; },
      set(value) { this.configuration.HUM_NUM = value; }
    },
    SAVE_DATA_FLAG: {
      get() { return this.configuration.SAVE_DATA_FLAG; },
      set(value) { this.configuration.SAVE_DATA_FLAG = value; }
    },
    INSERT_DATABASE_FLAG: {
      get() { return this.configuration.INSERT_DATABASE_FLAG; },
      set(value) { this.configuration.INSERT_DATABASE_FLAG = value; }
    },
    W_DATA_FILE_NAME: {
      get() { return this.configuration.W_DATA_FILE_NAME; },
      set(value) { this.configuration.W_DATA_FILE_NAME = value; }
    },
    R_DATA_FILE_NAME: {
      get() { return this.configuration.R_DATA_FILE_NAME; },
      set(value) { this.configuration.R_DATA_FILE_NAME = value; }
    },
    NLOS_DATA_NAME: {
      get() { return this.configuration.NLOS_DATA_NAME; },
      set(value) { this.configuration.NLOS_DATA_NAME = value; }
    },
    IN_NLOS_FLAG: {
      get() { return this.configuration.IN_NLOS_FLAG; },
      set(value) { this.configuration.IN_NLOS_FLAG = value; }
    },
    BUILD_RECT_FLAG: {
      get() { return this.configuration.BUILD_RECT_FLAG; },
      set(value) { this.configuration.BUILD_RECT_FLAG = value; }
    },
    NLOS_FIX_FLAG: {
      get() { return this.configuration.NLOS_FIX_FLAG; },
      set(value) { this.configuration.NLOS_FIX_FLAG = value; }
    },
    THRES_DIS: {
      get() { return this.configuration.THRES_DIS; },
      set(value) { this.configuration.THRES_DIS = value; }
    },
    WARN_MOD_FLAG: {
      get() { return this.configuration.WARN_MOD_FLAG; },
      set(value) { this.configuration.WARN_MOD_FLAG = value; }
    },
    FILTER_FLAG: {
      get() { return this.configuration.FILTER_FLAG; },
      set(value) { this.configuration.FILTER_FLAG = value; }
    },
    TOWARDS_COR_FLAG: {
      get() { return this.configuration.TOWARDS_COR_FLAG; },
      set(value) { this.configuration.TOWARDS_COR_FLAG = value; }
    },
    CAR_TX_RENDER_FLAG: {
      get() { return this.configuration.CAR_TX_RENDER_FLAG; },
      set(value) { this.configuration.CAR_TX_RENDER_FLAG = value; }
    },
    TDOA_FLAG: {
      get() { return this.configuration.TDOA_FLAG; },
      set(value) { this.configuration.TDOA_FLAG = value; }
    },
    rx1: {
      get() { return this.configuration.rx1; },
      set(value) { this.configuration.rx1 = value.split(',').map(item => parseFloat(item.trim())).filter(item => !isNaN(item)); }
    },
    rx2: {
      get() { return this.configuration.rx2; },
      set(value) { this.configuration.rx2 = value.split(',').map(item => parseFloat(item.trim())).filter(item => !isNaN(item)); }
    },
    rx3: {
      get() { return this.configuration.rx3; },
      set(value) { this.configuration.rx3 = value.split(',').map(item => parseFloat(item.trim())).filter(item => !isNaN(item)); }
    }

  },

  methods: {
    startLocationCalculation() {
      this.statusMessage = '计算中...';
      this.isCalculating = true; // 设置为计算中

      // 使用 EventSource 来接收实时数据
      const eventSource = new EventSource('http://localhost:5000/api/start-location');

      eventSource.onmessage = (event) => {
        // 将接收到的数据更新到页面
        const data = JSON.parse(event.data); // 假设数据是 JSON 格式
        this.outputMessage = data;
        this.outputMessageData = [
          {
            key: '标签编号',
            value: data.tag_id,
          },
          {
            key: '坐标',
            value: data.tx_location,
          },
          {
            key: 'NLoS状态',
            value: data.nlos_state,
          }
        ];
        this.coordinates[data.tag_id - this.configuration.TX_STR_NUM] = [data.tx_location]; // 将新的坐标数据加入数组
        this.updateChart(data);  // 更新图表
      };

      eventSource.onerror = (error) => {
        console.error('EventSource failed:', error);
        this.statusMessage = '发生错误';
        this.isCalculating = false;
        eventSource.close();
      };
    },

    stopLocationCalculation() {
      // 发送 POST 请求到后端停止脚本
      fetch('http://localhost:5000/api/stop-location', { method: 'POST' })
        .then(response => response.json())
        .then(() => {
          this.statusMessage = '系统已停止';
          this.isCalculating = false; // 停止计算标识
          this.$message('系统已停止');
        })
        .catch(error => {
          console.error('Error stopping the process:', error);
          this.statusMessage = '停止计算失败';
        });
    },

    setConfig() {
      // 设置配置项
      axios.post('http://localhost:5000/api/set-config', this.configuration)
        .then(response => {
          // 请求成功后的处理
          console.log('Response:', response);
          this.$message('配置修改成功');
        })
        .catch(error => {
          // 请求失败后的处理
          console.error('Error:', error);
        });
    },

    initSystem(){
      //获取配置项，初始化图表
      axios.get('http://localhost:5000/api/get-config')
        .then(response => {
          // 获取到后端返回的数据并更新组件的状态
          this.updateConfiguration(response.data.message);
          this.initChart();
        })
        .catch(error => {
          console.error("There was an error:", error);
        });
    },

     // 更新图表
    updateChart(msg) {

      const re_x = msg.area_location[0];
      const re_y = msg.area_location[1];

      // 使用 ECharts 更新图表数据
      if (this.coordinates != null) {
        const chart = echarts.getInstanceByDom(document.getElementById('scatterChart'));

        //console.log(this.coordinates[4][0]);

        if (chart) {
          const nlosSum = msg.nlos_state.reduce((acc, val) => acc + val, 0); // 计算 nlos 数组的和
          const color = nlosSum > 1 ? 'red' : 'blue';
          // 更新 ECharts 数据
          chart.setOption({
            series: [{
              type: 'scatter',
              data: [ { value: [this.configuration.rx1[0], this.configuration.rx1[1]], itemStyle: { color: 'black' } },
                  { value: [this.configuration.rx2[0], this.configuration.rx2[1]], itemStyle: { color: 'black' } },
                  { value: [this.configuration.rx3[0], this.configuration.rx3[1]], itemStyle: { color: 'black' } },
                    ...this.coordinates.map(coord => ({
                        value: coord[0], // 直接取坐标值
                        itemStyle: { color: color } // 设置每个点的颜色
                      }))
                ],
              symbolSize: 10,
            },
            
            {
              type: 'custom', // 自定义绘制类型
              renderItem: function (params, api) {
                // 获取四个顶点坐标
                const points = re_x.map((x, index) => {
                  const y = re_y[index];
                  return api.coord([x, y]);  // 转换为 ECharts 坐标系
                });

                // 绘制四边形
                return {
                  type: 'polygon',
                  shape: {
                    points: points
                  },
                  style: api.style({
                    fill: 'rgba(0, 255, 0, 0.5)', // 填充颜色
                    stroke: 'green', // 边框颜色
                    lineWidth: 2 // 边框宽度
                  })
                };
              },
              data: [re_x] // 只需要一个数据项
            }
            ]
          });
        }
      }
    },

    // 初始化图表
    initChart() {
      const chart = echarts.init(document.getElementById('scatterChart'));  // 获取 ECharts 实例

      const minRange = Math.min(Math.min(this.configuration.rx1[0], this.configuration.rx2[0], this.configuration.rx3[0]), Math.min(this.configuration.rx1[1], this.configuration.rx2[1], this.configuration.rx3[1]));
      const maxRange = Math.max(Math.max(this.configuration.rx1[0], this.configuration.rx2[0], this.configuration.rx3[0]), Math.max(this.configuration.rx1[1], this.configuration.rx2[1], this.configuration.rx3[1]));

      chart.setOption({
        title: {
          text: '坐标散点图'
        },
        xAxis: {
          type: 'value',
          name: 'X坐标',
          min: minRange - 3,  // 设置固定的最小值
          max: maxRange + 3,  // 设置固定的最大值
          axisLabel: {
            formatter: '{value}'
          }
        },
        yAxis: {
          type: 'value',
          name: 'Y坐标',
          min: minRange - 3,  // 设置固定的最小值
          max: maxRange + 3,  // 设置固定的最大值
          axisLabel: {
            formatter: '{value}'
          }
        },
        series: [{
          type: 'scatter',
          data: [
            [this.configuration.rx1[0], this.configuration.rx1[1]],
            [this.configuration.rx2[0], this.configuration.rx2[1]],
            [this.configuration.rx3[0], this.configuration.rx3[1]]
          ],
          symbolSize: 10,
          itemStyle: {
            color: '#000',
          }
        }],
        animationDurationUpdate: 0,  // 禁止更新时的动画
      });
      
    },

    updateConfiguration(message) {
      Object.assign(this.configuration, message);
      this.coordinates = new Array(message.TX_NUM);
    }
  },

  // 在组件挂载时初始化图表
  mounted() {
    this.initSystem();
  },
};
</script>

<style>
#app {
  text-align: center;
  margin-top: 20px;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
}

.container {
  display: flex;
  margin-top: 20px;
  width: 100%;
  height: 90vh; /* 让容器填满整个视口 */
}

/* 左侧区域：配置项，占4份 */
.left {
  flex: 3;
  padding: 20px;
  background-color: #fff;
  box-sizing: border-box;
  border: 1px solid #ddd; /* 可选：给左侧区域添加边框 */
  font-size: 14px;
  text-align: left;
}

/* 右侧区域：散点图，占6份 */
.middle {
  flex: 6;
  display: flex;
  justify-content: center; /* 水平居中 */
  align-items: center;     /* 垂直居中 */
  border: 1px solid #ddd;
  background-color: #fff;
}

.right {
  flex: 3;
  display: flex;
  border: 1px solid #ddd;
  background-color: #fff;
}

/* 左侧配置项中的每个输入框和标签 */
.config-item {
  margin-bottom: 15px;
}

.card-container {
  margin-top: 30px;
  margin-left: 20px;
  display: flex;
  flex-direction: column; /* 设置垂直排列 */
  gap: 20px; /* 设置卡片之间的间距 */
}

/* 让 scatterChart 容器有固定的宽高 */
#scatterChart {
  width: 550px;
  height: 550px;
}
</style>
