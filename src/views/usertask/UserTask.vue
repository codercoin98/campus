<template>
  <div id="usertask" class="usertask">
    <tabs
      v-model="active"
      type="line"
      animated
      line-width="12vw"
      color="#ffd300"
      title-active-color="#ffd300"
      class="tabs"
    >
      <tab title="发布的任务" class="tab">
        <!--下拉菜单-->
        <dropdown-menu active-color="#ffd300">
          <dropdown-item v-model="type" :options="types" @change="changeType" get-container="#app" />
          <dropdown-item
            v-model="status"
            :options="status_options"
            @change="changeStatus"
            get-container="#app"
          />
        </dropdown-menu>
        <!--显示列表-->
        <list
          v-model="ReleaseTaskListLoading"
          :error.sync="release_error"
          error-text="请求失败，点击重新加载"
          :finished="ReleaseTaskListFinished"
          finished-text="没有更多了"
          :offset="300"
          class="releaseTaskList"
          :immediate-check="false"
          @load="getTaskList"
        >
          <div
            v-for="(item,index) in ReleaseTaskList"
            :key="index"
            class="taskItem"
            @click="$router.push({path:'/task/'+item.tid})"
          >
            <van-row type="flex" justify="space-between" align="center" class="itemHeader">
              <van-col span="4" class="serial_number">{{index+1}}</van-col>
              <van-col span="12">
                <p class="task_title">{{item.title}}</p>
              </van-col>
              <van-col span="4">
                <tag v-if="item.type === '代取快递'" color="#7232dd" plain>代取快递</tag>
                <tag v-else-if="item.type === '代打印'" color="#7232dd" plain>代打印</tag>
                <tag v-else-if="item.type === '代购物'" color="#7232dd" plain>代购物</tag>
                <tag v-else color="#7232dd" plain>其他代跑</tag>
              </van-col>
              <van-col span="4">
                <tag v-if="item.status === 1" color="#7232dd">闲置中</tag>
                <tag v-else-if="item.status === 2" type="primary">进行中</tag>
                <tag v-else-if="item.status === 3" type="success">已完成</tag>
                <tag v-else-if="item.status === 4" type="warning">已过期</tag>
                <tag v-else type="danger">已超时</tag>
              </van-col>
            </van-row>
            <van-row class="itemBody">
              <van-row class="countDown">
                <van-col span="12">
                  <count-down
                    :time="new Date(item.expiration_time) - new Date()"
                    v-if="item.status !== 3"
                  >
                    <template #default="timeData">
                      <span class="block">{{ timeData.days }}</span>
                      <span class="colon">天</span>
                      <span class="block">{{ timeData.hours }}</span>
                      <span class="colon">小时</span>
                      <span class="block">{{ timeData.minutes }}</span>
                      <span class="colon">分</span>
                    </template>
                  </count-down>
                  <div class="tips" v-else>任务已完成，快去评价吧>>></div>
                </van-col>
                <van-col span="6" v-if="type === 1 || type === 2">
                  <van-row type="flex" justify="space-between" align="center">
                    <icon color="#ffd300" size="5.5vw" name="gold-coin-o" />
                    <p style="font-size:4vw">{{item.estimated_amount}}元</p>
                  </van-row>
                </van-col>
                <van-col span="6" :offset="type !==1 && type !==2 ?6:0">
                  <van-row type="flex" align="center">
                    <icon color="#ffd300" size="5.5vw" name="balance-o" />
                    <p style="font-size:4vw">{{item.commission}}元</p>
                  </van-row>
                </van-col>
              </van-row>
            </van-row>
          </div>
        </list>
      </tab>
      <!--接取任务列表-->
      <tab title="接取的任务">
        <!--下拉菜单-->
        <dropdown-menu active-color="#ffd300">
          <dropdown-item v-model="type" :options="types" @change="changeType" get-container="#app" />
          <dropdown-item
            v-model="status"
            :options="receive_status_options"
            @change="changeStatus"
            get-container="#app"
          />
        </dropdown-menu>
        <!--列表-->
        <list
          v-model="ReceiveTaskListLoading"
          :error.sync="receive_error"
          error-text="请求失败，点击重新加载"
          :finished="ReceiveTaskListFinished"
          finished-text="没有更多了"
          :offset="300"
          :immediate-check="false"
          @load="getTaskList"
        >
          <div
            v-for="(item,index) in ReceiveTaskList"
            :key="index"
            class="taskItem"
            @click="$router.push({path:'/task/taskProcess/'+item.tid})"
          >
            <van-row type="flex" justify="space-between" align="center" class="itemHeader">
              <van-col span="4" class="serial_number">{{index+1}}</van-col>
              <van-col span="12">
                <p class="task_title">{{item.title}}</p>
              </van-col>
              <van-col span="4">
                <tag v-if="item.type === '代取快递'" color="#7232dd" plain>代取快递</tag>
                <tag v-else-if="item.type === '代打印'" color="#7232dd" plain>代打印</tag>
                <tag v-else-if="item.type === '代购物'" color="#7232dd" plain>代购物</tag>
                <tag v-else color="#7232dd" plain>其他</tag>
              </van-col>
              <van-col span="4">
                <tag v-if="item.status === 2" type="primary">进行中</tag>
                <tag v-else-if="item.status === 3" type="success">已完成</tag>
                <tag v-else type="danger">已过期</tag>
              </van-col>
            </van-row>
            <van-row class="itemBody">
              <van-row class="countDown">
                <van-col span="12">
                  <count-down
                    :time="new Date(item.expiration_time) - new Date()"
                    v-if="item.status !== 3"
                  >
                    <template #default="timeData">
                      <span class="block">{{ timeData.days }}</span>
                      <span class="colon">天</span>
                      <span class="block">{{ timeData.hours }}</span>
                      <span class="colon">小时</span>
                      <span class="block">{{ timeData.minutes }}</span>
                      <span class="colon">分</span>
                    </template>
                  </count-down>
                  <div class="tips" v-else>任务完成，很棒哦</div>
                </van-col>
                <van-col span="6" v-if="type === 1 || type === 2">
                  <van-row type="flex" justify="space-between" align="center">
                    <icon color="#ffd300" size="5.5vw" name="gold-coin-o" />
                    <p style="font-size:4vw">{{item.estimated_amount}}元</p>
                  </van-row>
                </van-col>
                <van-col span="6" :offset="type !==1 && type !==2 ?6:0">
                  <van-row type="flex" align="center">
                    <icon color="#ffd300" size="5.5vw" name="balance-o" />
                    <p style="font-size:4vw">{{item.commission}}元</p>
                  </van-row>
                </van-col>
              </van-row>
            </van-row>
          </div>
        </list>
      </tab>
    </tabs>
    <tabbar></tabbar>
  </div>
