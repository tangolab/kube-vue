<template>
  <div>
    <h1>Pods in {{ getNamespace() }} namespace</h1>
  
    <div class="about">
      <div v-for="post in posts.items" :key="post.metadata.uid">
        <transition name="fade">
          <div class="pod-border card container style_prevu_kit" v-if="post.metadata.namespace === getNamespace()">
            <div class="node-name">
              {{ post.metadata.name }}
            </div>
            <div><span class="caption">Status</span> <span class="data">{{post.status.phase}} </span></div>
            <div><span class="caption">Created Date</span> <span class="data">{{moment(post.metadata.creationTimestamp).format('MM/DD/YYYY hh:mm:ss A ')}} </span></div>
            <div><span class="caption">Containers</span> <span class="data">{{post.spec.containers[0].image}} </span></div>
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import timeline from '../assets/pods.json'
  
  export default {
    data() {
      return {
        posts: [],
        errors: []
      }
    },
    methods: {
      getNamespace: function() {
        return this.$route.params['ns']
      },
      refresh: function(url) {
        if (process.env.NODE_ENV === "development") {
          this.posts = timeline
        } else {
          axios.get(url, {
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
      }
    },
    // Fetches posts when the component is created.
    created() {
      var url = "http://localhost:8001/api/v1/pods"
      var self = this
      this.refresh(url)
      setInterval(function() {
        self.refresh(url)
      }, 5000);
    }
  }
</script>

<style>
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity .5s;
  }
  
  .fade-enter,
  .fade-leave-to
  /* .fade-leave-active below version 2.1.8 */
  
  {
    opacity: 0;
  }
  
  .pod-border {
    border: 1px solid #d6d2d2;
    padding: 2px 16px;
    margin: 5px;
    width: 255px;
    float: left;
    height: 235px;
    overflow: hidden;
    font-size: 13px;
  }
  
  div.about div div {
    text-align: left;
  }
  
  span.caption {
    width: 145px;
    display: inline-block;
    overflow: hidden;
    font-weight: bold;
  }
  
  span.data {
    width: 865px;
    display: inline-block;
    overflow: hidden;
  }
  
  .node-name {
    font-size: 20px;
    padding-top: 5px;
    padding-bottom: 10px;
  }
  
  .image-size {
    text-align: right;
    padding-right: 20px;
  }
  
  div.header {
    padding: 15px 0px 5px 0px;
    font-weight: bold;
    font-size: 15px;
  }
  
  .card {
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
    transition: 0.3s;
  }

</style>
