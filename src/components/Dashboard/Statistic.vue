<template>
  <div class="statistic-all">
    <div class="statistic-p1">
      <div class="row">
        <div class="item-statistic col-sm-6 col-md-3">
          <div class="text-1">Symbol</div>
          <div class="text-2">{{ tokenSymbol }}</div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-sm-0">
          <div class="text-1">Total Supply</div>
          <div class="text-2">1,000,000,000,000,000 {{ tokenSymbol }}</div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-md-0">
          <div class="text-1">Total Burned</div>
          <div class="text-2">{{ totalBurn }} %</div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-md-0">
          <div class="text-1">Token Address</div>
          <div class="text-2">
            <a
              :href="`https://explorer.fuse.io/token/${mkatAddress}`"
              target="_blank"
              style="color: rgb(5, 0, 111)); font-size: 20px; word-break: break-all"
            >
              view on Fuse Explorer
            </a>
          </div>
        </div>
      </div>
      <div class="row-2 hide-on-mobile"></div>
      <div class="row">
        <div class="item-statistic col-sm-6 col-md-3">
          <div class="text-1">Volume (24h)</div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-sm-0">
          <div class="text-1">Market Cap</div>
          <div class="text-2">
            $
            <span class="card-panel-num"> {{ marketCap}} </span>
          </div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-md-0">
          <div class="text-1">Current Circulating Supply</div>
          <div class="text-2">{{ currentCircularingBalance }} {{ tokenSymbol }}</div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-md-0">
          <div class="text-1">Rewards</div>
          <div class="text-2">{{ contractBNBRewardPool }} USDC</div>
        </div>
      </div>
    </div>
    <div class="statistic-p1 mt-25">
      <div class="row">
        <div class="item-statistic col-sm-6 col-md-3">
          <div class="text-1">10 Bil {{ tokenSymbol }} price</div>
          <div class="text-2">
            <span class="card-panel-num">$ {{ hundredthousandmkatusd * 10000000 }} </span>
          </div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-sm-0">
          <div class="text-1">Total Liquidity Pool</div>
          <div class="text-2">
            <span class="card-panel-num"> $ {{ totalliquiditypoolusd * 10 }} </span>
          </div>
        </div>
        <div class="item-statistic col-sm-6 col-md-3 mt-3 mt-md-0">
          <div class="text-1">FUSE in liquidity pool</div>
          <div class="text-2">{{ totalbnbinpool}} FUSE</div>
        </div>
      </div>
    </div>
    <div class="hidden-input el-input el-input--medium">
      <input id="copy-value-max" type="text" autocomplete="off" class="el-input__inner" />
    </div>
  </div>
</template>

<script>
import { CONTRACT_ADDRESS, BURN_ADDRESS } from "@/constants";
import MetamaskService from "@/MetamaskService";
import { ethers, utils } from "ethers";
import { mapGetters } from "vuex";

export default {
  name: "Statistic",
  props: {
    contract: {
      default: null,
    },
    hundredthousandmkatusd: {
      default: "...",
    },
    totalliquiditypoolusd: {
      default: "...",
    },
    totalbnbinpool: {
      default: "...",
    },
  },
  data() {
    return {
      mkatAddress: CONTRACT_ADDRESS,
      marketCap: "...",
      currentCircularingBalance: "...",
      contractBNBRewardPool: "...",
      provider: null,
      totalBurn: "...",
      tokenSymbol: "",
    };
  },
  computed: {
    ...mapGetters(["signerAddress"]),
    ...mapGetters(["walletProviderType"]),
  },
  async mounted() {
    this.$loading(true);
    this.loadContractInfo();
    this.$loading(false);
  },
  methods: {
    async loadContractInfo() {
      console.log("statistics loading");

      const service = new MetamaskService(await MetamaskService.createWalletProviderFromType(this.walletProviderType));
      await service.initialize();

      this.provider = service.getWeb3Provider();

      this.tokenSymbol = await this.contract.symbol();

      this.marketCap = parseFloat(await this.calculateMarketCap(service) * 100).toFixed(2);

      this.totalBurn = await this.calculateTotalBurnPercent(service);

      this.currentCircularingBalance = parseFloat(
        utils.formatUnits(await this.getCurrentCircularingBalance(), 18)
      ).toFixed(2);

      const cakeTokenContract = await service.getCakeTokenContractInstance(CONTRACT_ADDRESS);
      this.contractBNBRewardPool = parseFloat(
        utils.formatUnits(await cakeTokenContract.balanceOf(CONTRACT_ADDRESS), 18)
      ).toFixed(2);
    },
    async calculateMarketCap(service) {
      const circularingBalance = await this.getCurrentCircularingBalance();

      return utils.formatUnits(await service.getMkatValueInBUSD(circularingBalance), 18);
    },
    async calculateTotalBurnPercent() {
      const { total, zero, burn } = await this.getCircularingBalances();
      return (((zero + burn) * 100) / total).toFixed(2);
    },
    async getCurrentCircularingBalance() {
      const { total, zero, burn, dev, deploy } = await this.getCircularingBalances();
      return total.sub(burn).sub(zero);
    },
    async getCircularingBalances() {
      let total = await this.contract.totalSupply();
      let zero = await this.contract.balanceOf("0x0000000000000000000000000000000000000000");
      let burn = await this.contract.balanceOf(BURN_ADDRESS);

      return { total, zero, burn };
    },
  },
};
</script>

<style scoped></style>
