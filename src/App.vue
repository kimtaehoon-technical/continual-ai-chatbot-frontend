
<template>
  <div id="app">
    <ChatComponent />
  </div>
</template>

<script>
import ChatComponent from './components/ChatScreen.vue';

export default {
  name: 'AppRoot', // ESLint 대응 위해 두 단어 이상으로 변경
  components: {
    ChatComponent,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
