<template>
  <div id="app" class="flex flex-column justify-center items-center">
    <a-form v-if="wallet" @submit="spend">
      <div class="balance mb2 flex flex-column">
        <small>Wallet balance</small>
        {{ balance }}
      </div>
      <a-form-item label="Spend to" layout="vertical">
        <a-input size="large" v-model="spendTo">
          <a-tooltip slot="suffix" title="Copy to clipboard">
            <a-icon
              @click="clickToCopy('Spend to', spendTo)"
              class="copy-to-clipboard"
              type="copy"
              style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </a-input>
      </a-form-item>
      <a-form-item label="Amount" layout="vertical">
        <a-input size="large" v-model="amount"></a-input>
      </a-form-item>
      <a-form-item class="flex justify-center items-center">
        <a-button type="primary" html-type="submit">Spend</a-button>
      </a-form-item>
    </a-form>
    <a-form v-if="!wallet" class="form" @submit="handleSubmit">
      <a-form-item label="Secret key" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
        <a-input size="large" v-model="secretKey">
          <a-tooltip slot="suffix" title="Copy to clipboard">
            <a-icon
              @click="clickToCopy('Secret key', secretKey)"
              class="copy-to-clipboard"
              type="copy"
              style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </a-input>
      </a-form-item>
      <a-form-item label="Public key" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
        <a-input size="large" v-model="publicKey">
          <a-tooltip slot="suffix" title="Copy to clipboard">
            <a-icon
              @click="clickToCopy('Public key', publicKey)"
              class="copy-to-clipboard"
              type="copy"
              style="color: rgba(0,0,0,.45)" />
          </a-tooltip>
        </a-input>
      </a-form-item>
      <a-form-item :wrapper-col="{ span: 12, offset: 5 }">
        <a-button @click="generate" class="mr1">Generate new</a-button>
        <a-button type="primary" html-type="submit">Connect</a-button>
      </a-form-item>
    </a-form>
    <footer-logo />
  </div>
</template>

<script lang="ts">
import { generateKeyPair } from '@aeternity/aepp-sdk/es/utils/crypto';
import Wallet from '@aeternity/aepp-sdk/es/ae/wallet';
import MemoryAccount from '@aeternity/aepp-sdk/es/account/memory';
import { createComponent } from '@vue/composition-api';
import { ref } from '@vue/composition-api';
import copyToClipboard from './utils/copyToClipboard';
import FooterLogo from './components/FooterLogo.vue';

export default createComponent({
  name: 'app',
  components: {
    FooterLogo,
  },
  setup(props, { root, parent }) {
    const secretKey = ref('');
    const publicKey = ref('');
    const spendTo = ref('');
    const wallet = ref(null);
    const balance = ref(0);
    const amount = ref(0);

    /**
     * Generate key pairs.
     */
    function generate() {
      const keypair = generateKeyPair();
      secretKey.value = keypair.secretKey;
      publicKey.value = keypair.publicKey;
    }
    generate();

    /**
     * Handle submit.
     */
    async function handleSubmit() {
      const w = await Wallet({
        url: 'https://sdk-testnet.aepps.com/',
        accounts: [MemoryAccount({ keypair: { secretKey: secretKey.value, publicKey: publicKey.value } })],
        address: publicKey.value,
        onTx: confirm,
        onChain: confirm,
        onAccount: confirm,
        onContract: confirm,
      });
      wallet.value = w;
      balance.value = await w.getBalance(publicKey.value);
    }

    /**
     * Clicked on copy-to-clipboard icon.
     */
    function clickToCopy(title: string, str: string) {
      (parent as any).$notification.open(
        {
          message: title,
          description: 'Successfully copied to clipboard.',
        },
      );
      copyToClipboard(str);
    }

    /**
     * Spend some ae.
     */
    function spend() {
      (wallet as any).value.spend(spendTo.value, amount.value, {});
    }

    return {
      secretKey,
      publicKey,
      generate,
      handleSubmit,
      clickToCopy,
      wallet,
      balance,
      amount,
      spend,
      spendTo,
    };
  },
});
</script>

<style lang="scss">
#app {
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  position: fixed;
  display: flex;

  .form{
    width: 800px;
  }

  .form-send{
    width: 600px;
  }

  .copy-to-clipboard{
    cursor: pointer;
  }

  .balance{
    font-size: 72px;
    font-weight: 100;
    line-height: 1;

    small{
      font-size: 14px;
    }
  }
}
</style>
