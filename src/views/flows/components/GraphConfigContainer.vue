<template>
  <div id="header">
    <a-tooltip title="放大">
      <span class="iconfont header-iconfont" @click="()=>{this.graph.zoom(0.2)}">&#xe898;</span>
    </a-tooltip>
    <a-tooltip title="缩小">
      <span class="iconfont header-iconfont" @click="()=>{this.graph.zoom(-0.2)}">&#xe897;</span>
    </a-tooltip>
    <a-tooltip title="撤销">
      <span class="iconfont header-iconfont" @click="()=>{this.graph.history.undo()}">&#xe739;</span>
    </a-tooltip>
    <a-tooltip title="取消撤销">
      <span class="iconfont header-iconfont" @click="()=>{this.graph.history.redo()}">&#xe652;</span>
    </a-tooltip>
    <a-tooltip title="全屏">
      <span class="iconfont header-iconfont" @click="()=>{this.$emit('full',true)}">&#xec13;</span>
    </a-tooltip>
    <a-tooltip title="退出全屏">
      <span class="iconfont header-iconfont" @click="()=>{this.$emit('full',false)}">&#xe7d6;</span>
    </a-tooltip>
    <a-tooltip title="导出svg">
      <span class="iconfont header-iconfont" @click="saveToSvg">&#xe64a;</span>
    </a-tooltip>
    <a-tooltip title="ooo">
      <span class="iconfont header-iconfont" @click="ooo">&#xe64a;</span>
    </a-tooltip>
  </div>
</template>

<script>
import { Graph, DataUri } from '@antv/x6'
export default {
  name: 'GraphConfigContainer',
  props: {
    graph: Graph
  },
  methods: {
    ooo () {
      console.log(this.graph.toJSON())

      const jsonStr = JSON.stringify(this.graph.toJSON())
      console.log(jsonStr)
      const blob = new Blob([jsonStr], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.download = 'data.json';
      link.href = url;
      document.body.appendChild(link);
      link.click();
      URL.revokeObjectURL(url);

    },

    // 保存成svg图片
    saveToSvg () {
      this.graph.toSVG(dataUri => {
        // 下载
        DataUri.downloadDataUri(DataUri.svgToDataUrl(dataUri), 'chart.svg')
        // }, {
        //   preserveDimensions: {
        //     width: 800,
        //     height: 800
        //   }
        // })
      })
    }
  }
}
</script>

<style scoped>
#header {
  display: flex;
  border-top: 1px solid #dfe3e8;
  padding-top: 5px;
  margin:0;
  height: 35px;
  width: 100%;
}
.header-iconfont{
  margin-left: 20px;
  cursor: pointer;
  color: #595959;
}
</style>
