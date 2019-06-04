<template>
  <div>
    <h1>Kube-Vue</h1>

    <div class="app">
        <div class="svg-wrapper" id="drawing">
            <svg class="node" :width="vm.deployments.length * d.w + 70" :height="c.h + 5">
                <rect class="just-border" x="10" y="10" :width="vm.deployments.length * d.w + 50" :height="c.h-15"></rect>
                <g v-for="(dep, ref) in vm.deployments" :key="ref">
                    /*dep*/     <rect :x="d.left + ((d.left + d.w) * ref)" :y="d.top" :width="d.w" :height="d.h" :class="d.style"></rect>
                                <!--<text :x="d.left + ((d.left + d.w) * ref) + d.w/2" :y="200" text-anchor="middle" fill="#000" class="podlabel">{{dep.name}}</text>-->

                    /*NodePort*/<rect v-if="dep.pods.service.nodePort!==''" :x="d.left + ((d.left + d.w) * ref) + d.w/2 - s.w/6" :y="d.top+s.top-28" 
                                  :width="s.w/3" :height="s.h/2.0" :class="np.style"></rect>
                                <text v-if="dep.pods.service.nodePort!==''" :x="d.left + ((d.left + d.w) * ref) + d.w/2" :y="d.top+s.top-15+10" 
                                  text-anchor="middle" fill="#000" class="nplabel">(Node Port)</text>
                                <text v-if="dep.pods.service.nodePort!==''" :x="d.left + ((d.left + d.w) * ref) + d.w/2" :y="d.top+s.top-17" 
                                  text-anchor="middle" fill="#000" class="servicelabel">{{dep.pods.service.nodePort}}</text>
                    /*service*/ <rect v-if="dep.pods.service.name!=='None'" :x="d.left + ((d.left + d.w) * ref) + d.w/2 - s.w/2" :y="d.top+s.top" 
                                  :width="s.w" :height="s.h" :class="s.style"></rect>
                                <text v-if="dep.pods.service.name!=='None'" :x="d.left + ((d.left + d.w) * ref) + d.w/2" :y="d.top+s.top+ s.th-4" 
                                  text-anchor="middle" fill="#000" class="nplabel">IP - {{dep.pods.service.clusterIP}}:{{dep.pods.service.port}}</text>
                                <text v-if="dep.pods.service.name!=='None'" :x="d.left + ((d.left + d.w) * ref) + d.w/2" :y="d.top+s.top+ s.th*2" 
                                  text-anchor="middle" fill="#000" class="servicelabel">Service - {{dep.pods.service.name}}</text>
                                <line v-if="dep.pods.service.name!=='None'" class="service-1" 
                                  :x1="d.left + ((d.left + d.w) * ref) + d.w/2" :y1="d.top+s.top+s.h" 
                                  :x2="d.left + ((d.left + d.w) * ref) + d.w/2" :y2="d.top+s.top+p.top+s.h"></line>
                        
                    <g v-for="(pod, index) in dep.pods.items" :key="index">
                    /*pod*/     <rect :x="(index * p.offset) + d.left + ((d.left + d.w) * ref) + d.w/2 - p.w/2" 
                                  :y="(index * p.offset) + d.top+s.top+p.top+s.h" :width="p.w" :height="p.h" 
                                  :class="pod.status"></rect>
                                <text :x="(index * p.offset) + d.left + ((d.left + d.w) * ref) + d.w/2" 
                                  :y="(index * p.offset) + d.top+s.top+p.top+s.h+ 11" text-anchor="middle" fill="#000" 
                                  class="nplabel">IP - {{(pod.status==='Running'?pod.podIP:pod.status)}}:{{dep.pods.service.targetPort}}</text>
                                <text :x="(index * p.offset) + d.left + ((d.left + d.w) * ref) + d.w/2" 
                                  :y="(index * p.offset) + d.top+s.top+p.top+s.h+ 28" text-anchor="middle" fill="#000" 
                                  class="podlabel">{{pod.name}}</text>
                    </g>

                    
                </g>
            </svg>
        </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import deploymentsJson from "../assets/deployments.json";
import podsJson from "../assets/pods.json";
import servicesJson from "../assets/services.json";

