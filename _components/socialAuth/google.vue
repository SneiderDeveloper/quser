<template>
  <socialBtn 
    v-if="clientIdGoogle" 
    @click.native="signIn()" 
    :title="`${$tr('isite.cms.label.continueWith')} ${$tr('isite.cms.label.google')}`"
    :icon="require('@imagina/quser/_components/socialAuth/icons/google.svg')"
  />
</template>

<script>
import socialBtn from '@imagina/quser/_components/socialAuth/socialBtn.vue';

export default {
  props: {
    buttonProps: {
      default: () => {
        return {}
      }
    }
  },
  components: {socialBtn},
  watch: {},
  mounted() {
    this.$nextTick(function () {
      this.addCDN()
    })
  },
  data() {
    return {
      success: true,
      loading: false
    }
  },
  computed: {
    clientIdGoogle() {
      return this.$store.getters['qsiteApp/getSettingValueByName']('isite::googleClientId')
    },
    propsButton() {
      return {
        color: 'red',
        icon: 'fab fa-google',
        loading: this.loading,
        vIf: this.success,
        ...this.buttonProps
      }
    }
  },
  methods: {
    //add CDN  to head
    addCDN() {
      let cdn = document.createElement('script')//create CDN google recaptcha
      cdn.setAttribute('src', 'https://apis.google.com/js/platform.js')
      cdn.onload = this.loadClientId()//callback when loaded cdn
      document.head.appendChild(cdn)//add to head
    },
    //Load Client ID
    loadClientId() {
      setTimeout(() => {
        let clientId = this.clientIdGoogle || null;
        if(!clientId) return;
        gapi.load('auth2', () => {
          gapi.auth2.init({client_id: clientId}).then(() => {
            this.success = true
          }).catch(error => {
          })
        })
      }, 500)
    },
    //SignIn method
    signIn() {
      let auth2 = gapi.auth2.getAuthInstance();
      let app = this
      this.loading = true

      auth2.signIn({'scope': 'profile email'}).then(response => {
        app.login(response)
        this.loading = false
      }).catch(error => {
        this.loading = false
      })
    },
    //Request Login
    login(response) {
      let token = false;

      // Finding access_token in google response
      for (var k in response)
        if (typeof response[k] == "object") {
          let object = response[k];
          for (var j in object)
            if (j == "access_token")
              token = object[j]
        }

      //Validate response
      if (!token) {
        this.loading = false
        return this.$alert.error(this.$tr('isite.cms.message.errorRequest'))
      }

      //Dispath auth social network
      this.$store.dispatch('quserAuth/AUTH_SOCIAL_NETWORK', {
        type: 'google',
        token: token
      }).then(response => {
        this.$emit('logged')
        this.loading = false
      }).catch(error => {
        this.$alert.error(this.$tr('isite.cms.message.errorRequest'))
        this.loading = false
      })
    }
  }
}
</script>
<style lang="stylus">

</style>
