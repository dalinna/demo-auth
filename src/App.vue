<template>
  <button type="primary" @click="logoutClick" v-if="hasLogin">登出</button>
  <button type="primary" @click="startWithRedirect" v-else>去登录页</button>
  -------{{ hasLogin }}
</template>
<script >
import { Authing } from '@authing/web';

export default {
  data() {
    return {
      sdk: null,
      hasLogin: false,
    };
  },
  methods: {
    init() {
      const {
        VUE_APP_AUTH_APPID, VUE_APP_AUTH_DOMAIN, VUE_APP_AUTH_REDIREACTURI, VUE_APP_AUTH_POOLID,
      } = process.env;
      this.sdk = new Authing({
        domain: VUE_APP_AUTH_DOMAIN,
        appId: VUE_APP_AUTH_APPID,
        redirectUri: VUE_APP_AUTH_REDIREACTURI,
        userPoolId: VUE_APP_AUTH_POOLID,
        // redirectToOriginalUri: true,
      });
    },
    startWithRedirect() {
      this.sdk.loginWithRedirect();
    },
    async getLoginState() {
      const state = await this.sdk.getLoginState();
      console.log('state', state);
      this.hasLogin = !!state;
      // if(!state){ // 未登录
      // this.startWithRedirect();
      // }
    },
    logoutClick() {
      const { VUE_APP_AUTH_REDIREACTURI } = process.env;
      this.sdk.logoutWithRedirect({
        redirectUri: VUE_APP_AUTH_REDIREACTURI,
      });
    },
  },
  async mounted() {
    this.init();
    this.getLoginState();
    if (this.sdk.isRedirectCallback()) {
      this.sdk.handleRedirectCallback().then(async (res) => {
        console.log('res>>>>>>>>>>>>>>', res);
        this.getLoginState();
        const userInfo = await this.sdk.getUserInfo();
        console.log('userInfo', userInfo);
        // server.sendUserinfo(userInfo)
      });
    } else {
      console.log('d0d0d0');
    }
  },
};
</script>
