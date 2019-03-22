<template>
  <div>
    <h1>Pod Distribution</h1>
    <div class="ns">
      <div>
        <i class="fas fa-circle"></i>Dev
      </div>
      <div>
        <i class="fas fa-circle"></i>SIT
      </div>
      <div>
        <i class="fas fa-circle"></i>UAT
      </div>
    </div>

    <div class="pods">
      <div class="env">Showing Dev. Environment</div>
      <div class="node" v-for="node in nodes.items" :key="node.metadata.uid">
        <div class="header">
          <i class="fas fa-circle"></i>{{ node.metadata.name }}
        </div>
        <div class="box" v-for="pod in podsInNode(node.metadata.name)" :key="pod.metadata.uid">
          {{pod.metadata.name}}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import nodesJson from '../assets/nodes.json'
  import podsJson from '../assets/pods.json'
  
  export default {
    data() {
      return {
        nodes: [],
        pods: [],
        errors: []
      }
    },
    
    methods: {
      getNamespace: function() {
        return this.$route.params['ns']
      },
      podsInNode(node){
        return this.pods.items.filter(m => m.spec.nodeName === node)
      },
      refreshNodes: function(url) {
        if (process.env.NODE_ENV === "development") {
          this.nodes = nodesJson
        } else {
          axios.get(url, {
              headers: {
                'Content-Type': 'application/json',
              }
            })
            .then(response => {
              // JSON responses are automatically parsed.
              this.nodes = response.data
            })
            .catch(e => {
              this.errors.push(e)
            })
        }
      },
      refreshPods: function(url) {
        if (process.env.NODE_ENV === "development") {
          this.pods = podsJson
        } else {
          axios.get(url, {
              headers: {
                'Content-Type': 'application/json',
              }
            })
            .then(response => {
              // JSON responses are automatically parsed.
              this.pods = response.data
            })
            .catch(e => {
              this.errors.push(e)
            })
        }
      }
    },
    // Fetches posts when the component is created.
    created() {
      var nodesUrl = "http://localhost:8001/api/v1/nodes"
      var self = this
      this.refreshNodes(nodesUrl)
      setInterval(function() {
        self.refreshNodes(nodesUrl)
      }, 1000);

      var podsUrl = "http://localhost:8001/api/v1/pods"
      this.refreshPods(podsUrl)
      setInterval(function() {
        self.refreshPods(podsUrl)
      }, 1000);

    }
  }
</script>

<style scoped>
div.ns > div
{
  padding: 0px 0px 10px 0px;
}
.header i,
.ns div i {
  font-size: 12px !important;
  padding-right: 5px;
  color: #76ff5d;
}
.ns {
  width: 10%;
  float: left;
  max-width: 100px;
  min-width: 50px;
}
.pods {
  float: left;
  width: 90%;
}
.box {
  margin: 2%;
  float: left;
  width: 40%;
  background-color: #5d9aff;
  padding: 10px 2px 10px 5px;
  font-size: 15px;
  border-radius: 12px;
  font-family: "arial";
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
.env {
  padding: 0px 0px 10px 15px;
}
.node {
  border: 1px #555 solid;
  float: left;
  width: 8%;
  margin-left: 15px;
  background: #222;
  min-height: 300px;
  min-width: 150px;
}
.header {
  background-color: #55657f;
  padding: 5px;
  margin-bottom: 10px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
</style>
