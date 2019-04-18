<template>
  <div :class="$style['wrapper']">
    <div :class="$style['table-column']">
      <div
        ref="table-container"
        :class="$style['table-container']"
      >
        <div
          :class="$style['table']"
          ref="table"
        >
          <!-- <tbody> -->
            <div
              class="vs-buf-top"
              :style="topBufferStyles"
            >
            </div>
            <div class="cluster-container">
              <div v-for="row in currentClusterArr" :key="row" :class="$style['scroll-row']">
                <div>{{row}}</div>
                <div>C1 {{row}}</div>
                <div>C2 {{row}}</div>
              </div>
            </div>
            <div :style="bottomBufferStyles"></div>
          <!-- </tbody> -->
        </div>
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
        <div><b>Height of one Row:</b> {{ heightOfRowPx }}</b></div>
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

    const heightOfRowPx = -1;

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
        console.log(drawedBottom);
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
    const firstRow = this.$refs.table.querySelector('.cluster-container').children[0];
    if (firstRow) {
      this.heightOfRowPx = firstRow.offsetHeight;
    } else {
      this.heightOfRowPx = 0;
    }    
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
.scroll-row {
  height: 35px;
  border: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
