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
        <div v-bind:class="[role(node)==='Proxy'? proxyClass : role(node)==='Master'? masterClass: headerClass]">
          <div class="left"><i class="fas fa-circle"></i>{{ node.metadata.name }} </div>
          <div class="right">{{role(node)}}</div>
        </div>
        <div class="pod" v-for="pod in podsInNode(node.metadata.name)" :key="pod.metadata.uid">
          <div class="box" v-if="myPod(pod)">{{plainPodName(pod)}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import nodesJson from "../assets/nodes.json";
import podsJson from "../assets/pods.json";

export default {
  data() {
    return {
      proxyClass: 'proxy',
      masterClass: 'master',
      headerClass: 'header',
      nodes: [],
      pods: [],
      errors: []
    };
  },

  methods: {
    plainPodName(pod) {
      if (pod.metadata.labels) {
        var name = pod.metadata.labels.app;
        if (name) {
        } else {
          name = pod.metadata.name;
        }
      } else {
        name = pod.metadata.name;
      }
      return name;

      /* if(podName.match(/mvdls-ui/g)){
          return "mvdls-ui"
        }
          var ind = podName.indexOf("-")
          if (ind > 0){
            return podName.substring(0,ind)
          }else{
            return podName
          } */
    },
    role(node) {
      var isMaster = node.metadata.labels["role"] === "master";
      var isProxy = node.metadata.labels["proxy"] === "true";
      return isMaster ? "Master" : isProxy ? "Proxy" : "Worker";
    },
    getNamespace: function() {
      return this.$route.params["ns"];
    },
    podsInNode(node) {
      return this.pods.items.filter(m => m.spec.nodeName === node);
    },
    myPod(pod) {
        return !(pod.metadata.name.match(/es-/g) || 
          pod.metadata.name.match(/metrics/g) || 
          pod.metadata.name.match(/metering/g) || 
          pod.metadata.name.match(/vulnerability/g) || 
          pod.metadata.name.match(/k8s-proxy/g) || 
          pod.metadata.name.match(/k8s-proxy/g) || 
          pod.metadata.name.match(/prometh/g) || 
          pod.metadata.name.match(/calico/g)) 
    },
    refreshNodes: function(url) {
      if (process.env.NODE_ENV === "development") {
        this.nodes = nodesJson;
      } else {
        axios
          .get(url, {
            headers: {
              "Content-Type": "application/json"
            }
          })
          .then(response => {
            // JSON responses are automatically parsed.
            this.nodes = response.data;
          })
          .catch(e => {
            this.errors.push(e);
          });
      }
    },
    refreshPods: function(url) {
      if (process.env.NODE_ENV === "development") {
        this.pods = podsJson;
      } else {
        axios
          .get(url, {
            headers: {
              "Content-Type": "application/json"
            }
          })
          .then(response => {
            // JSON responses are automatically parsed.
            this.pods = response.data;
          })
          .catch(e => {
            this.errors.push(e);
          });
      }
    }
  },
  // Fetches posts when the component is created.
  created() {
    var nodesUrl = "http://localhost:8001/api/v1/nodes";
    var self = this;
    this.refreshNodes(nodesUrl);
    setInterval(function() {
      self.refreshNodes(nodesUrl);
    }, 5000);

    var podsUrl = "http://localhost:8001/api/v1/pods";
    this.refreshPods(podsUrl);
    setInterval(function() {
      self.refreshPods(podsUrl);
    }, 5000);
  }
};
</script>

<style scoped>
.master .right,.proxy .right,.header .right{
  text-align: right;
  width: 30%;
}
.proxy{
  background-color: #fb67d4;
  padding: 5px;
  margin-bottom: 10px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;

}
.master{
  background-color: #fc2020;
  padding: 5px;
  margin-bottom: 10px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
.master .left,.proxy .left,.header .left{
  float:left;
  width: 65%;
}
.pod{
  display: block;
}
div.ns > div {
  padding: 0px 0px 10px 0px;
}
.master i,
.proxy i,
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
  width: 92%;
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
  width: 5%;
  margin-left: 15px;
  background: #222;
  min-height: 20px;
  min-width: 100px;
  margin-top: 10px;
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
