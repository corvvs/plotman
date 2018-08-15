<template lang="pug">
  .hello.container
    .row
      .section.col-lg-3
        .netdef
          textarea.network_text(v-model="network_text")
        .panel
          button.btn.btn-info(@click="parse_network") Parse
      .section.col-lg-9
        Network.network(
          ref="network" :nodes="network.nodes" :edges="network.edges" :options="network.options"
          @nodes-update="networkEvent('nodes-update')" @edges-update="networkEvent('edges-update')"
          )
</template>

<script>
import { Network } from 'vue2vis';
import _ from 'underscore';

const default_network_text = `8 15
1 8 1
7 3 14
8 2 13
3 5 4
5 7 5
6 4 1
6 8 17
7 8 5
1 4 2
4 7 1
6 1 3
3 1 10
2 6 5
2 4 12
5 1 30
`

const default_network = {
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
    edges: {
      arrows: { to: { enabled: true, scaleFactor: 0.5 } }, smooth: true, scaling: {},
    }
  },      
}

export default {
  name: 'HelloWorld',
  components: {
    Network,
  },
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      networkEvents: '',
      network: {},
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

        const lines = text.split("\n")
        const [N,M] = lines.shift().split(/ +/).map(i => parseInt(i))
        const nh = {}
        const scaling = { }
        const edges = _(M).chain().range().map(() => lines.shift()).map((line,i) => {
          const [from, to, value] =  line.split(/ +/)
          nh[from] = 1; nh[to] = 1
          return { from, to, label: value }
        }).value()

        const ns = Object.keys(nh)
        const nodes = ns.map(i => ({ id: i, label: i }))
        this.network = {
          nodes, edges, options: default_network.options
        }
        this.$forceUpdate()
      } catch(e) {
        console.error(e)
        return
      }

    }
  },

  mounted: function () {
    this.network_text = default_network_text
    this.parse_network()
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
  height: 100%;
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
