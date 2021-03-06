<template>
  <section id="portfolio-header" v-if="$store.state.portfolio">
    <simple-donut :percent="$store.state.portfolio.changePercent"></simple-donut>
    <div class="ticker">
      <h1 class="underlined">My Portfolio</h1>
      <div class="balance">
        <span class="amount">{{ $formatMoney($store.state.portfolio.balance) }}</span><span class="currency">{{ $store.state.currency }}</span>
      </div>
      <div class="change">
        <div class="label">24h change</div>
        <div :class="['amount', {increase: $store.state.portfolio.changeValue > 0, decrease: $store.state.portfolio.changeValue < 0}]">{{ $formatMoney($store.state.portfolio.changeValue) }}</div>
      </div>
    </div>
    <div class="btn-group">
      <div class="receive-btn" @click="showSendAddressModal">
        <aph-icon name="receive"></aph-icon>
        <p>Receive</p>
      </div>
      <router-link class="send-btn" to="/authenticated/dashboard/send">
        <aph-icon name="send"></aph-icon>
        <p>Send</p>
      </router-link>
    </div>
    <zoom></zoom>
    <address-modal v-if="this.$store.state.showSendAddressModal" :address="getCurrentWalletAddress()" :onDone="hideSendAddressModal"></address-modal>
  </section>
  <section v-else></section>
</template>

<script>
import AddressModal from './modals/AddressModal';
import SimpleDonut from './charts/SimpleDonut';
import Zoom from './Zoom';

let fetchPortfolioIntervalId;

export default {
  components: {
    AddressModal,
    SimpleDonut,
    Zoom,
  },

  methods: {
    fetchPortfolio() {
      this.$store.dispatch('fetchPortfolio');
    },

    getCurrentWalletAddress() {
      return this.$services.wallets.getCurrentWallet().address;
    },

    hideSendAddressModal() {
      this.$store.commit('setShowSendAddressModal', false);
    },

    showSendAddressModal() {
      this.$store.commit('setShowSendAddressModal', true);
    },
  },

  mounted() {
    this.fetchPortfolio();

    fetchPortfolioIntervalId = setInterval(() => {
      this.fetchPortfolio();
    }, this.$constants.intervals.POLLING);
  },

  beforeDestroy() {
    clearInterval(fetchPortfolioIntervalId);
  },
};
</script>

<style lang="scss">
h1.underlined {
  @extend %underlined-header;
}

#portfolio-header {
  display: flex;
  flex-direction: row;
  padding: $space-lg $space-lg 0;
  align-items: center;

  .simple-donut {
    margin: 0 $space-xl 0 0;
    height: toRem(150px);
  }

  #zoom {
    align-self: flex-start;
    margin-left: $space-lg;
  }

  .btn-group {
    display: flex;
    flex-direction: row;
    flex: none;
    width: $right-sidebar-width;

    > * {
      width: auto;
      flex: 1;
    }
  }

  .ticker {
    flex: 1;

    .balance {
      .amount {
        font-size: toRem(45px);
        margin: 0 $space 0 0;
      }

      .currency {
        font-size: toRem(30px);
      }
    }

    .change {
      align-items: center;
      display: flex;
      margin-top: $space;

      .label {
        @extend %small-uppercase-grey-label-dark;
      }

      .amount {
        font-size: toRem(16px);
        margin-left: $space;

        &.increase {
          color: $green;

          &:before {
            content: "+";
            }
        }

        &.decrease {
          color: $red;
        }
      }
    }
  }

  .receive-btn, .send-btn {
    @extend %btn-square;

    height: toRem(160px);

    .aph-icon {
      .fill {
        fill: $dark;
      }
    }
  }

  .send-btn {
    background: $purple;
    margin-left: $space-lg;

    &:hover {
      box-shadow: $box-shadow-lg;
    }

    p {
      color: white;
    }
  }
}
</style>


