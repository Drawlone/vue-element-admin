<template>
  <div>
    <div ref="container" />
  </div>
</template>

<script>
import G6 from '@antv/g6'
import axios from 'axios'

export default {
  name: 'Graph',
  data() {
    return {
      nodes: null,
      edges: null
    }
  },
  mounted() {
    const container = this.$refs.container

    const tooltip = new G6.Tooltip({
      offsetX: 10,
      offsetY: 10,
      fixToNode: [1, 0.5],
      // the types of items that allow the tooltip show up
      // 允许出现 tooltip 的 item 类型
      itemTypes: ['node', 'edge'],
      // custom the tooltip's content
      // 自定义 tooltip 内容
      getContent: (e) => {
        const outDiv = document.createElement('div')
        outDiv.style.width = 'fit-content'
        outDiv.style.height = 'fit-content'
        const model = e.item.getModel()

        if (e.item.getType() === 'node') {
          outDiv.innerHTML = `${model.companyName
          }<br/>公司详情：<br/>公司法人：`
        } else {
          const source = e.item.getSource()
          const target = e.item.getTarget()
          outDiv.innerHTML = `来源：${source.getModel().name}<br/>去向：${target.getModel().name
          }`
        }
        return outDiv
      }
    })

    const graph = new G6.Graph({
      container,
      width: container.scrollWidth,
      height: container.scrollHeight || 500,
      // ...其他配置
      defaultEdge: {
        style: {
          endArrow: true
        }
      },
      layout: {
        type: 'fruchterman',
        gpuEnabled: true,
        maxIteration: 300,
        preventOverlap: true
      },
      animate: true,
      modes: {
        default: ['drag-node', 'activate-relations']
      },
      plugins: [tooltip]
    })

    // 获取节点数据
    const nodesPromise = axios.get('https://mock.apifox.cn/m1/3340473-0-default/companyInfo')
      .then((response) => response.data.data)

    // 获取边的关系数据
    const edgesPromise = axios.get('https://mock.apifox.cn/m1/3340473-0-default/getCompanyRelation')
      .then((response) => response.data.data)
    // 合并节点和边的关系数据
    Promise.all([nodesPromise, edgesPromise])
      .then(([nodeData, edgeData]) => {
        this.nodes = nodeData
        this.edges = edgeData
        const data = {
          nodes: this.nodes,
          edges: this.edges
        }
        console.log(data)
        // 渲染图
        graph.data(data)
        graph.render()
      })
    if (typeof window !== 'undefined') {
      window.onresize = () => {
        if (!graph || graph.get('destroyed')) return
        if (!container || !container.scrollWidth || !container.scrollHeight) { return }
        graph.changeSize(container.scrollWidth, container.scrollHeight)
      }
    }
  }
}
</script>

<style scoped>
/* ...样式 */
</style>
