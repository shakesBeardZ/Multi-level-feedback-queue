<template>
  <div class="bottom">
    <div class="thirdQueue">
      <transition-group name="slide">
        <Row
          is="my-card"
          v-model="thirdQueueCards"
          v-for="msg3 in thirdQueueCardsCache"
          v-bind:message="msg3"
          v-bind:key="msg3['index']"
        >
        </Row>
      </transition-group>
    </div>
    <div class="thirdCost">
      {{ thirdQueueCost }}
    </div>
    <div class="secondQueue">
      <transition-group name="slide">
        <Row
          is="my-card"
          v-model="secondQueueCards"
          v-for="msg2 in secondQueueCardsCache"
          v-bind:message="msg2"
          v-bind:key="msg2['index']"
        >
        </Row>
      </transition-group>
    </div>
    <div class="secondCost">
      {{ secondQueueCost }}
    </div>
    <div class="firstQueue">
      <transition-group name="slide">
        <Row
          is="my-card"
          v-model="firstQueueCards"
          v-for="msg1 in firstQueueCardsCache"
          v-bind:message="msg1"
          v-bind:key="msg1['index']"
        >
        </Row>
      </transition-group>
    </div>
    <div class="firstCost">
      {{ firstQueueCost }}
    </div>
    <div class="buttonsFirstRow">
      <Button-group shape="circle" size="small">
        <Button @click.native="speedDownQueue" type="success">
          <Icon type="chevron-left"></Icon>
          减速
        </Button>
        <Button @click.native="pauseQueue" type="success">
          暂停
        </Button>
        <Button @click.native="speedUpQueue" type="success">
          加速
          <Icon type="chevron-right"></Icon>
        </Button>
      </Button-group>
    </div>
    <div class="buttonsSecondRow">
      <Button @click.native="startQueue" type="primary" size="small" shape="circle" long>
        开始
      </Button>
    </div>
    <div class="buttonsThirdRow">
      <Button @click.native="modal = true" type="error" size="small" shape="circle" long>
        图表
      </Button>
    </div>
    <Modal
      title="多级反馈处理结果"
      v-model="modal"
      :mask-closable="false">
      <Table height="230" :columns="resultColumns" :data="resultQueueCache"></Table>
    </Modal>
    <draggable class="selected" v-model="selectedCards" :options="dragOptions" :move="onMove" >
      <Row>
      </Row>
    </draggable>
    <draggable class="chooseCard" v-model="waitingCards" :options="dragOptions" :move="onMove" @start="isDragging=true" @end="isDragging=false">
      <Row
        is="my-card"
        v-for="msg in waitingCards"
        v-bind:message="msg"
        v-bind:key="msg['index']"
      >
      </Row>
    </draggable>
    <draggable class="deleted" :options="dragOptions" :move="onMove" >
    </draggable>
  </div>
</template>

