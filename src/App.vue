<template>
  <div id="app" v-if="typeof squaredPopUpData === 'object' || typeof roundedPopUpData === 'object'">
    <app-squared v-if="selectedComponent === 'appSquared'"
                 :data="squaredPopUpData"
                 :counter="myCounter"
                 :imageStorage="myStoragePath"
                 :isBranded="isBranded"
                 :brandingData="brandingData"></app-squared>
    <app-rounded v-if="selectedComponent === 'appRounded'"
                 :data="roundedPopUpData"
                 :counter="myCounter"
                 :imageStorage="myStoragePath"
                 :isBranded="isBranded"
                 :brandingData="brandingData"></app-rounded>
  </div>
</template>

<script>
/**
 * NOTE: written and maintained in ECMA6 version. Make sure you use the same version when you try to code even more touch.
 * author Tier5 LLC (Jonathan Vaughn)
 */
import squared from './components/squared.vue';
import rounded from './components/rounded.vue';
import axios from 'axios';
export default {
  name: 'app',
  data () {
    return {
      selectedComponent: '',
      myapiURL: '',
      campaignId: '',
      erros: [],
      switchme: true,
      squaredPopUpData: Array,
      roundedPopUpData: Array,
      myCounter: 0,
      myStoragePath: '',
      isBranded: Boolean,
      brandingData: Object
    };
  },
  components: {
    appSquared: squared,
    appRounded: rounded
  },
  computed: {
    /**
     * this function decodes if its a testing from localhost or live url
     */
    selectMyApiURL() {
      // includes method only came in the play just because im writting this code on ES6 , dont use if you are in ES5 use indexOf instead
      if (window.location.host == "localhost" || window.location.host == "127.0.0.1" || window.location.host.includes('localhost')) {
        this.myapiURL = "http://127.0.0.1:8000/api/v1/";
      } else {
        this.myapiURL = "http://34.239.247.213/api/v1/";
      }
      return this.myapiURL;
    },
    /**
     * this function gives back the campaign Id for which we need the sticky reviews
     */
    myApiId() {
      this.campaignId = document.querySelector('script[data-name="_emv"][data-token]').getAttribute('data-token');
      return this.campaignId;
    },
  },
  methods: {
    /**
     * this function fires up a request to backend to get all the details of a campaign
     */
    showComponent() {
      try {
        axios.get(this.selectMyApiURL + 'campaign-details/' + this.myApiId)
          .then(data => {
            if (data.status === 200 && data.statusText === "OK") {
              this.checkDomainName(data.data.response);
            } else {
              console.error('[evenmorevago error::]- Something went wrong while fetching campaign data!');
            }
          })
          .catch(e => {
            this.erros.push(e);
            console.error('[evenmorevago error::]- ' + e);
          });
      } catch (e) {
        console.error('[evenmorevago error::]- ' + e);
      }
    },
    /**
     * this function checks the domain name for what it has been made for matched or not
     * @param {string} dataresponse
     */
    checkDomainName(dataresponse) {
      //this.formPopUps(dataresponse);
      if (window.location.origin == dataresponse.domain_name || window.location.origin+'/' == dataresponse.domain_name) {
        this.formPopUps(dataresponse);
      } else {
        console.error('Campaign URL did not match with current URL');
      }
    },
    /**
     * this function forms the array index which needs to passed to the child component for showing up data
     * @param {object} wholeresponse
     */
    formPopUps(wholeresponse) {
      if (wholeresponse) {
        const vmm = this;
        if (wholeresponse.styles === 'square') {
          vmm.selectedComponent = 'appSquared';
          this.createPopUps(wholeresponse, true);
        } else {
          vmm.selectedComponent = 'appRounded';
          this.createPopUps(wholeresponse, false);
        }
      } else  {
        console.error('[evenmorevago error::]- ' + 'Empty response!');
      }
    },
    /**
     * this function sets up actual array index which we are passing and data shows up forever
     * @param {object} wholeresponse
     * @param {boolean} type
     */
    createPopUps(wholeresponse, type) {
      if (wholeresponse) {
        if (wholeresponse.sticky_reviews !== null) {
          if (wholeresponse.sticky_reviews.length > 0) {
            if (type) {
              this.squaredPopUpData = wholeresponse.sticky_reviews;
            } else {
              this.roundedPopUpData = wholeresponse.sticky_reviews;
            }
            // check branding is there or not
            this.isBranded = wholeresponse.branding ===  1 ? true: false;
            this.brandingData = wholeresponse.branding_details;
            // getting the image storage path
            this.getStoragePath();
            let vm = this;
            setInterval(function () {
              if (vm.myCounter === wholeresponse.sticky_reviews.length -1) {
                vm.myCounter = 0;
              } else {
                vm.myCounter++;
              }
            },wholeresponse.delay);
          } else {
            console.warn('[evenmorevago warn::]- ' + 'Empty set of sticky reviews');
          }
        } else {
          console.warn('[evenmorevago warn::]- ' + 'Empty set of sticky reviews');
        }
      } else {
        console.error('[evenmorevago error::]- ' + 'Empty response!');
      }
    },
    /**
     * get the storage path for image files
     */
    getStoragePath() {
      try {
        axios.get(this.selectMyApiURL + 'forget-storage-path/')
          .then(data => {
            if (data.status === 200 && data.statusText === "OK") {
              this.myStoragePath = data.data.response;
            } else {
              console.error('[evenmorevago error::]- Something went wrong while fetching image storage path!');
            }
          })
          .catch(e => {
            this.erros.push(e);
            console.error('[evenmorevago error::]- ' + e);
          });
      } catch (e) {
        console.error('[evenmorevago error::]- ' + e);
      }
    }
  },
  /**
   * calling a method on page load using life cycle hook
   * I choose mounted rather beforeMount just because vue js uses virtual DOM so it also keep the track of the value changes
   * on mount if there is any
   */
  mounted () {
    this.showComponent();
  }
}
</script>

<style scoped>
</style>
