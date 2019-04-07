<template>
  <div :class="$style['wrapper']">
    <div :class="$style['table-column']">
      <div
        ref="table-container"
        :class="$style['table-container']"
      >
        <table
          :class="$style['table']"
          ref="table"
        >
          <tbody>
            <tr
              class="vs-buf-top"
              :style="topBufferStyles"
            >
            </tr>
            <tr v-for="row in currentClusterArr" :key="row" >
              <td>{{row}}</td>
              <td>C1 {{row}}</td>
              <td>C2 {{row}}</td>
            </tr>
            <tr :style="bottomBufferStyles"></tr>
          </tbody>
        </table>
      </div>
    </div>
    <div
      :class="$style['info-column']"
    >
      <div>
        <div><b>Count of rowsCount: </b>{{ rowsCount }}</div>
        <div><b>Scroll Position:</b> {{ scrollPosition }}</div>
        <div>
          <b>Current cluster:</b>
          {{ currentCluster }}
        </div>
        <div>
          <b>Cluster size:</b>
          {{ clusterSize }}
        </div>
        <div><b>Full Rows Height size:</b> {{ rowsCount * heightOfRowPx }}</div>
        <div><b>TopBuffer size:</b> {{ topBufferStyles.height }}</div>
        <div><b>BottomBuffer size:</b> {{ bottomBufferStyles.height }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    const rowsCount = 50000;
    const arr = [];
    for (let i=0;i<rowsCount;i++) arr.push(i+1);

    const scrollPosition = 0;

    // TODO высчитывать по вставке одной строки
    const heightOfRowPx = -1;

    // TODO расчитывать на основе высоты экрана 
    // и количества строк которые влезают в один экран
    const blockHeight = 0;
    // TODO добавить скролл на ресайз 
    return {
      rowsCount,
      arr,
      scrollPosition,
      heightOfRowPx,
      blockHeight,
    };
  },
  computed: {
    countOfClusters() {
      const { rowsCount, clusterSize } = this;
      return Math.ceil(rowsCount / clusterSize);
    },
    bufferBetweenClusters() {
      return this.clusterSize;
    },
    currentClusterArr() {
      const { arr, currentCluster, clusterSize, bufferBetweenClusters } = this;
      let start = currentCluster * clusterSize - bufferBetweenClusters;
      if (start < 0) {
        start = 0;
      }
      const end = start + clusterSize + bufferBetweenClusters;
      return arr.slice(start, end);
    },
    topBufferStyles() {
      const { clusterSize, currentCluster, heightOfRowPx, countOfClusters, bufferBetweenClusters } = this;
      let height = 0;
      if (currentCluster != 0) {
        height = clusterSize * currentCluster * heightOfRowPx - bufferBetweenClusters * heightOfRowPx;
      }      
      return {height: `${height}px`};
    },
    bottomBufferStyles() {
      const { rowsCount, clusterSize, currentCluster, heightOfRowPx, countOfClusters, bufferBetweenClusters } = this;
      let height = 0;
      if (currentCluster < countOfClusters - 1) {
        const fullHeight = rowsCount * heightOfRowPx;
        const drawedBottom = currentCluster * clusterSize * heightOfRowPx - bufferBetweenClusters * heightOfRowPx;
        height = fullHeight - drawedBottom;
      }

      return {height: `${height}px`};
    },
    currentCluster() {
      const { scrollPosition, heightOfRowPx, clusterSize } = this;
      const scrollOnRow = Math.ceil(scrollPosition / heightOfRowPx);
      const currentCluster = Math.ceil(scrollOnRow / clusterSize);
      return currentCluster;
    },
    clusterSize() {
      // On start draw 1 row for check heightOfRow in mounted hook
      if (this.heightOfRowPx == -1) return 1;
      const { blockHeight, heightOfRowPx } = this;
      const countOfRowsIntoBlock = Math.ceil(blockHeight / heightOfRowPx);
      return countOfRowsIntoBlock * 2;
    },
  },
  mounted() {
    const scrollBlock = this.$refs['table-container'];
    scrollBlock.addEventListener('scroll', this.onScroll);
    window.addEventListener('resize', this.onResize);
    this.onResize();
    this.heightOfRowPx = this.$refs.table.querySelector('.vs-buf-top').nextElementSibling.offsetHeight;
  },
  beforeDestroy() {
    const scrollBlock = this.$refs['table-container'];
    scrollBlock.removeEventListener('scroll', this.onScroll);
    window.removeEventListener('resize', this.onResize);
  },
  methods: {
    onScroll(e) {
      const { target } = e;
      const { scrollTop } = target;
      this.scrollPosition = scrollTop;
    },
    onResize() {
      this.blockHeight = this.$refs['table-container'].clientHeight;  
      console.log(this.blockHeight);
    },
  },
};
</script>

<style module>
.wrapper {
  display: flex;
  flex-direction: row;
  height: 100vh;
  padding: 10px;
  margin-left: auto;
  margin-right: auto;
}
.table-column {
  width: 60%;
  margin-right: 30px;
}
.table-container {
  overflow-y: scroll;
  height: 100%;
}
.info-column {
  padding: 20px;
}
.table {
  border: 1px solid black;
  border-collapse: collapse;
  overflow-y: scroll;
  width: 100%;
}
.table tr {
  height: 35px;
}
.table tbody {
  overflow: scroll;
}
.table td {
  border: 1px solid black;
  padding: 5px;
}
</style>
