<template>
  <q-layout view="lHh Lpr lFf">
    <q-header class="bg-pink-13" text-gray-10 bordered>
      <q-toolbar class="constrain">
        <!-- adding a button for the logo next to the title -->
        <!-- q-mt-sm padding for margin top (mt) -->
        <q-btn
          to="/camera"
          class="large-screen q-mt-sm"
          icon="fas fa-camera-retro"
          size="14px"
          flat
          round
          dense
        />
        <!-- dense occupies less space -->
        <q-seperator class="large-screen" vertical spaced />
        <!-- this is the toolbar title-->
        <!-- using grand-hotel from css file and making text-bold-->
        <q-toolbar-title class="text-grand-hotel text-bold"
          >InstaQuasar</q-toolbar-title
        >
        <q-btn
          to="/"
          class="large-screen q-mt-sm"
          icon="fas fa-home"
          size="14px"
          flat
          round
          dense
        />
      </q-toolbar>
    </q-header>

    <!-- a basic footer copied from quasar manual -->
    <q-footer class="bg-white small-screen" bordered>
      <!-- elevated gives it a shadow and bordered gives it a default border-->
      <!-- here we have a footer title which will be removed for footer tabs -->
      <q-tabs class="text-pink-11" active-color="primary">
        <!-- active-color changes to the primary color of the page which is located in scss
        you can add indicator-color="transparant" which changes the color of the line when you click on the tab-->
        <q-route-tab to="/" name="Home" icon="fas fa-home" />

        <!-- adding to allows us to click on the home or camera icon and take us to the page -->
        <q-route-tab to="/camera" name="Camera" icon="fas fa-camera-retro" />
      </q-tabs>
      <!-- to remove the icons from the left and center them, remove the toolbar component
        to change icons go to quasar icon pages install icon libraries  and type their name on the icon class  -->
    </q-footer>

    <!-- the q-drawer that was here in the beginning is a side navigation bar --->
    <!-- this is the whole page excluding the nav and footbar -->
    <!-- bg-grey-1 changes the page color -->
    <q-page-container class="bg-grey-1">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<!-- mobile to put the navbar text in the middle in the center -->
<style lang="sass">
@media (min-width: $breakpoint-sm-min)
  .q-toolbar
    height: 77px

@media (max-width: $breakpoint-xs-max)
  .q-toolbar__title
    font-size: 30px
    text-align: center

.q-footer .q-tab__icon
  font-size: 25px
</style>
