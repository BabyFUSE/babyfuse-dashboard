<template>
  <div class="moonkat-sidebar">
    <div class="logo-and-buy">
      <div class="text-center">
        <router-link :to="{ name: 'Home' }">
          <img src="@/assets/images/beaglecakeLogo.png" class="logo" alt="BabyFuse" />
        </router-link>
      </div>
      <div class="mrat-text hide-on-mobile" style="color: #9dc64e">BABYFUSE</div>
      <div class="mrat-desc">A new way to earn fUSD</div>
      <div class="button-buy-mrat hide-on-mobile">
        <a
          href="https://app.next-gen.finance/swap?outputCurrency=0xe260ed9cF92933D1363A7d5581E2Bf8eA2EA668f"
          target="_blank"
          class="button-custom-new button-sidebar"
          ><i class="fa fa-shopping-cart"></i>
          <div class="buyCard">
            <div class="buyCardBody">
              <img src="@/assets/images/shop.png" alt="" />
              <span class="buyCake">BUY BABY FUSE</span>
            </div>
          </div>
        </a>
      </div>
    </div>
    <div class="address-info">
      <div class="text-1 hide-on-mobile">Address information</div>
      <div class="text-2 hide-on-mobile">Your address</div>
      <div id="myAddress" class="text-3 hide-on-mobile">
        <div class="d-flex">
          <span id="my-address" ref="myAddr" class="truncate">{{ signerAddress }}</span>
          <input id="testing-code" type="hidden" :value="signerAddress" />
        </div>
      </div>
      <div class="text-4 hide-on-mobile">
        <span id="copy-address" @click="copyAddress()">
          <span style="margin-right: 3px"> <i class="fa fa-clone"></i></span> Copy address
        </span>
        <a id="bscscan" :href="`https://explorer.fuse.io/address/${signerAddress}`" target="_blank" style="margin-left: 10px"
          ><span style="margin-right: 3px"><i class="fa fa-clone"></i></span> View on Fuse Explorer
        </a>
      </div>
      <div class="text-2">Your Baby Fuse balance:</div>
      <div class="text-3">
        {{ tokenSymbol }}
        <span> {{ myMkatBalance }} </span><br />
        ({{ myMkatBalanceInBUSD }}$)
      </div>
    </div>
    <div class="address-info sidebar-menu">
      <div class="sidebar-menu-item sidebar-menu-item-nonlast">
        <a href="">
          <img src="@/assets/images/home.png" alt="" />
          <span>Homepage</span>
        </a>
      </div>

      <div
        class="sidebar-menu-item sidebar-menu-item-nonlast"
        @click="redirectTo(`https://explorer.fuse.io/token/${CONTRACT_ADDRESS}`)"
      >
        <a>
          <img src="@/assets/images/charts.png" alt="" />
          <span>Contract</span>
        </a>
      </div><div
        class="sidebar-menu-item sidebar-menu-item-nonlast"
        @click="redirectTo(`https://docs.babyfuse.finance`)"
      >
        <a>
          <img src="@/assets/images/book.png" alt="" />
          <span>Whitepaper</span>
        </a>
      </div>
      <div
        class="sidebar-menu-item"
        @click="redirectTo(`https://dexscreener.com/fuse/0xF6597B6fdd136fA643d583695Ec6Db99986B4A4d`)"
      >
        <a>
          <img src="@/assets/images/chartsup.png" alt="" />
          <span>Chart</span>
        </a>
      </div>
      <div
        class="sidebar-menu-item"
        @click="redirectTo(`https://explorer.fuse.io/address/0xe260ed9cF92933D1363A7d5581E2Bf8eA2EA668f/write-contract`)"
      >
        <a>
          <img src="@/assets/images/claim.png" alt="" />
          <span>Manual Claim</span>
        </a>
      </div>
    </div>
    <div class="sidebar-locale">
      <img src="@/assets/images/globus.png" alt="" />
      <span>English</span>
    </div>
    <div class="button-logout-wrapper hide-on-mobile" style="cursor: pointer" @click="logout()">
      <a target="_blank" class="button-custom-new button-sidebar"
        ><i class="fa fa-sign-out"></i>
        <div class="buyCard">
          <div class="buyCardBody">
            <img src="@/assets/images/logout.png" alt="" />
            <span class="logoutText">LOGOUT</span>
          </div>
        </div>
      </a>
    </div>
  </div>
</template>

<script>
import { CONTRACT_ADDRESS } from "@/constants.ts";
import { ContractFactory, ethers } from "ethers";
import { mapGetters } from "vuex";
import MetamaskService from "@/MetamaskService";
import { WalletType } from "../../MetamaskService";

