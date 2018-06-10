<template>
  <div class="bigdatalist">
    <h1 class="bigdatalist-title">{{ msg }}</h1>
    <main class="bigdatalist-main" ref="datalistContent">
      <div class="bigdatalist-datalist" ref="dataListBox" ondragstart="return false">
        <div class="bigdatalist-dataItem" v-for="(dataItem,index) in activeData"
             :style="{'height':dataItem.height+'px','backgroundColor':dataItem.backGroundColor}">
          <span>
            {{dataItem.serialNum}}
          </span>
          <span>
            {{dataItem.value}}
          </span>
        </div>

      </div>
      <div class="bigdatalist-scroll" ref="scrollRail" ondragstart="return false">
        <div class="bigdatalist-scroll-bar" ref="scrollBar">

        </div>
      </div>

    </main>

  </div>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data() {
      return {
        msg: '加载大量数据列表',
        readyToDrag: false,
        scrollbarToTopRatio: 0,
        totalDataCounts: 200000,
        countsPerPage: 5,
        activeData: [],
      }
    },
    mounted: function () {
      this.refreshDataList(0);//初始化数据列表
      this.defineScrollBar();//拖动滚动条
      this.mousewheel();  //鼠标滚轮事件
    },
    methods: {
      /*初始化数据列表*/
      refreshDataList(initNum) {
        console.log("initNum-刷新序列号");
        console.log(initNum);
        let dataList = [];
        for (let key = 0; key < this.countsPerPage; key++) {
          let dataItem = {
            height: this.$refs.dataListBox.offsetHeight / this.countsPerPage,
            serialNum: key + initNum,
            value: "一组数据！",
            backGroundColor: (0 === key % 2) ? '#fff' : 'rgba(170,93,198,0.3)',
          };
          dataList.push(dataItem);
        }
        this.activeData = dataList;
        // console.log(this.activeData)
      },

      /*自定义滚动条*/
      defineScrollBar() {
        let self = this;
        let scrollBar = self.$refs.scrollBar;
        let scrollRail = self.$refs.scrollRail;
        let datalistContent = self.$refs.datalistContent;
        let scrollbarToTopRatio = 0;
        scrollBar.style.height = self.countsPerPage / self.totalDataCounts * scrollRail.offsetHeight + "px";
        /*准备拖动*/
        scrollBar.onmousedown = function () {
          self.readyToDrag = true;
          scrollBar.style.backgroundColor = "#409EFF";
        };
        /*开始拖动*/
        datalistContent.onmousemove = function (e) {
          if (self.readyToDrag) {
            let offsetY = e.clientY - datalistContent.offsetTop;
            let railH = scrollRail.offsetHeight;
            let scrollBarH = scrollBar.offsetHeight;
            let scrollBarToTop = offsetY;
            if (scrollBarToTop >= 0 && scrollBarToTop <= railH - scrollBarH) {//处于可拖动状态
              scrollbarToTopRatio = scrollBarToTop / (railH - scrollBarH);
              scrollBar.style.marginTop = scrollBarToTop + "px";
              self.scrollbarToTopRatio = scrollbarToTopRatio; //更新最终滚动条的位置；
            }
            self.loadData();//加载数据；
          }
        };
        /*点击滚动条滑轨直接定位*/
        scrollRail.onclick = function (e) {
          let scrollBarH = scrollBar.offsetHeight;
          let railH = scrollRail.offsetHeight;
          let clickPosition = e.pageY - scrollRail.offsetTop;
          if (clickPosition >= 0 && clickPosition <= railH - scrollBarH) {
            scrollBar.style.marginTop = clickPosition + "px";
            scrollbarToTopRatio = clickPosition / (railH - scrollBarH);
            self.scrollbarToTopRatio = scrollbarToTopRatio; //更新最终滚动条的位置；
          }
          else if (clickPosition > railH - scrollBarH) {
            scrollBar.style.marginTop = railH - scrollBarH + "px";
            self.scrollbarToTopRatio = 1; //更新最终滚动条的位置；
          }
          self.loadData();//加载数据；

        };
        /*停止拖动*/
        datalistContent.onmouseup = function () {
          self.readyToDrag = false;
          scrollBar.style.backgroundColor = " rgba(64, 157, 255, 0.7)";
        };
        /*停止拖动*/
        datalistContent.onmouseleave = function (e) {
          self.readyToDrag = false;
          scrollBar.style.backgroundColor = " rgba(64, 157, 255, 0.7)";
        };
        /*调整屏幕尺寸*/
        window.onresize = function () {
          // console.log("调整屏幕尺寸");
          let railH = scrollRail.offsetHeight;
          let scrollBarH = scrollBar.offsetHeight;
          scrollBar.style.marginTop = (railH - scrollBarH) * scrollbarToTopRatio + "px";
        }

      },

      /*填充数据*/
      loadData() {
        let firstDataSerNum = Math.round(this.scrollbarToTopRatio * this.totalDataCounts);
        // console.log(firstDataSerNum);
        if (firstDataSerNum <= this.totalDataCounts - this.countsPerPage) {
          this.refreshDataList(firstDataSerNum);//刷新数据
        }
        else {
          this.refreshDataList(this.totalDataCounts - this.countsPerPage);//刷新数据
        }
      },

      /*鼠标滚轮事件*/
      mousewheel() {
        let self = this;
        let datalistContent = self.$refs.datalistContent;
        datalistContent.onmousewheel = function (e) {
          let activeFirstDataNum = self.activeData[0].serialNum;
          /*向下滚动*/
          if (e.deltaY > 0 && activeFirstDataNum + 1 <= self.totalDataCounts - self.countsPerPage) {
            self.refreshDataList(activeFirstDataNum + 1);//刷新数据

          }
          else if (e.deltaY < 0 && activeFirstDataNum > 0) {
            self.refreshDataList(activeFirstDataNum - 1);//刷新数据
          }

          self.scrollbarToTopRatio=activeFirstDataNum/self.totalDataCounts;
          let scrollRail = self.$refs.scrollRail;
          let scrollBar = self.$refs.scrollBar;
          let railH = scrollRail.offsetHeight;
          let scrollBarH = scrollBar.offsetHeight;
          scrollBar.style.marginTop = (railH - scrollBarH)* self.scrollbarToTopRatio+ "px";

        }
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  $backGray01: #E4E7ED;
  $backBlue01: #D0E9FF;
  $black01: #606266;
  $blue01: #409EFF;
  $blue02: rgba(64, 157, 255, 0.7);
  $green01: rgba(103, 194, 58, 0.2);
  $purple01: rgba(170, 93, 198, 0.3);

  .bigdatalist {
    height: 100vh;
    background-color: $backGray01;
    display: flex;
    justify-content: center;
    flex-direction: column;
    align-items: center;
    .bigdatalist-title {
      height: 60px;
      line-height: 60px;
      background-color: $backBlue01;
      width: 100%;
    }
    .bigdatalist-main {
      height: calc(100vh - 100px);
      height: -webkit-calc(100vh - 100px);
      height: -moz-calc(100vh - 100px);
      background-color: #fff;
      width: 100%;
      min-height: 400px;
      .bigdatalist-datalist {
        height: 100%;
        width: calc(100% - 25px);
        float: left;
        background-color: $green01;
        .bigdatalist-dataItem {
          background-color: $purple01;
          font-size: 22px;
        }
      }
      .bigdatalist-scroll {
        height: 100%;
        width: 10px;
        border-radius: 10px;
        overflow: hidden;
        border: 1px solid $black01;
        float: right;
        margin-right: 10px;
        box-shadow: 0 0 5px inset #2d374b;
        .bigdatalist-scroll-bar {
          min-height: 40px;
          width: 10px;
          background-color: $blue02;
          border-radius: 10px;
        }
      }
    }
  }


</style>
