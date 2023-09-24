<template>
  <div>
    <div ref="container" />
  </div>
</template>

<script>
import G6 from '@antv/g6'

export default {
  mounted() {
    const container = this.$refs.container
    const graph = new G6.Graph({
      container,
      width: 800,
      height: 600
      // ...其他配置
    })

    // 获取节点数据
    const nodePromise = fetch('https://your-backend-server.com/node-data')
      .then((response) => response.json())

    // 获取边的关系数据
    const edgePromise = fetch('https://your-backend-server.com/edge-data')
      .then((response) => response.json())

    // 合并节点和边的关系数据
    Promise.all([nodePromise, edgePromise])
      .then(([nodeData, edgeData]) => {
        const data = {
          nodes: nodeData,
          edges: edgeData
        }

        // 渲染图
        graph.data(data)
        graph.render()
      })
  }
}
</script>

  <style scoped>
  /* ...样式 */
  </style>
