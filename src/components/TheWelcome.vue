<template>
  <div style="background-color: #fff">
    <apple-pay-button
      buttonstyle="black"
      type="buy"
      locale="en-GB"
      @click="onApplePayButtonClicked"
      
    ></apple-pay-button>
  </div>
</template>
<!-- v-if="showButton()" -->
<script>
import Axios from "axios";
const serverUrl = 'https://dev.acmedao.com';
const accessToken = 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IkFmdkNWWUsxRGxKWkRkNzRtSTI3VSJ9.eyJpc3MiOiJodHRwczovL2FjbWVjb3JlLWRldi51cy5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMDk0NTYyNTIxOTAzMjE5MjU3MzIiLCJhdWQiOlsiaHR0cHM6Ly9kZXYuYWNtZWRhby5jb20vYXV0aCIsImh0dHBzOi8vYWNtZWNvcmUtZGV2LnVzLmF1dGgwLmNvbS91c2VyaW5mbyJdLCJpYXQiOjE2NjcyODg5MDAsImV4cCI6MTY2NzI5NjEwMCwiYXpwIjoib0pyaDBrd012Y1ExTDJuUXd5Yjh0b0F0OE95WmlWQ2QiLCJzY29wZSI6Im9wZW5pZCBwcm9maWxlIGVtYWlsIn0.TaUonoP_MvxoKisk02cQkSuWh4RXThwXSo_weg5ThxbVSAHgYGDIsBQkTDmJHrMKgaCq5Nv34eGHJgx2ngPiebyuvTbj5FxRvbd2CEU7yjGnCaynj9J2Q6TFMhbYsYIWsYcPw9gdkh3JH51DbDKbNwFllpDrbGFmn7G1BaQQPNHh7pEht5NwMgcFc7lpjvO3XwfF8d1-HjnLeY3QZc9m0c5b0KJem1oZZ4zmZG8eJEl9wkCwQeesJnCovrEPocCyKoVGcW_To7TKF2s9fwd2xtPXETEmAbQaUF644Tc_7_vHJcKI6KOSXQCU_k1QNujnaf-vmY4xoE_ApujPspWz5g'

//window.vm.config.globalProperties.$cookie.getCookie('session');

const headers = {
    'Content-Type': 'application/json',
    Accept: 'application/json',
    Authorization: 'Bearer ' + accessToken,
    withCredentials: true,
};

function loginToAcmeBackend() {
    try {
        Axios.post(`${serverUrl}/user/login`, {}, { headers });
        console.log('Successfully logged in with Apple Pay');
    } catch (err) {
        console.log('Failed to log in ' + err);
    }
}

export default {
  data() {
    return {
      info: null,
      loading: true,
      errored: false,
      // These need to be set
      nftPrice: 15.59,
      estimatedTransactionCostUSD: 49.59,
      amount: 14.55,
      token_id: 1,
      contract_id: '0x0eAfA761ce742Fb9C9Dc99190cEC726BE85E6Ca4',
      walletId: null,
      token: null,
      ipaddress: '0.0.0.0',
    };
  },
  mounted() {
    let recaptchaScript = document.createElement("script");
    recaptchaScript.setAttribute(
      "src",
      "https://applepay.cdn-apple.com/jsapi/v1/apple-pay-sdk.js"
    );
    document.head.appendChild(recaptchaScript);
    console.log('Apple Pay mounted');

    Axios.defaults.withCredentials = true;

    loginToAcmeBackend();
  },
  methods: {
    showButton() {
      return window.safari !== undefined;
    },
    //Invoked Method
    onApplePayButtonClicked() {
      console.log('onApplePayButtonClicked');
      if (!ApplePaySession) {
        return;
      }
      // Define ApplePayPaymentRequest
      var applePayRequest = this.getApplePayRequest();
      //Get Request Based Wyre Quote
      // Create ApplePaySession
      const session = new ApplePaySession(3, applePayRequest);
      session.onvalidatemerchant = async (event) => {
        // Call your own server to request a new merchant session.
        const merchantSession = this.authapplepay();
        session.completeMerchantValidation(merchantSession);
      };
      session.onpaymentauthorized = (event) => {
        // Define ApplePayPaymentAuthorizationResult
        const result = {
          status: ApplePaySession.STATUS_SUCCESS,
        };
        session.completePayment(result);
        handleApplePayOrder(event);
      };
      session.oncancel = (event) => {
        console.log("Payment cancelled by WebKit");
        // Payment cancelled by WebKit
      };
      session.begin();
    },
    getApplePayRequest() {
      console.log('posting to createApplePayRequest');
      const serverUrl = "https://dev.acmedao.com";
      this.amount = this.nftPrice + this.estimatedTransactionCostUSD;
      const raw = JSON.stringify({
        amount: this.amount.toString(),
      });
      return Axios.post(`${serverUrl}/user/createApplePayRequest`, raw, {
        headers,
      })
        .then((response) => {
          this.info = response;
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
    authapplepay() {
      console.log('posting to createApplePaySession');
      return Axios.post(`${serverUrl}/user/createApplePaySession`, { }, { headers })
        .then((response) => {
          this.info = response;
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
    handleApplePayOrder(e) {
      console.log('handleApplePayOrder');
      // variables
      const orderParams = {
        price: this.nftPrice,
        usdPrice: this.amount,
        fees: this.estimatedTransactionCostUSD,
        token_id: this.tokenId,
        contract_id: this.contractId,
        walletId: this.walletId,
        token: e.payment.token,
        ipaddress: this.ipAddress,
      };
      return Axios.post(`${serverUrl}/user/createApplePayOrder`, orderParams, {
        headers,
      })
        .then((response) => {
          this.info = response;
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
  },
};
</script>

<style>
apple-pay-button {
  --apple-pay-button-width: 140px;
  --apple-pay-button-height: 30px;
  --apple-pay-button-border-radius: 5px;
  --apple-pay-button-padding: 5px 0px;
  display: initial;
}
</style>