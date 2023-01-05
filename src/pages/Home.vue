<template>
  <div class="wrap">
    <Header />

    <div class="container">
      <MainView />
      <!-- <SubView /> -->
    </div>
  </div>
</template>

<script>
import Header from "../components/Header.vue";
import MainView from "../components/MainView.vue";
// import SubView from "../components/SubView.vue";

// export default { components: { MainView, SubView } };
export default {
  layout: "app",
  components: { MainView, Header },
};
</script>

<style lang="scss" scoped>
@import "~/scss/main.scss";
@import "~/scss/mainview.scss";

.wrap {
  width: 100%;
  font-size: 16px;

  .container {
    margin-top: 50px;
    @include center-sb;
    width: 100vw;
  }
}
</style>