export default {
  name: "Sidebar",
  props: {
    contract: {
      type: ContractFactory,
      required: true,
    },
  },
  data() {
    return {
      canCopy: false,
      myMkatBalance: "...",
      myMkatBalanceInBUSD: "0.00",
      mkatContract: null,
      service: null,
      tokenSymbol: "",
    };
  },
  computed: {
    ...mapGetters(["signerAddress"]),
    ...mapGetters(["walletProviderType"]),
    CONTRACT_ADDRESS() {
      return CONTRACT_ADDRESS;
    },
  },
  watch: {
    contract() {
      this.updateUserBalances();
    },
  },
  async mounted() {
    this.canCopy = !!navigator.clipboard;

    this.service = new MetamaskService(await MetamaskService.createWalletProviderFromType(this.walletProviderType));
    await this.service.initialize();

    this.mkatContract = this.service.getTokenContractInstance();
    this.tokenSymbol = await this.mkatContract.symbol();

    this.updateUserBalances();

    setTimeout(this.updateUserBalance, 600000);
  },
  methods: {
    redirectTo(redirectUrl) {
      window.open(redirectUrl, "_blank");
    },
    async updateUserBalances() {
      this.myMkatBalance = ethers.utils.formatUnits(await this.mkatContract.balanceOf(this.signerAddress), 18);
      this.myMkatBalanceInBUSD = parseFloat(
        ethers.utils.formatUnits(
          await this.service.getMkatValueInBUSD(ethers.utils.parseUnits(this.myMkatBalance, 18)),
          18
        )
      ).toFixed(2);
    },
    async copyAddress() {
      const address = this.$refs.myAddr;
      await navigator.clipboard.writeText(address.innerHTML);
      alert("Copied!");
    },
    async logout() {
      if (this.walletProviderType == WalletType.WalletConnect) this.service.walletProvider.disconnect();

      this.$store.commit("logout");
      this.$router.replace("/connect-wallet");
    },
  },
};
</script>

<style scoped>
.buyCardBody {
  display: flex;
  align-items: center;
  gap: 5px;
}
.buyCardBody img {
  padding-top: 2px;
}

.buyCard {
  display: inline-block;
}

.buyCake {
  font-weight: 500;
  padding-top: 6px;
  font-family: "Arial";
  font-size: 18px;
  color: #151505;
}

.sidebar-menu {
  display: flex;
  flex-direction: column;
  align-items: start;
  padding: 10px 0 10px;
  font-size: 18px;
}
.sidebar-menu-item {
  width: 100%;
  cursor: pointer;
}
.sidebar-menu-item:hover {
  background-color: #200069;
  transition: color 0.3s linear;
}
.sidebar-menu-item:nth-child(1) {
  margin-top: -10px;
}
.sidebar-menu-item:hover:nth-child(1) {
  border-radius: 20px 20px 0px 0px;
}
.sidebar-menu-item:nth-last-child(1) {
  margin-bottom: -10px;
}
.sidebar-menu-item:hover:nth-last-child(1) {
  border-radius: 0px 0px 20px 20px;
}
.sidebar-menu-item a {
  display: flex;
  color: #151505;
  align-items: center;
  gap: 10px;
  padding: 8px 0;
  padding-left: 40px;
}
.sidebar-menu-item a span {
  font-weight: 500;
  cursor: pointer;
}
.sidebar-menu-item:hover a span {
  transition: color 0.3 linear;
  color: #9dc64e;
}
.sidebar-menu-item img {
  height: 20px;
}
.sidebar-menu-item-nonlast {
  border-bottom: 1px solid #000000;
}
.sidebar-locale {
  display: flex;
  align-items: center;
  background-color: #9dc64e;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  margin: 15px 0px;
  color: #151505;
  font-family: "Arial";
  font-size: 18px;
  border-radius: 20px;
  padding: 8px 15px;
  padding-left: 40px;
  gap: 10px;
}
.sidebar-locale:hover {
  background-color: #200069;
  transition: color 0.3s linear;
}
.sidebar-locale span {
  font-weight: 500;
  cursor: pointer;
}
.sidebar-locale:hover span {
  transition: color 0.1s linear;
  color: #9dc64e;
}
.sidebar-locale img {
  height: 20px;
}

.logoutText {
  color: #151505;
  font-weight: 500;
  font-family: "Arial";
  font-size: 18px;
  padding-top: 4px;
}

a#Fuse Explorer{
  color: #151505;
}
a#bscscan:hover {
  color: #fff;
}
</style>
