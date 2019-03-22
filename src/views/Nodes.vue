<template>
  <div>
    <h1>Nodes</h1>
    <div class="about">
      <div class="node" v-for="node in nodes.items" :key="node.metadata.uid">

        <div class="node-name">
          {{ node.metadata.name }}
        </div>  
        <div><span class="caption">CPU</span> <span class="data">{{node.status.capacity.cpu}} ({{node.status.allocatable.cpu}})</span></div>
        <div><span class="caption">Memory</span> <span class="data">{{node.status.capacity.memory}} ({{node.status.allocatable.memory}})</span></div>
        <div><span class="caption">Storage</span> <span class="data">{{node.status.capacity['ephemeral-storage']}}  ({{node.status.allocatable['ephemeral-storage']}})</span></div>
        <div><span class="caption">Pods</span> <span class="data">{{node.status.capacity.pods}}</span></div>
        <div><span class="caption">Role</span> <span class="data">{{role(node)}}</span></div>
        <div class="header">Conditions</div>
        <div v-for="(condition, index) in node.status.conditions" :key="`${index}-A`">
          <span class="caption">{{condition.type}}</span> <span class="data">{{condition.status}} </span> 
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios' 
import timeline from '../assets/nodes.json'

export default {
  data() {
    return {
      nodes: [],
      errors: []
    }
  },
  methods: {
      role(node){
        var isMaster = node.metadata.labels["role"]==="master"
        var isProxy = node.metadata.labels["proxy"]==="true"
        return isMaster?"Master":isProxy?"Proxy":"Worker" 
      }
  },
  // Fetches nodes when the component is created.
  created() {
    if(process.env.NODE_ENV==="development"){
      this.nodes = timeline 
    }else{
      axios.get(`http://localhost:8001/api/v1/nodes`,
      {
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


    // async / await version (created() becomes async created())
    //
    // try {
    //   const response = await axios.get(`http://jsonplaceholder.typicode.com/nodes`)
    //   this.nodes = response.data
    // } catch (e) {
    //   this.errors.push(e)
    // }
  }  
}
</script>

<style scoped>
  .node{
    float:left;
    width:300px;
    border: 1px lightgrey solid;
    margin: 3px;
  }
  div.about div div{
    text-align: left;
  }
  span.caption{
    width: 145px;
    display: inline-block; 
    overflow: hidden;
  }
  span.data{
    width: 865px;
    display: inline-block; 
    overflow: hidden;
  }
  .node-name{
    font-size: 35px;
  }
  .image-size{
    text-align: right;
    padding-right: 20px;
  }
  div.header{
    padding: 15px 0px 5px 0px;
    font-weight: bold;
    font-size: 20px;
  }
</style>
