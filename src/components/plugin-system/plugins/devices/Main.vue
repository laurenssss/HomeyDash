<template>
<div>

  <div style="width:100%;text-align:center;" v-if="!loaded">
    <q-transition appear enter="fadeIn">
    <q-spinner style="margin-top:50px;margin-left:0 auto;" color="teal-4" size="50px" />
    </q-transition>
  </div>

  <div class="row" v-if="!$route.params.zone && loaded">
    <h5>
      <q-icon name="fa-arrow-left" /> Select a zone in the sidebar
    </h5>
  </div>



  <div class="row devices scroll" v-if="$route.params.zone && loaded">

    <!-- ONOFF Capabilities -->
    <div v-if="device.zone.id === $route.params.zone && device.capabilities.onoff && !device.capabilities.alarm_motion && device.class != 'windowcoverings'" class="col-lg-2 col-md-2 col-xs-4 col-sm-2" v-for="device in devices">
        <q-transition appear enter="fadeIn" leave="fadeOut"><onoff :device="device" /></q-transition>
    </div>

    <!-- LOCK -->
    <div v-if="device.zone.id === $route.params.zone && device.capabilities.locked" class="col-lg-2 col-md-2 col-xs-4 col-sm-2" v-for="device in devices">
      <q-transition appear enter="fadeIn" leave="fadeOut">  <lock :device="device" /></q-transition>
    </div>

    <!-- MOTION -->
    <!-- <div v-if="device.zone.id === $route.params.zone && device.capabilities.alarm_motion" class="col-lg-2 col-md-2 col-xs-4 col-sm-2" v-for="device in devices">
      <motion :device="device" />
    </div> -->

    <!-- SENSOR -->
    <div v-if="device.zone.id === $route.params.zone && device.class == 'sensor' && !device.capabilities.onoff" class="col-lg-2 col-md-2 col-xs-4 col-sm-2" v-for="device in devices">
        <q-transition appear enter="fadeIn" leave="fadeOut"><sensor :device="device" /></q-transition>
    </div>

    <!-- WINDOWBLINDS -->
    <div v-if="device.zone.id === $route.params.zone && device.class == 'windowcoverings'" class="col-lg-2 col-md-2 col-xs-4 col-sm-2" v-for="device in devices">
      <q-transition appear enter="fadeIn" leave="fadeOut">  <windowblinds :device="device" /> </q-transition>
    </div>

  </div>

</div>
</template>

<script>
import onoff from '@/devices/Onoff'
import lock from '@/devices/Lock'
import motion from '@/devices/Motion'
import windowblinds from '@/devices/Windowblinds'
import sensor from '@/devices/Sensor'

export default {
  components: {
    onoff,
    lock,
    motion,
    windowblinds,
    sensor
  },
  data() {
    return {
      // initializing for second tab to be selected by default
      devices: {},
      loaded: false
    }
  },
  async mounted() {
    if(this.devices) {
      setTimeout(() => {
        this.loaded = true;
      }, 500);
    }
    if(!this.$route.params.zone && window.localStorage.getItem('lastDevicePage')){
      this.$router.push({ name: 'Devices', params: { zone: window.localStorage.getItem('lastDevicePage') }})
    }
    await this.$homey.devices.subscribe();
    this.devices = await this.$homey.devices.getDevices();
    _.forEach(this.devices, device => {
      device.on('$state', state => {
        // console.log(state);
      })
    });
  },
  beforeRouteLeave (to, from, next) {
    if(to.name !== 'Devices'){
      window.localStorage.setItem('lastDevicePage', from.params.zone)
    }
    next();
  },
  methods: {

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" scoped>
@import '~variables'

h5
  width 100%
  color $neutral
  text-align center


</style>
