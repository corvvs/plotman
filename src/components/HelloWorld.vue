<template lang="pug">
  .hello.container
    .row
      .section.col-md-3
        .netdef
          textarea.network_text(v-model="network_text")
        .panel
          button.btn.btn-info(@click="parse_network") Parse
      .section.col-md-9
        Network.network(
          ref="network" :nodes="network.nodes" :edges="network.edges" :options="network.options"
          @nodes-update="networkEvent('nodes-update')" @edges-update="networkEvent('edges-update')"
          )
</template>

<script>
import { Network } from 'vue2vis';
import _ from 'underscore';

export default {
  name: 'HelloWorld',
  components: {
    Network,
  },
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      networkEvents: '',
      network: {
        nodes: [
          { id: 1, label: 'Node 1' },
          { id: 2, label: 'Node 2' },
          { id: 3, label: 'Node 3' },
          { id: 4, label: 'Node 4' },
          { id: 5, label: 'Node 5' },
        ],
        edges: [
          { id: 1, from: 1, to: 3, label: "1", value: 1, },
          { id: 2, from: 1, to: 2, label: "1", value: 1, },
          { id: 3, from: 2, to: 4, label: "1", value: 1, },
          { id: 4, from: 2, to: 5, label: "1", value: 1, },
          { id: 5, from: 3, to: 3, label: "1", value: 1, },
        ],
        options: {
          nodes: {
            shape: 'circle',
          },
        },      
      },

      network_text: "",
    }
  },

  computed: {
    r: function () {
      return this.network_text
    }
  },

  methods: {
    networkEvent(eventName) {
      if (this.networkEvents.length > 500) this.networkEvents = '';
      this.networkEvents += `${eventName}, `;
    },
    
    stringify_network: function() {
      const ns = {};
      const es = _(this.network.edges).map(e => {
        ns[e.from] = 1; ns[e.to] = 1
        return `${e.from} ${e.to} ${e.value || ""}`
      })
      const nss = Object.keys(ns).sort()
      return _([
        `${nss.length} ${es.length}`,
        es,
      ]).flatten().join("\n")
    },

    parse_network: function () {
      const text = this.network_text
      if (!text) { return }
      try {
        console.log(text)
        const lines = text.split("\n")
        const [N,M] = lines.shift().split(/ +/).map(i => parseInt(i))
        const nh = {}
        const edges = _(M).chain().range().map(() => lines.shift()).map((line,i) => {
          const [from, to, value] =  line.split(/ +/)
          nh[from] = 1; nh[to] = 1
          return { id: i+1, from, to, value: 1, width: 0.1, label: value, arrows: "to", smooth: true }
        }).value()
        const ns = Object.keys(nh)
        const nodes = ns.map(i => ({ id: i, label: i }))
        this.$set(this.network, "nodes", nodes)
        this.$set(this.network, "edges", edges)
        console.log(nodes, edges)
      } catch(e) {
        console.error(e)
        return
      }

    }
  },

  mounted: function () {
    this.network_text = this.stringify_network()
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import 'vue2vis/dist/vue2vis.css';
* {
  font-family: sans-serif;
}
.wrapper {
  padding: 20px 50px;
  text-align: center;
}
.events {
  text-align: left;
  height: 70px;
}
.network {
  height: 400px;
  border: 1px solid #ccc;
  margin: 5px 0;
}
.netdef {
  height: 100%;
  textarea.network_text {
    height: 100%;
    width: 100%;
  }
}
</style>
