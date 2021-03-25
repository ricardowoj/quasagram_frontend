<template>
  <q-layout view="lHh Lpr lFf">
    <q-header class="bg-white text-grey-10" bordered>
      <q-toolbar class="constrain">
        <q-toolbar-title class="text-grand-hotel text-bold">
          Quasagram
        </q-toolbar-title>
        <q-btn
          to="/"
          class="text-grey-10 large-screen-only"
          color="primary"
          icon="eva-home"
          flat
          size="18px"
          push
          round
          dense
        />
        <q-btn
          to="/camera"
          class="text-grey-10 large-screen-only"
          color="primary"
          icon="eva-camera-outline"
          flat
          size="18px"
          push
          round
          dense
        />
      </q-toolbar>
    </q-header>

    <q-footer class="bg-white small-screen-only" bordered>
      <q-tabs
        class="text-grey-10"
        active-color="primary"
        indicator-color="transparent"
      >
        <q-route-tab to="/" icon="eva-home-outline" />
        <q-route-tab to="/camera" icon="eva-camera-outline" />
      </q-tabs>
    </q-footer>

    <q-page-container class="bg-grey-1">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  name: "MainLayout",
  data() {
    return {};
  }
};
</script>

<style lang="scss">
.q-toolbar {
  height: 75px;
}
.q-toolbar__title {
  font-size: 30px;
  @media (max-width: $breakpoint-xs-max) {
    text-align: center;
  }
}
.q-footer .q-tab__icon {
  font-size: 30px;
}
</style>
