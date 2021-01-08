<template>
  <div>
    <section class="ui two column centered grid" style="position:relative;z-index:1;">
      <div class="column">
        <form class="ui segment large form">
          <div class="ui message red" v-show="error">{{ error }}</div>
          <div class="ui segment">
            <div class="field">
              <div class="ui right icon input large" :class="{ loading:spinner }">
                <input type="text" placeholder="Enter your address" v-model="address" id="autcomplete">
                <i class="dot circle link icon" @click="locatorButtonPressed"></i>
              </div>
            </div>
            <button type="button" class="ui button">Go</button>
          </div>
        </form>
      </div>
    </section>
    <section id="map"></section>
  </div>
</template>
<script>
import axios from 'axios';
export default {
  data() {
    return {
      key: '',
      address: '',
      error: '',
      spinner: false
    };
  },
  mounted() {
    let autocomplete = new google.maps.places.Autocomplete(
      document.getElementById('autcomplete'),
      {
        bounds: new google.maps.LatLngBounds(
          new google.maps.LatLng(21.0181, -101.258)
        )
      }
    );
    autocomplete.addListener("place_changed", () => {
      let place = autocomplete.getPlace();
      console.log(place);
      this.showUserLocationOnTheMap(place.geometry.location.lat(), place.geometry.location.lng());
    });
  },
  methods: {
    locatorButtonPressed() {
      this.spinner = true;
      if(navigator.geolocation){
        navigator.geolocation.getCurrentPosition(position => {
          this.getAddressFrom(position.coords.latitude, position.coords.longitude);
          this.showUserLocationOnTheMap(position.coords.latitude, position.coords.longitude);
        }, error => {
          this.error = "Locator is unable to find your address. Please type your address manually.";
          this.spinner = false;
        });
      }else{
        this.error = error.message;
        this.spinner = false;
      }
    },
    getAddressFrom(lat, long){
      axios.get(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${lat},${long}&key=${this.key}`).then(response => {
        if(response.data.error_message){
          this.error = response.data.error_message;
        }else{
          this.address = response.data.results[0].formatted_address;
        }
      }).catch(error => {
        this.error = error.message;
      });
    },
    showUserLocationOnTheMap(latitude, longitude){
      let map = new google.maps.Map(document.getElementById("map"),{
        zoom: 15,
        center: new google.maps.LatLng(latitude, longitude),
        mapTypeId: google.maps.MapTypeId.ROADMAP
      });
      new google.maps.Marker({
        position: new google.maps.LatLng(latitude, longitude),
        map: map
      });
    }
  }
}
</script>
<style>
.ui.button,
.circle.icon{
  background-color: #ff5a5f;
  color: #fff;
}
.pac-icon{
  display: none;
}
.pac-item{
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}
.pac-item:hover{
  background-color: #ececec;
}
#map{
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: teal;
}
</style>
