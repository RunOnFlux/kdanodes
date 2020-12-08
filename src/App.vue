<template>
  <div id="app">
    <div class="mainDivStyle">
      <div class="header">
        <div class='header-logo'>
          <span class="helper"></span>
          <img src="./assets/img/flux_white_logo.svg">
          <img
            src="./assets/img/love.svg"
            style="margin-left: 10px; margin-right: 10px;"
          >
          <img
            src="./assets/img/kadena.png"
            style="height: 40px;"
          >
        </div>
        <ElLink
          type="info"
          href="https://zel.network/flux/"
          target="_blank"
          rel="noopener noreferrer"
        >Your gateway to a decentralized world!</ElLink>
      </div>
      <div class="content">
        <NodeTracker />
      </div>
    </div>
  </div>
</template>

<script>
import NodeTracker from './components/NodeTracker.vue'

export default {
  name: 'App',
  components: {
    NodeTracker
  }
}
</script>

<style src="./assets/css/main.css"></style>