<script>
  import myCard from './myCard'
  import draggable from 'vuedraggable'
  export default {
    data: function () {
      return {
        thirdQueueCards: [
        ],
        secondQueueCards: [
        ],
        firstQueueCards: [
        ],
        waitingCards: [
          {
            id: 'progress1',
            index: 1,
            totaltime: 2,
            time: 2,
            startTime: 0
          },
          {
            id: 'progress2',
            index: 2,
            totaltime: 4,
            time: 4,
            startTime: 0
          },
          {
            id: 'progress3',
            index: 3,
            totaltime: 6,
            time: 6,
            startTime: 0
          },
          {
            id: 'progress4',
            index: 4,
            totaltime: 8,
            time: 8,
            startTime: 0
          }
        ],
        selectedCards: [
        ],
        resultColumns: [
          {
            title: '进程名',
            key: 'id',
            width: 120
          },
          {
            title: '需要时间',
            key: 'totaltime',
            width: 120
          },
          {
            title: '周转时间',
            key: 'startTime',
            width: 120
          },
          {
            title: '带权周转时间',
            key: 'weightStartTime',
            width: 120
          }
        ],
        resultQueueCards: [
        ],
        modal: false,
        index: 4,
        isCaculating: false,
        caculateApp: null,
        QueueRuntime: 0,
        firstQueueCost: 2,
        secondQueueCost: 4,
        thirdQueueCost: 6,
        isDragging: false,
        delayedDragging: false
      }
    },
    methods: {
      onMove ({relatedContext, draggedContext}) {
        const relatedElement = relatedContext.element
        const draggedElement = draggedContext.element
        return (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
      },
      caculateQueue () {
        if (this.firstQueueCards.length !== 0) {
          this.QueueRuntime += this.firstQueueCost
          this.firstQueueCards[0]['time'] -= this.firstQueueCost
          if (this.firstQueueCards[0]['time'] > 0) {
            this.secondQueueCards.push(this.firstQueueCards.shift())
          } else {
            this.QueueRuntime += this.firstQueueCards[0]['time']
            this.firstQueueCards[0]['startTime'] = this.QueueRuntime - this.firstQueueCards[0]['startTime']
            this.resultQueueCards.push(this.firstQueueCards.shift())
          }
        } else if (this.secondQueueCards.length !== 0) {
          this.QueueRuntime += this.secondQueueCost
          this.secondQueueCards[0]['time'] -= this.secondQueueCost
          if (this.secondQueueCards[0]['time'] > 0) {
            this.thirdQueueCards.push(this.secondQueueCards.shift())
          } else {
            this.QueueRuntime += this.secondQueueCards[0]['time']
            this.secondQueueCards[0]['startTime'] = this.QueueRuntime - this.secondQueueCards[0]['startTime']
            this.resultQueueCards.push(this.secondQueueCards.shift())
          }
        } else if (this.thirdQueueCards.length !== 0) {
          this.QueueRuntime += this.thirdQueueCost
          this.thirdQueueCards[0]['time'] -= this.thirdQueueCost
          if (this.thirdQueueCards[0]['time'] > 0) {
            this.thirdQueueCards.push(this.thirdQueueCards.shift())
          } else {
            this.QueueRuntime += this.thirdQueueCards[0]['time']
            this.thirdQueueCards[0]['startTime'] = this.QueueRuntime - this.thirdQueueCards[0]['startTime']
            this.resultQueueCards.push(this.thirdQueueCards.shift())
          }
        } else {
          this.pauseQueue()
        }
      },
      startQueue () {
        if (this.firstQueueCards.length === 0 && this.secondQueueCards.length === 0 && this.thirdQueueCards.length === 0) {
          alert('请将至少一张进程卡片加入队列')
        } else if (this.caculateApp == null) {
          this.caculateApp = setInterval(this.caculateQueue, 1000)
        }
      },
      pauseQueue () {
        clearInterval(this.caculateApp)
        this.caculateApp = null
      },
      speedUpQueue () {
        clearInterval(this.caculateApp)
        this.caculateApp = setInterval(this.caculateQueue, 500)
      },
      speedDownQueue () {
        clearInterval(this.caculateApp)
        this.caculateApp = setInterval(this.caculateQueue, 2000)
      }
    },
    computed: {
      dragOptions () {
        return {
          animation: 0,
          group: 'description',
          ghostClass: 'ghost'
        }
      },
      firstQueueCardsCache: function () {
        var firstCache = this.firstQueueCards.slice(0, 3)
        if (this.firstQueueCards.length >= 4) {
          var firstCacheTime = 0
          var firstCacheTotalTime = 0
          for (var i = 3; i < this.firstQueueCards.length; i++) {
            firstCacheTotalTime += this.firstQueueCards[i]['totaltime']
            firstCacheTime += this.firstQueueCards[i]['time']
          }
          firstCache.push({id: 'MORE', index: this.firstQueueCards[3]['index'], totaltime: firstCacheTotalTime, time: firstCacheTime})
        }
        return firstCache
      },
      secondQueueCardsCache: function () {
        var secondCache = this.secondQueueCards.slice(0, 3)
        if (this.secondQueueCards.length >= 4) {
          var secondCacheTime = 0
          var secondCacheTotalTime = 0
          for (var i = 3; i < this.secondQueueCards.length; i++) {
            secondCacheTotalTime += this.secondQueueCards[i]['totaltime']
            secondCacheTime += this.secondQueueCards[i]['time']
          }
          secondCache.push({id: 'MORE', index: this.secondQueueCards[3]['index'], totaltime: secondCacheTotalTime, time: secondCacheTime})
        }
        return secondCache
      },
      thirdQueueCardsCache: function () {
        var thirdCache = this.thirdQueueCards.slice(0, 3)
        if (this.thirdQueueCards.length >= 4) {
          var thirdCacheTime = 0
          var thirdCacheTotalTime = 0
          for (var i = 3; i < this.thirdQueueCards.length; i++) {
            thirdCacheTotalTime += this.thirdQueueCards[i]['totaltime']
            thirdCacheTime += this.thirdQueueCards[i]['time']
          }
          thirdCache.push({id: 'MORE', index: this.thirdQueueCards[3]['index'], totaltime: thirdCacheTotalTime, time: thirdCacheTime})
        }
        return thirdCache
      },
      resultQueueCache: function () {
        var resultCache = []
        for (var i = 0; i < this.resultQueueCards.length; i++) {
          resultCache.push({id: this.resultQueueCards[i]['id'], totaltime: this.resultQueueCards[i]['totaltime'], startTime: this.resultQueueCards[i]['startTime'], weightStartTime: (this.resultQueueCards[i]['startTime'] / this.resultQueueCards[i]['totaltime']).toFixed(2)})
        }
        return resultCache
      }
    },
    watch: {
      isDragging (newValue) {
        if (newValue) {
          this.delayedDragging = true
          return
        }
        this.$nextTick(() => {
          this.delayedDragging = false
        })
      },
      selectedCards () {
        if (this.selectedCards.length >= 1) {
          this.selectedCards[0]['startTime'] += this.QueueRuntime
          this.firstQueueCards.push(this.selectedCards.shift())
        }
      },
      waitingCards () {
        if (this.waitingCards.length < 4) {
          var randomProcessId = 'process' + Math.ceil(Math.random() * 20)
          var randomProcessTime = Math.ceil(Math.random() * 20)
          ++(this.index)
          this.waitingCards.push({id: randomProcessId, index: this.index, totaltime: randomProcessTime, time: randomProcessTime, startTime: 0})
        }
      }
    },
    components: {
      myCard,
      draggable
    },
    name: 'main-desk'
  }
</script>

<style scoped>
  .bottom{
    width: 997px;
    height: 617px;
    background: url(../../../dist/background.png);
    background-size: cover;
  }
  .thirdCost{
    left: 700px;
    top: 90px;
  }
  .secondCost{ 
    left: 700px;
    top: 240px;
  }
  .firstCost{
    left: 700px;
    top: 400px;
  }
  .thirdCost,.secondCost,.firstCost{
    position: fixed;
    font-size: 50px;
    opacity: 0.5;
    color: #ffffff;
  }
  .thirdQueue{
    position: fixed;
    width: 480px;
    left: 160px;
    top: 80px;
  }
  .secondQueue{
    position: fixed;
    width: 480px;
    left: 160px;
    top: 230px;
  }
  .firstQueue{
    position: fixed;
    width: 480px;
    left: 160px;
    top: 390px;
  }
  .buttonsFirstRow{
    position: fixed;
    width: 160px;
    left: 815px;
    top: 387px;
  }
  .buttonsSecondRow{
    position: fixed;
    width: 140px;
    left: 815px;
    top: 422px;
  }
  .buttonsThirdRow{
    position: fixed;
    width: 140px;
    left: 815px;
    top: 455px;
  }
  .chooseCard{
    position: fixed;
    width: 400px;
    left: 230px;
    top: 515px;
  }
  .selected{
    position: fixed;
    height: 100px;
    width: 70px;
    left: 130px;
    top: 515px;
  }
  .deleted{
    position: fixed;
    height: 100px;
    width: 70px;
    left: 810px;
    top: 515px;
  }
  .slide-enter-active {
    transition: all .5s ease-in;
    transition-delay: .5s;
  }
  .slide-leave-active {
    transform: translateX(-20px);
    transition: all .5s ease-out;
    opacity: 0;
  }
  .slide-enter{
    transform: translateX(10px);
    opacity: 0;
  }
  .slide-leave{
    opacity: 1;
  }
  .ghost{
    opacity: 0.5;
  }
</style>
