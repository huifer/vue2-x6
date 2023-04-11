<template>
  <div :class="visible?'full-screen':''">
    <graph-config-container :graph="this.graph" @full="handleFull"></graph-config-container>
    <div id="container">
      <div id="stencil"></div>
      <div id="graph-container" ref="graph"></div>
      <div v-show="configVisibe" id="config-container">
        <node-config-container v-show="isNode" :node="curNode" @updateData="handleNode"></node-config-container>
        <edge-config-container v-show="!isNode" :edge="curEdge" @updateData="handleEdge"></edge-config-container>
        <span class="iconfont collpase-icon" @click="handleCollpaseConfig">&#xe68a;</span>
      </div>
      <span v-show="!configVisibe" class="iconfont open-icon" @click="handleOpenConfig">&#xe689;</span>
    </div>

    <a-modal v-model:visible="visibleModalLine" title="Basic Modal" @ok="mod">

      <a-radio-group v-model="modalLineParam.bol">
        <a-radio :value="1">满足</a-radio>
        <a-radio :value="0">不满足</a-radio>
      </a-radio-group>
    </a-modal>

    <a-modal v-model:visible="nodeInfoVisible" width="80%" title="Basic Modal" @ok="closeNodeInfo">

      <div>
        <Condition v-if="curNodeType=='条件'"></Condition>
        <Execute v-if="curNodeType=='执行器'"></Execute>
        <ParamMapping v-if="curNodeType=='参数'"></ParamMapping>
        <Response v-if="curNodeType=='响应'"></Response>
        <Watcher v-if="curNodeType=='观察器'"></Watcher>
      </div>
    </a-modal>
  </div>


</template>

<script>
import {Addon, Graph, Shape} from '@antv/x6'
import d from './a'
import {Sketch} from 'vue-color'
import '@antv/x6-vue-shape'
import NodeConfigContainer from './components/NodeConfigContainer'
import EdgeConfigContainer from './components/EdgeConfigContainer'
import GraphConfigContainer from './components/GraphConfigContainer'

import Condition from './nodeview/Conaditon'
import Execute from './nodeview/Execute'
import ParamMapping from './nodeview/ParamMapping'
import Response from './nodeview/Response'
import Watcher from './nodeview/Watcher'

