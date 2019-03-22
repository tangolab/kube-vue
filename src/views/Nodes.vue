<template>
  <div>
    <h1>Nodes</h1>
    <div class="about">
      <div v-for="post in posts.items" :key="post.metadata.uid">

        <div class="node-name">
          {{ post.metadata.name }}
        </div>  
        <div v-for="(addr, index) in post.status.addresses" :key="`${index}`">
          <span class="caption">{{addr.type}}</span> <span class="data">{{addr.address}}</span>
        </div>
        <div><span class="caption">CPU</span> <span class="data">{{post.status.capacity.cpu}} ({{post.status.allocatable.cpu}})</span></div>
        <div><span class="caption">Memory</span> <span class="data">{{post.status.capacity.memory}} ({{post.status.allocatable.memory}})</span></div>
        <div><span class="caption">Storage</span> <span class="data">{{post.status.capacity['ephemeral-storage']}}  ({{post.status.allocatable['ephemeral-storage']}})</span></div>
        <div><span class="caption">Pods</span> <span class="data">{{post.status.capacity.pods}}</span></div>
        <div class="header">Conditions</div>
        <div v-for="(condition, index) in post.status.conditions" :key="`${index}-A`">
          <span class="caption">{{condition.type}}</span> <span class="data">{{condition.status}} </span> 
        </div>
        <div class="header">Images in Node Registry </div>
        <div v-for="(image, index) in post.status.images" :key="`${index}-B`">
          <span class="caption image-size">{{image.sizeBytes | prettyBytes}}</span> <span class="data">{{image.names[1]}} </span>
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
      posts: [],
      errors: []
    }
  },

  // Fetches posts when the component is created.
  created() {
    if(process.env.NODE_ENV==="development"){
      this.posts = timeline 
    }else{
      axios.get(`http://localhost:8001/api/v1/nodes`,
      {
        headers: {
          'Content-Type': 'application/json',
        }
      })
      .then(response => {
        // JSON responses are automatically parsed.
        this.posts = response.data
      })
      .catch(e => {
        this.errors.push(e)
      })
    }


    // async / await version (created() becomes async created())
    //
    // try {
    //   const response = await axios.get(`http://jsonplaceholder.typicode.com/posts`)
    //   this.posts = response.data
    // } catch (e) {
    //   this.errors.push(e)
    // }
  }  
}
</script>

<style scoped>
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
