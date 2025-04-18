<template>
  <div>
    <div class="text warning">{{ i18n.security_warning }}</div>
    <label>{{ i18n.phrase }}</label>
    <input class="input" type="password" v-model="phrase" />
    <label>{{ i18n.confirm_phrase }}</label>
    <input
      class="input"
      type="password"
      v-model="confirm"
      v-on:keyup.enter="changePassphrase()"
    />
    <div v-show="!enforcePassword">
      <div id="security-save" v-on:click="changePassphrase()">
        {{ i18n.ok }}
      </div>
      <div id="security-remove" v-on:click="removePassphrase()">
        {{ i18n.remove }}
      </div>
    </div>
    <div
      class="button-small"
      v-show="enforcePassword"
      v-on:click="changePassphrase()"
    >
      {{ i18n.ok }}
    </div>
    <div
      class="button-u2f"
      v-on:click="enableSecurityKey()"
      v-show="encryption.getEncryptionStatus() && u2fSupportedPlatform"
    >
      <IconWindowsHello v-if="u2fSupportedPlatform === 'Win'" />
      <IconTouchId v-if="u2fSupportedPlatform === 'Mac'" />
    </div>
  </div>
</template>
<script lang="ts">
import Vue from "vue";

import IconWindowsHello from "../../../svg/windows-hello.svg";
import IconTouchId from "../../../svg/touch-id.svg";

export default Vue.extend({
  data: function() {
    return {
      phrase: "",
      confirm: ""
    };
  },
  computed: {
    encryption: function() {
      return this.$store.state.accounts.encryption;
    },
    enforcePassword: function() {
      return this.$store.state.menu.enforcePassword;
    },
    u2fSupportedPlatform() {
      if (navigator.platform.indexOf("Win") > -1) {
        return "Win";
      } else if (navigator.platform.indexOf("Mac") > -1) {
        return "Mac";
      } else {
        return null;
      }
    }
  },
  methods: {
    async removePassphrase() {
      this.$store.commit("currentView/changeView", "LoadingPage");
      await this.$store.dispatch("accounts/changePassphrase", "");
      this.$store.commit("notification/alert", this.i18n.updateSuccess);
      this.$store.commit("style/hideInfo");
      return;
    },
    async changePassphrase() {
      if (this.phrase === "") {
        return;
      }

      if (this.phrase !== this.confirm) {
        this.$store.commit("notification/alert", this.i18n.phrase_not_match);
        return;
      }

      if (
        localStorage.securityTokenEncryptedKey &&
        !(await this.$store.dispatch(
          "notification/confirm",
          this.i18n.updatePassphraseDisableSecurityKeyWarning
        ))
      ) {
        return;
      }

      this.$store.commit("currentView/changeView", "LoadingPage");
      await this.$store.dispatch("accounts/changePassphrase", this.phrase);
      this.$store.commit("notification/alert", this.i18n.updateSuccess);
      this.$store.commit("style/hideInfo");
      return;
    },
    async enableSecurityKey() {
      if (
        !localStorage.securityTokenEncryptedKey ||
        (await this.$store.dispatch(
          "notification/confirm",
          this.i18n.alreadyEnabledSecurityKeyWarning
        ))
      ) {
        const result = await this.$store.dispatch("accounts/enableSecurityKey");
        this.$store.commit("notification/alert", result);
        this.$store.commit("style/hideInfo");
      }
      return;
    }
  },
  components: {
    IconWindowsHello,
    IconTouchId
  }
});
 </script># Authenticator [![Build Status](https://travis-ci.com/Authenticator-Extension/Authenticator.svg?branch=dev)](https://travis-ci.com/Authenticator-Extension/Authenticator) [![Crowdin](https://d322cqt584bo4o.cloudfront.net/authenticator-firefox/localized.svg)](https://crowdin.com/project/authenticator-firefox) <img align="right" width="100" height="100" src="https://github.com/Authenticator-Extension/Authenticator/raw/dev/images/icon.svg">

> Authenticator generates 2-Step Verification codes in your browser.

## Available for Chrome, Firefox, Microsoft Edge and Safari

[<img src="https://raw.githubusercontent.com/wiki/Authenticator-Extension/Authenticator/readme-images/chrome-web-store.png" title="Chrome Web Store" width="170" height="48" />](https://chrome.google.com/webstore/detail/authenticator/bhghoamapcdpbohphigoooaddinpkbai) [<img src="https://raw.githubusercontent.com/wiki/Authenticator-Extension/Authenticator/readme-images/firefox-add-ons.png" title="Firefox Add-ons" width="170" height="48" />](https://addons.mozilla.org/en-US/firefox/addon/auth-helper?src=external-github) [<img src="https://raw.githubusercontent.com/wiki/Authenticator-Extension/Authenticator/readme-images/microsoft-store.png" title="Microsoft Store" height="48">](https://microsoftedge.microsoft.com/addons/detail/ocglkepbibnalbgmbachknglpdipeoio) [<img width="150" alt="Download on the App Store" src="https://developer.apple.com/assets/elements/badges/download-on-the-app-store.svg"/>](https://apps.apple.com/us/app/authen/id1602945200?mt=12)

## Build Setup

``` bash
# install development dependencies
npm install
# compile
npm run [chrome, firefox, prod]