export default {
  name: 'CreateFlow',
  components: {
    GraphConfigContainer,
    NodeConfigContainer,
    EdgeConfigContainer,
    Condition,
    Execute,
    ParamMapping,
    Response,
    Watcher,
    'sketch-picker': Sketch
  },
  data () {
    return {
      nodeInfoVisible: false,
      visible: true,
      graph: {},
      stencil: {},
      ports: {},
      curNode: {},
      curEdge: {},
      configVisibe: true,
      isNode: true,
      nodeParam: {},
      edgeParam: {},
      visibleModalLine: false,
      modalLineParam: {
        bol: 0
      },
      source: {},

      curNodeType: '',

    }
  },
  methods: {
    // 是否全屏
    handleFull (value) {
      this.visible = value
    },
    // 初始化画布
    initGraph () {
      // 初始化画布
      this.graph = new Graph({
        container: document.getElementById('graph-container'),
        grid: true,
        mousewheel: {
          enabled: true,
          zoomAtMousePosition: true,
          modifiers: 'ctrl',
          minScale: 0.5,
          maxScale: 3
        },
        connecting: {
          router: {
            name: 'manhattan',
            args: {
              padding: 1
            }
          },
          connector: {
            name: 'rounded',
            args: {
              radius: 8
            }
          },
          anchor: 'center',
          connectionPoint: 'anchor',
          allowBlank: false,
          snap: {
            radius: 20
          },
          createEdge () {
            return new Shape.Edge({
              attrs: {
                line: {
                  stroke: '#A2B1C3',
                  strokeWidth: 2,
                  targetMarker: {
                    name: 'block',
                    width: 12,
                    height: 8
                  }
                }
              },
              zIndex: 0
            })
          },
          validateConnection ({targetMagnet}) {
            return !!targetMagnet
          }
        },
        highlighting: {
          magnetAdsorbed: {
            name: 'stroke',
            args: {
              attrs: {
                fill: '#5F95FF',
                stroke: '#5F95FF'
              }
            }
          }
        },
        resizing: true,
        rotating: true,
        selecting: {
          enabled: true,
          rubberband: true,
          showNodeSelectionBox: true
        },
        snapline: true,
        keyboard: true,
        clipboard: true,
        history: true
      })
    },
    mod () {
      this.visibleModalLine = false
      const source = this.curEdge.getSourceCell()

      console.log('11111=1=1=1=1=1=1=1=', source)
      var attrs1 = source.getData()
      console.log("1111111111111",attrs1)
      // 如果开始节点是条件节点则需要做输入满足或者不满足
      if (attrs1.type == '条件' || attrs1.type == '观察器') {
        this.curEdge.removeLabelAt(0)
        this.curEdge.appendLabel({
          attrs: {
            text: {
              text: this.modalLineParam.bol == 0 ? '不满足' : '满足',
            }
          }
        })
      }
      this.visibleModalLine = false
    },
    closeNodeInfo () {
      this.curNodeType = ''
      this.nodeInfoVisible = false
    },

    // 画布绑定监听事件
    graphOnEvent () {
      // 控制连接桩显示/隐藏
      const showPorts = (ports, show) => {
        for (let i = 0, len = ports.length; i < len; i = i + 1) {
          ports[i].style.visibility = show ? 'visible' : 'hidden'
        }
      }
      this.graph.on('node:mouseenter', () => {
        const container = document.getElementById('graph-container')
        const ports = container.querySelectorAll('.x6-port-body')
        showPorts(ports, true)
      })
      this.graph.on('node:mouseleave', () => {
        const container = document.getElementById('graph-container')
        const ports = container.querySelectorAll('.x6-port-body')
        showPorts(ports, false)
      })
      // 监听点击画布的节点
      this.graph.on('node:click', ({e, x, y, node, view}) => {
        this.curNode = node
        this.isNode = true
        var data = node.getData()
        console.log('ccc', node)
        this.curNodeType = data.type
        this.nodeInfoVisible = true
      })
      // 监听画布添加节点动作
      this.graph.on('node:added', ({node, index, options}) => {
        this.curNode = node
        this.isNode = true
        // todo 监听到画布添加了node之后，调后端接口创建一个接口
        console.log('1111', node)
        // 获取节点类型
        let attrs = node.getAttrs()
        let nodeType = attrs.type
        console.log(nodeType)
        // todo: 根据不同的节点类型做不同的弹框
      })
      // 监听画布移除节点动作
      this.graph.on('node:removed', ({node, index, options}) => {
        this.curNode = node
        // console.log('node:removed')
        // console.log(node)
      })
      // 监听节点之间连接动作
      this.graph.on('edge:added', ({edge, index, options}) => {
        this.isNode = false
        this.curEdge = edge
        // todo 监听到连接线动作之后，调后端接口绑定关系
        // console.log('edge:added')
        // console.log(edge)

      })
      // 监听点击节点之间连接动作
      this.graph.on('edge:click', ({e, x, y, edge, view}) => {
        this.isNode = false
        this.curEdge.setAttrs({
            line: {
              stroke: '#A2B1C3'
            }
          }
        )
        this.curEdge = edge
        this.curEdge.setAttrs({
            line: {
              stroke: '#de1379'
            }
          }
        )

        // console.log('edge:click')
      })
      // 监听节点之间移除连接动作
      this.graph.on('edge:removed', ({edge, index, options}) => {
        // todo 监听到连接线动作之后，调后端接口绑定关系
        // console.log('edge:removed')
        // console.log(edge)
      })
      this.graph.on('edge:connected', ({isNew, edge}) => {
        if (isNew) {
          const source = edge.getSourceCell()
          const target = edge.getTarget()
          // todo 线段链接
          console.log('source', source)
          console.log('target', target)
          this.curEdge = edge
          var attrs1 = source.getData()
          // 如果开始节点是条件节点则需要做输入满足或者不满足
          if (attrs1.type == '条件' || attrs1.type == '观察器') {
            this.visibleModalLine = true
          }

          // console.log(this.graph.getCellById(target.cell))
        }
      })

    },
    // 绑定画布快捷键
    graphBindKey () {
      this.graph.bindKey(['meta+c', 'ctrl+c'], () => {
        const cells = this.graph.getSelectedCells()
        if (cells.length) {
          this.graph.copy(cells)
        }
        return false
      })
      this.graph.bindKey(['meta+x', 'ctrl+x'], () => {
        const cells = this.graph.getSelectedCells()
        if (cells.length) {
          this.graph.cut(cells)
        }
        return false
      })
      this.graph.bindKey(['meta+v', 'ctrl+v'], () => {
        if (!this.graph.isClipboardEmpty()) {
          const cells = this.graph.paste({offset: 32})
          this.graph.cleanSelection()
          this.graph.select(cells)
        }
        return false
      })
      // undo redo
      this.graph.bindKey(['meta+z', 'ctrl+z'], () => {
        if (this.graph.history.canUndo()) {
          this.graph.history.undo()
        }
        return false
      })
      this.graph.bindKey(['meta+shift+z', 'ctrl+shift+z'], () => {
        if (this.graph.history.canRedo()) {
          this.graph.history.redo()
        }
        return false
      })
      // select all
      this.graph.bindKey(['meta+a', 'ctrl+a'], () => {
        const nodes = this.graph.getNodes()
        if (nodes) {
          this.graph.select(nodes)
        }
      })
      // delete
      this.graph.bindKey('backspace', () => {
        const cells = this.graph.getSelectedCells()
        if (cells.length) {
          this.graph.removeCells(cells)
        }
      })
      // zoom
      this.graph.bindKey(['ctrl+1', 'meta+1'], () => {
        const zoom = this.graph.zoom()
        if (zoom < 1.5) {
          this.graph.zoom(0.1)
        }
      })
      this.graph.bindKey(['ctrl+2', 'meta+2'], () => {
        const zoom = this.graph.zoom()
        if (zoom > 0.5) {
          this.graph.zoom(-0.1)
        }
      })
    },
    // 初始化左侧流程控件面板
    initStencil () {
      this.stencil = new Addon.Stencil({
        title: '流程图',
        target: this.graph,
        stencilGraphWidth: 250,
        stencilGraphHeight: 320,
        collapsable: true,
        notFoundText: '暂未匹配到结果',
        search: (cell, keyword, groupName, stencil) => {
          if (keyword) {
            return cell.label.includes(keyword)
          }
          return true
        },
        groups: [
          {
            title: '基础节点',
            name: 'group1',
            graphHeight: 250,
            layoutOptions: {
              rowHeight: 70
            }
          },
        ],
        layoutOptions: {
          columns: 2,
          columnWidth: 110,
          rowHeight: 60
        }
      })
      document.getElementById('stencil').appendChild(this.stencil.container)
    },
    // 初始化链接桩
    initPorts () {
      this.ports = {
        groups: {
          top: {
            position: 'top',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          },
          right: {
            position: 'right',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          },
          bottom: {
            position: 'bottom',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          },
          left: {
            position: 'left',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          }
        },
        items: [
          {
            group: 'top'
          },
          {
            group: 'right'
          },
          {
            group: 'bottom'
          },
          {
            group: 'left'
          }
        ]
      }
    },
    // 渲染所有左侧控件图形
    loadStencil () {
      Graph.registerNode(
        'custom-rect',
        {
          inherit: 'rect',
          width: 80,
          height: 36,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626'
            }
          },
          ports: {...this.ports}
        },
        true
      )

      Graph.registerNode(
        'custom-polygon',
        {
          inherit: 'polygon',
          width: 80,
          height: 38,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626'
            }
          },
          ports: {...this.ports}
        },
        true
      )

      Graph.registerNode(
        'custom-circle',
        {
          inherit: 'circle',
          width: 48,
          height: 48,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626'
            }
          },
          ports: {...this.ports}
        },
        true
      )
      Graph.registerNode(
        'custom-cylinder',
        {
          inherit: 'cylinder',
          width: 62,
          height: 48,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626',
              refY: 32
            },
            top: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            }
          },
          ports: {...this.ports}
        },
        true
      )

      Graph.registerNode(
        'custom-text',
        {
          inherit: 'text-block',
          width: 60,
          height: 36,
          attrs: {
            body: {
              // strokeWidth: 0,
              rx: 4,
              ry: 4,
              fill: '#fff',
              stroke: '#fff'
            },
            text: {
              fontSize: 12,
              fill: '#000'
            }
          },
          ports: {...this.ports}
        },
        true
      )

      Graph.registerNode(
        'custom-path',
        {
          inherit: 'path',
          width: 60,
          height: 36,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626'
            }
          },
          ports: {...this.ports}
        },
        true
      )
      Graph.registerNode(
        'custom-image',
        {
          inherit: 'rect',
          width: 52,
          height: 52,
          markup: [
            {
              tagName: 'rect',
              selector: 'body'
            },
            {
              tagName: 'image'
            },
            {
              tagName: 'text',
              selector: 'label'
            }
          ],
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#00A4FF',
              fill: '#00A4FF'
            },
            image: {
              width: 26,
              height: 26,
              refX: 13,
              refY: 16
            },
            label: {
              refX: 3,
              refY: 2,
              textAnchor: 'left',
              textVerticalAnchor: 'top',
              fontSize: 12,
              fill: '#fff'
            }
          },
          ports: {...this.ports}
        },
        true
      )

      const r1 = this.graph.createNode({
        shape: 'custom-rect',
        label: '执行器',
        type: '执行器',
        data: {
          type: '执行器',
        },
        attrs: {
          body: {
            rx: 18,
            ry: 26
          }
        }
      })
      const r2 = this.graph.createNode({
        shape: 'custom-circle',
        label: '观察器',
        data: {
          type: '观察器',
        }, attrs: {
          type: '观察器'
        }
      })
      const r3 = this.graph.createNode({
        shape: 'custom-circle',
        label: '参数',
        type: '参数',
        data: {
          type: '参数',
        },
        attrs: {
          body: {
            fill: '#fca'
          },
        }
      })
      const r4 = this.graph.createNode({
        shape: 'custom-polygon',
        data: {
          type: '条件',
        },
        attrs: {
          body: {
            refPoints: '0,10 10,0 20,10 10,20'
          }
        },
        label: '条件'
      })
      const r8 = this.graph.createNode({
        shape: 'custom-path',
        label: '响应',
        data: {
          type: '响应',
        },
        attrs: {
          body: {
            refD: 'M 0 0 0 4 C 10 8 15 2 25 5 L 25 0 Z'
          }
        }
      })

      this.stencil.load([r1, r3, r2, r4, r8], 'group1')
    },
    // 更新node的数据
    handleNode (data) {
      this.curNode = data
    },
    // 更新edge的数据
    handleEdge (data) {
      this.curEdge = data
    },
    // 收缩配置面板
    handleCollpaseConfig () {
      this.configVisibe = false
      this.$refs.graph.style.width = 'calc(100% - 250px)'
    },
    // 展开配置面板
    handleOpenConfig () {
      this.configVisibe = true
      this.$refs.graph.style.width = 'calc(100% - 500px)'
    }
  },
  mounted () {
    // 监听键盘按键事件
    let self = this
    this.$nextTick(function () {
      document.addEventListener('keyup', function (e) {
        // 此处填写你的业务逻辑即可
        if (e.keyCode === 27) {
          self.visible = false
        }
      })
    })
    // 初始化画布
    this.initGraph()
    // 绑定画布快捷键
    this.graphBindKey()
    // 画布绑定监听事件
    this.graphOnEvent()
    // 初始化流程控件面板
    this.initStencil()
    // 初始化连接桩
    this.initPorts()
    // 渲染所有左侧控件图形
    this.loadStencil()
    // 初始化数据
    this.graph.fromJSON(d)

  },

  watch: {}
}
</script>

