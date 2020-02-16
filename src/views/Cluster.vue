<template>
  <div>
    <h1>
      Pod Distribution
      <input type="text" v-model="filtext" />
    </h1>
    <div class="ns">
      <div v-for="(namesp, id) in namespaces.items" :key="`${id}`">
        <div v-if="1">
          <router-link :to="{ path: '/cluster?ns=' + namesp.metadata.name}">{{namesp.metadata.name}}</router-link>
        </div>
      </div>
    </div>

    <div class="pods">
      <div class="env" v-if="this.$route.query['ns']">Showing {{this.$route.query["ns"]}}</div>
      <div class="node" v-for="node in nodes.items" :key="node.metadata.uid">
        <div
          v-bind:class="[role(node)==='Proxy'? proxyClass : role(node)==='Master'? masterClass: headerClass]"
        >
          <div class="left">{{ node.metadata.name }}</div>
          <div class="right">{{role(node)}}</div>
        </div>
        <div class="pod" v-for="pod in podsInNode(node.metadata.name)" :key="pod.metadata.uid">
          <div
            v-bind:class="{blue : pod.status.phase == 'Running', yellow : pod.status.phase == 'Pending', gray: pod.status.phase == 'Succeeded', red: pod.status.phase == 'Terminating'}"
            v-if="myPod(pod)"
          >
            {{plainPodName(pod)}}
            <div class="pod-details">
              <pre>Namespace: {{pod.metadata.namespace}}
Name: {{pod.metadata.name}}
Status: {{pod.status.phase}}
Created: {{pod.metadata.creationTimestamp}}
</pre>
            </div>
          </div>
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
      filtext: "",
      proxyClass: "proxy",
      masterClass: "master",
      headerClass: "header",
      nodes: [],
      pods: [],
      namespaces: [],
      errors: []
    };
  },

  methods: {
    plainPodName(pod) {
      if (pod.metadata.labels) {
        var name = pod.metadata.labels.app;
        if (!name) {
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
    getNamespace() {
      return this.$route.query["ns"];
    },
    podsInNode(node) {
      if (this.pods.items) {
        var ns = this.getNamespace();
        if (ns) {
          return this.pods.items.filter(
            m =>
              m.spec.nodeName === node &&
              m.metadata.namespace == ns &&
              m.metadata.name.includes(this.filtext)
          );
        } else {
          return this.pods.items.filter(
            m =>
              m.spec.nodeName === node && m.metadata.name.includes(this.filtext)
          );
        }
      }
    },
    myPod(pod) {
      // eslint-disable-next-line
      if (pod) {
      }
      return true;
      /* return !(
        pod.metadata.name.match(/es-/g) ||
        pod.metadata.name.match(/metrics/g) ||
        pod.metadata.name.match(/metering/g) ||
        pod.metadata.name.match(/vulnerability/g) ||
        pod.metadata.name.match(/k8s-proxy/g) ||
        pod.metadata.name.match(/k8s-proxy/g) ||
        pod.metadata.name.match(/prometh/g) ||
        pod.metadata.name.match(/calico/g)
      ); */
    },
    refreshNamespaces: function(url) {
      axios
        .get(url, {
          headers: {
            "Content-Type": "application/json"
          }
        })
        .then(response => {
          // JSON responses are automatically parsed.
          this.namespaces = response.data;
        })
        .catch(e => {
          this.errors.push(e);
        });
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
    var nodesUrl = "/api/v1/nodes";
    var self = this;
    this.refreshNodes(nodesUrl);
    this.refreshNamespaces("http://localhost:8001/api/v1/namespaces");
    setInterval(function() {
      self.refreshNodes(nodesUrl);
    }, 5000);

    var podsUrl = "/api/v1/pods";
    this.refreshPods(podsUrl);
    setInterval(function() {
      self.refreshPods(podsUrl);
    }, 5000);
  }
};
</script>

<style scoped>
.master .right,
.proxy .right,
.header .right {
  text-align: center;
}
.proxy {
  background-color: #fb67d4;
  padding: 1px;
  margin-bottom: 1px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
  font-size: 10px;
}
.master {
  background-color: #fc2020;
  padding: 1px;
  margin-bottom: 1px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
  font-size: 10px;
}
.master .left,
.proxy .left,
.header .left {
  overflow: hidden;
}
div.ns > div {
  padding: 0px 0px 10px 0px;
}
.master i,
.proxy i,
.header i,
.ns div i {
  font-size: 10px !important;
  padding-right: 5px;
  color: #76ff5d;
}
.ns {
  width: 10%;
  float: left;
  max-width: 200px;
  min-width: 50px;
}
.pods {
  float: left;
  width: 90%;
}

.pod div{
  width: 66px;
  margin: 1px;
  float: left;
  padding: 0px 1px 0px 5px;
  font-size: 10px;
  border-radius: 0px;
  font-family: arial;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
.yellow {
  background-color: #ca7711;
}
.red {
  background-color: #fc2020;
}
.gray {
  background-color: #2b2b2a;
}
.blue {
  background-color: #0a44a1;
}
.pod {
  display: inline-block;
  position: relative;
}
.pod:hover div .pod-details {
  z-index: 10;
  opacity: 1;
  top: 1px;
  visibility: visible;
  transform: translate(0, 20px);
  transition: all 0.1s cubic-bezier(0.75, -0.02, 0.2, 0.97);
}

.pod div .pod-details {
  opacity: 0;
  visibility: hidden;
  position: absolute;
  top: 15px;
  transform: translate(0, 10px);
  background-color: #503030;
  padding: 0.5rem;
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.26);
  width: auto;
}
.env {
  padding: 0px 0px 10px 1px;
}
.node {
  border: 1px solid #555;
  float: left;
  width: 75px;
  background: #222;
  min-height: 20px;
  margin-top: 0px;
}
.header {
  background-color: #55657f;
  padding: 1px;
  margin-bottom: 1px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
</style>