</template>

<script>
const Tabbar = () => import('components/common/tabbar/Tabbar')
import {
  Icon,
  Row as VanRow,
  Col as VanCol,
  Tab,
  Tabs,
  DropdownMenu,
  DropdownItem,
  List,
  Tag,
  CountDown,
  Toast
} from 'vant'
import userRequest from 'network/http'
export default {
  name: 'UserTask',
  data() {
    return {
      active: 0,
      type: 0,
      status: 0,
      types: [
        { text: '代取快递', value: 0 },
        { text: '代打印', value: 1 },
        { text: '代购物', value: 2 },
        { text: '其他代跑', value: 3 }
      ],
      status_options: [
        { text: '闲置', value: 0 },
        { text: '进行中', value: 1 },
        { text: '已完成', value: 2 },
        { text: '过期', value: 3 },
        { text: '超时', value: 4 }
      ],
      receive_status_options: [
        { text: '进行中', value: 0 },
        { text: '已完成', value: 1 },
        { text: '超时', value: 2 }
      ],
      ReleaseTaskList: [],
      ReceiveTaskList: [],
      ReleaseTaskListLoading: false,
      ReleaseTaskListFinished: false,
      ReceiveTaskListLoading: false,
      ReceiveTaskListFinished: false,
      release_error: false,
      receive_error: false
    }
  },
  components: {
    Icon,
    VanRow,
    VanCol,
    Tabbar,
    Tab,
    Tabs,
    DropdownMenu,
    DropdownItem,
    List,
    Tag,
    CountDown
  },
  watch: {
    active(val) {
      if (val === 0) {
        this.type = 0
        this.status = 0
        this.ReleaseTaskList = []
        this.getTaskList()
      } else if (val === 1) {
        this.type = 0
        this.status = 0
        this.ReceiveTaskList = []
        this.getTaskList()
      }
    }
  },
  methods: {
    //获取任务列表,active代表发布或者接取，type为任务类型，status为任务状态
    getTaskList() {
      userRequest
        .get('/task/getUserTaskList', {
          params: {
            list_type: this.active,
            type: this.type,
            status: this.status,
            uid: localStorage.getItem('ID')
          }
        })
        .then(res => {
          console.log(res.data)
          if (res.data.length === 0) {
            //没有对应的任务
            Toast({
              type: 'fail',
              message: '没有对应的任务',
              duration: 500,
              onClose: () => {
                if (this.active === 0) {
                  this.ReleaseTaskListFinished = true
                } else {
                  this.ReceiveTaskListFinished = true
                }
              }
            })
            return
          } else {
            if (this.active === 0) {
              res.data.forEach(item => {
                this.ReleaseTaskList = this.ReleaseTaskList.concat(item)
                this.ReleaseTaskListLoading = false
              })
              if (this.ReleaseTaskList.length === res.data.length) {
                this.ReleaseTaskListFinished = true
              }
              return
            } else {
              res.data.forEach(item => {
                this.ReceiveTaskList = this.ReceiveTaskList.concat(item)
                this.ReceiveTaskListLoading = false
              })
              if (this.ReceiveTaskList.length === res.data.length) {
                this.ReceiveTaskListFinished = true
              }
            }
          }
        })
        .catch(err => {
          console.log(err)
          if(this.active === 0){
            this.release_error = true
          }else {
            this.receive_error = true
          }
        })
    },
    //监听用户点击下拉菜单
    changeType(value) {
      this.type = value
      this.ReleaseTaskList = []
      this.ReceiveTaskList = []
      this.getTaskList()
    },
    changeStatus(value) {
      this.status = value
      this.ReleaseTaskList = []
      this.ReceiveTaskList = []
      this.getTaskList()
    }
  },
  created() {
    this.getTaskList()
  }
}
</script >

<style scoped>
.usertask {
  height: 100%;
  padding-bottom: 12vw;
}
.tab {
  height: 100%;
}
.dropdown {
  width: 100%;
}
.taskItem {
  padding: 2vw;
  margin: 2vw;
  background-color: #fff;
  border-radius: 3vw;
}
.itemHeader {
  margin-bottom: 2vw;
}
.serial_number {
  display: inline-block;
  width: 8vw;
  height: 8vw;
  text-align: center;
  border-radius: 50%;
  border: solid 0.1vw #ffd300;
  color: #ffd300;
  font-size: 5vw;
  margin-right: 5vw;
}
.task_title {
  font-size: 4.5vw;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.itemBody {
  margin-left: 13vw;
  margin-top: 10vw;
}
.countDown {
  vertical-align: middle;
}
.tips {
  font-size: 4vw;
  color: #ffd300;
}
.colon {
  display: inline-block;
  margin: 0 0.5vw;
  color: #ee0a24;
}
.block {
  display: inline-block;
  width: 6vw;
  color: #fff;
  font-size: 1.6vw;
  text-align: center;
  border-radius: 10%;
  background-color: #ee0a24;
}
.task_type_label {
  margin-top: 2vw;
  font-size: 4vw;
}
</style>