<style scoped>
.full-screen {
  position: fixed;
  overflow: auto;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: #fff;
  border: 0;
  border-radius: 0px;
  padding: 0;
  margin: 0;
  height: 100%;
}

#container {
  display: flex;
  border: 1px solid #dfe3e8;
  height: 100%;
  width: 100%;
}

#stencil {
  width: 250px;
  height: 100%;
  position: relative;
  border-right: 1px solid #dfe3e8;
}

#graph-container {
  width: calc(100% - 500px);
  /*width: 100%;*/
  height: 100%;
  /*background-color: #fffbe6*/
}

#config-container {
  position: relative;
  width: 250px;
  height: 100%;
  border-left: 1px solid #dfe3e8;
}

.collpase-icon {
  position: absolute;
  top: 25px;
  left: -8px;
  color: #aaa;
  background-color: #fff;
  cursor: pointer;
}

.open-icon {
  position: relative;
  top: 10px;
  right: 10px;
  color: #aaa;
  background-color: #fff;
  cursor: pointer;
}

.drawer-iconfont {
  position: absolute;
  top: 100px;
  left: 100px;
}

.color-container {
  width: 24px;
  height: 24px;
  padding: 4px;
  border: 1px solid #dfe3e8;
  border-radius: 2px;
}

.edit-color {
  cursor: pointer;
  height: 100%;
}

/deep/ .x6-widget-stencil {
  background-color: #fff;
}

/deep/ .x6-widget-stencil-title {
  background-color: rgba(0, 0, 0, .03)
}

/deep/ .x6-widget-stencil-group-title {
  background-color: rgba(0, 0, 0, .03)
}

.x6-widget-transform {
  margin: -1px 0 0 -1px;
  padding: 0px;
  border: 1px solid #239edd;
}

.x6-widget-transform > div {
  border: 1px solid #239edd;
}

.x6-widget-transform > div:hover {
  background-color: #3dafe4;
}

.x6-widget-transform-active-handle {
  background-color: #3dafe4;
}

.x6-widget-transform-resize {
  border-radius: 0;
}

.x6-widget-selection-inner {
  border: 1px solid #239edd;
}

.x6-widget-selection-box {
  opacity: 0;
}
</style>
