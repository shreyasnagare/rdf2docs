<template>

  <section class="container" v-breathing-colors="sample">
    <vue-progress-bar></vue-progress-bar>
    <div>
      <div class="title">
        BrickDocs
      </div>
      <div class="subtitle">
        Refer to the latest Brick concepts available
      </div>
      <search/>
    </div>
  </section>
</template>

<script>
  // import Logo from '~/components/Logo.vue'
  import Search from '~/components/Search.vue'
  // import BootstrapVue from 'bootstrap-vue'
  // import 'bootstrap/dist/css/bootstrap.css'
  // import 'bootstrap-vue/dist/bootstrap-vue.css'
  //
  // Vue.use(BootstrapVue)
  import Vue from 'vue';
  import BreathingColors from 'vue-breathing-colors';

  import VueTextareaAutosize from 'vue-textarea-autosize'
  import VueProgressBar from 'vue-progressbar'

  Vue.use(VueProgressBar, {
  color: '#007bff',
  failedColor: 'red',
  height: '8px'
})
Vue.use(VueTextareaAutosize)
Vue.use(BreathingColors);

export default {
  data() {
    return {
      sample: {
        colors: ['white'],
        interval: 3000,
        transition: {
          duration: 1000
        }
      }
    }
  },
  components: {
    // Logo,
    Search
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 6em;
  color: #35495e;
  letter-spacing: 1px;
  background: -webkit-linear-gradient(#57a2ff, #49b3b3);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.subtitle {
  font-weight: 300;
  font-size: 2em;
  color: #526488;
  /*word-spacing: 5px;*/
  padding-bottom: 15px;
  margin-bottom: 16px;
}

.links {
  padding-top: 15px;
}


.v--modal-background-click{
  background: #0579fc30;
}
</style>