export default {
  data() {
    return {
      c: {
          //canvas ui properties
          w: 1200,
          h: 400
      },
      d: {
          //deployment ui properties
          style: "plain",
          left: 20,
          top: 20,
          w: 290,
          h: 200
      },
      p: {
          //pod ui properties
          style: "Running",
          top: 50,
          w: 180,
          h: 50,
          offset:14,
      },
      np:{
        style:"nodeport",
      },
      s: {
          //service ui properties
          style: "service",
          top: 12,
          w: 180,
          h: 55,
          th: 15,
      },
      l: {
          //service ui properties
          style: "service-1"
      },
      errors : [],
      deployments: [],
      pods: [],
      services: [],
      vm:{deployments:[]},
    };
  },

  methods: {
    prepareData: function(){
      var self = this
      this.vm = {deployments:[]}
      this.deployments.items.forEach(function(depl){
        var obj = depl.spec.selector.matchLabels
        var selectors = []
        for (var key in obj) {
          selectors.push(key + "+" + obj[key])
        }

        var serviceForDeployment = null;
        self.services.items.forEach(function(svc){
          if("selector" in svc.spec){
            var lobj = svc.spec.selector;
            for (var selectKey in lobj){
              var match = selectors.filter(n => n == selectKey + "+" + lobj[selectKey]);
              if (typeof match !== 'undefined'){
                if (match.length>0){
                  serviceForDeployment = svc;
                }
              }
            }
          }
        })

        var podsFound = self.pods.items.filter(m => m.metadata.name.startsWith(depl.metadata.name+"-")); 
        var podsInDeployment = []
        var ldepl = null;
        podsFound.forEach(function(p){
          podsInDeployment.push({"name":p.metadata.name,"status":p.status.phase,"podIP": ("podIP" in p.status? p.status.podIP:"")})
        });
        var port="";
        var nodePort="";
        var targetPort="";

        if(serviceForDeployment){
          if(serviceForDeployment.spec.ports.length>0){
            var portInfo = serviceForDeployment.spec.ports[0]
            nodePort = "nodePort" in portInfo? portInfo["nodePort"]:""
            targetPort = "targetPort" in portInfo? portInfo["targetPort"]:""
            port = "port" in portInfo? portInfo["port"]:""
          }

          ldepl = {
            "name":depl.metadata.name,
            "pods":{
              "items":podsInDeployment,
              "service":{
                "port":port,
                "nodePort":nodePort,
                "targetPort":targetPort,
                "name":serviceForDeployment.metadata.name,
                "clusterIP":serviceForDeployment.spec.clusterIP,
                "type":serviceForDeployment.spec.type,
              }
            },
          }
        }
        else{
          ldepl = {
            "name":depl.metadata.name,
            "pods":{
              "items":podsInDeployment,
              "service":{
                "port":"",
                "nodePort":"",
                "targetPort":"",
                "name":"None",
                "clusterIP":"",
                "type":"",
              }
            },
          }
        }
        self.vm.deployments.push(ldepl);
      })
      //      return this.pods.items.filter(m => m.spec.nodeName === node);

    },
    refreshDeployments: function(url) {
      if (process.env.NODE_ENV === "development") {
        this.deployments = deploymentsJson;
        //console.log(this.deployments.items.length);
        this.pods = podsJson;
        //console.log(this.pods.items.length);
        this.services = servicesJson;
        //console.log(this.services.items.length);
        this.prepareData();
      } else {
        axios
          .get(url, {
            headers: {
              "Content-Type": "application/json"
            }
          })
          .then(response => {
            // JSON responses are automatically parsed.
            this.deployments = response.data;
            this.refreshPods("http://localhost:8001/api/v1/namespaces/default/pods");
          })
          .catch(e => {
            this.errors.push(e);
          });
      }
    },
    refreshPods: function(url) {
      axios
        .get(url, {
          headers: {
            "Content-Type": "application/json"
          }
        })
        .then(response => {
          // JSON responses are automatically parsed.
          this.pods = response.data;
          this.refreshServices("http://localhost:8001/api/v1/namespaces/default/services");
        })
        .catch(e => {
          this.errors.push(e);
        });
    },
    refreshServices: function(url) {
      axios
        .get(url, {
          headers: {
            "Content-Type": "application/json"
          }
        })
        .then(response => {
          // JSON responses are automatically parsed.
          this.services = response.data;
          this.prepareData();
        })
        .catch(e => {
          this.errors.push(e);
        });
    }
  },
  mounted() {
    var deploymentsUrl = "http://localhost:8001/apis/extensions/v1beta1/namespaces/default/deployments";
    var self = this;
    this.refreshDeployments(deploymentsUrl);
    if (process.env.NODE_ENV === "production"){
      setInterval(function() {
        self.refreshDeployments(deploymentsUrl);
      }, 5000);
    }
  }
};
</script>

<style scoped>

.animate{
    transition: opacity .5s ease-in-out;
}
svg-wrapper{
    margin: 0 auto;
}
svg {
  background-color: #fff;
}

.app{
    margin: 10px;
}
circle { 
  fill: #6495ed;
  animation-duration: 1s;
  animation-name: pulse;
}

.just-border{
  stroke: #777;
  fill: #eee;
}
.plain{
  fill: #eee;
}

rect.Running {
  stroke: #006400;
  stroke-width: 2px;
  fill: #88dd44;
  animation-duration: 1s;
  animation-name: pulse;
}

rect.Pending {
  stroke: #daa520;
  stroke-width: 2px;
  fill: #ffd700;
  animation-duration: 1s;
  animation-name: pulse;
}
rect.depl{
  stroke: #4682b4;
  fill: white;
}
.nodeport{
  stroke: #a57f40;
  fill: #e8b257;
}
rect.service {
  stroke: #4682b4;
  fill: #6495ed;
  stroke-width: 2px;
  animation-duration: 1s;
  animation-name: pulse;
}

rect.deployment {
  stroke: #9932cc;
  fill: #b954d3;
  stroke-width: 2px;
  animation-duration: 1s;
  animation-name: pulse;
}

line.service-1 {
  stroke: #888;
  stroke-width: 2px;
}

line.deployment-1 {
  stroke: #888;
  stroke-width: 2px;
}

.podlabel {
  font: 12px arial;
}

.servicelabel {
  font: 12px arial;
}

.nplabel {
  font: 10px arial;
}

img.podlogo {
  position: relative;
  top: -300px;
}

@keyframes pulse {
  from {
    fill-opacity: 0;
    stroke-opacity: 0;
  }

  to {
    fill-opacity: 1;
    stroke-opacity: 1;
  }
}


</style>
