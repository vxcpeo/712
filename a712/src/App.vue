<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { ArrowUpRight, Plus, Minus, X, Wallet, Menu, ShieldCheck, Clock3, Link2, CircleCheck, ExternalLink, Activity, Layers3, ReceiptText } from 'lucide-vue-next'
import RewardStudio from './components/RewardStudio.vue'
import XStudio from './components/XStudio.vue'
import RewardsApp from './components/RewardsApp.vue'
import { wallets, walletAddress, walletBalance, walletChain, connectWallet, disconnectWallet, discoverWallets } from './wallet'
import { shortAddress, friendlyError, CHAIN_ID, EXPLORER } from './lib'

const studio = ref<InstanceType<typeof RewardStudio>>()
const xStudio = ref<InstanceType<typeof XStudio>>()
const rewardsApp = ref<InstanceType<typeof RewardsApp>>()
const walletDialog = ref<HTMLDialogElement>()
const infoDialog = ref<HTMLDialogElement>()
const infoType = ref('privacy')
const mobileMenu = ref(false)
const walletError = ref('')
const connecting = ref(false)
const activeFaq = ref<number | null>(0)
const onTestnet = computed(() => !!walletAddress.value && walletChain.value === CHAIN_ID)
const walletStatus = computed(() => !walletAddress.value ? 'Not connected' : onTestnet.value ? 'Robinhood testnet' : 'Switch network required')
const faq = [
  { q: 'What is Honoro?', a: 'Honoro connects appreciation for a public X post with an onchain reward. The author-rewards app supports test ETH, a tUSD demo token, a public note and a 90-day claim window. You can follow every reward in a public receipt. A separate wallet pilot is also available.' },
  { q: 'Is this real money?', a: 'No. This release runs on Robinhood Chain testnet. Test ETH and tUSD have no monetary value. tUSD is a mintable demo token, not USDG, not a stablecoin backed by dollars, and not redeemable. There is no investment offering or promise of returns.' },
  { q: 'How does the creator receive a reward?', a: 'In author mode, the creator signs in with X, proves control of a payout wallet and claims a wallet-bound authorization. This service remains disabled until the operator configures and verifies X API access. The separate wallet pilot works now with an explicit recipient address, which the sender must verify directly.' },
  { q: 'What if a reward is never claimed?', a: 'Author rewards expire after 90 days. An unclaimed reward then becomes fully refundable to the original sender, with no protocol fee. The sender requests the refund in their wallet. The older wallet pilot uses its own shorter, selected deadline; each receipt shows the exact expiry.' },
  { q: 'What are the fees?', a: 'Author rewards deduct 3% only when claimed, leaving 97% for the creator. There is no funding fee or refund fee. Network gas is separate. A configured, funded relayer may sponsor claim gas within its budget; otherwise the wallet pays it. The separate wallet pilot charges 0% protocol fees. All current assets are valueless test tokens.' },
  { q: 'Is Honoro affiliated with X or Robinhood Chain?', a: 'No. Honoro is an independent project built on the public Robinhood Chain testnet. X and Robinhood Chain do not endorse this project. The pilot contract is experimental and has not received an independent security audit.' },
]

function openStudio() { mobileMenu.value = false; rewardsApp.value?.show() }
function openWalletPilot() { mobileMenu.value = false; studio.value?.show() }
function openXStudio() { mobileMenu.value = false; xStudio.value?.show() }
function openWallet() { walletError.value = ''; walletDialog.value?.showModal() }
async function selectWallet(id: string) {
  const option = wallets.value.find(w => w.id === id)
  if (!option) return
  connecting.value = true; walletError.value = ''
  try { await connectWallet(option); walletDialog.value?.close() }
  catch (e) { walletError.value = friendlyError(e) }
  finally { connecting.value = false }
}
function openInfo(type: string) { infoType.value = type; infoDialog.value?.showModal() }
onMounted(discoverWallets)
</script>


<template>
    <a class="hz-skip" href="#main">Skip to content</a>
  <div class="hz-stage">
    <div class="hz-bar">
          <a href="#" class="hz-brand" aria-label="Honoro home"><img src="/logo.svg" alt="" />honoro<span class="hz-brand-tag">TESTNET</span></a>
      <div class="hz-bar-ruler" aria-hidden="true"><i></i><i></i><i></i><i></i><i></i><i></i></div>
          <div class="hz-actions"><button class="hz-wallet" @click="openWallet"><Wallet :size="16" />{{ walletAddress ? shortAddress(walletAddress) : 'Connect wallet' }}</button><button class="hz-btn hz-btn-primary hz-btn-sm" @click="openStudio">Open app <ArrowUpRight :size="16" /></button><button class="hz-icon hz-menu" aria-label="Toggle navigation" :aria-expanded="mobileMenu" @click="mobileMenu = !mobileMenu"><X v-if="mobileMenu" :size="22" /><Menu v-else :size="22" /></button></div>
    </div>

    <header class="hz-command" aria-labelledby="terminal-title">
      <div class="hz-command-inner">
        <div class="hz-command-copy">        <div><div class="hz-kicker">THE ONCHAIN REWARD DESK</div><h1 id="terminal-title">Back the signal.</h1><p>Turn a useful X post into a reward on Robinhood Chain.</p></div></div>
                <div class="hz-chain"><Layers3 :size="18" aria-hidden="true" /><span>Robinhood Chain<small>Testnet &middot; Chain ID {{ CHAIN_ID }}</small></span></div>
      </div>
          <nav class="hz-commands" aria-label="Main navigation"><a class="hz-on" href="#main"><Layers3 :size="15" /> Overview</a><a href="#why">Rewards</a><a href="#how">How it works</a><a href="/docs/technical-overview.html" target="_blank" rel="noopener noreferrer">Docs <ArrowUpRight :size="12" /></a></nav>
          <nav v-if="mobileMenu" class="hz-mobile" aria-label="Mobile navigation"><a href="#why" @click="mobileMenu = false">Rewards</a><a href="#how" @click="mobileMenu = false">How it works</a><a href="#roadmap" @click="mobileMenu = false">Build status</a><button @click="openWallet(); mobileMenu = false">{{ walletAddress ? 'Manage wallet' : 'Choose wallet' }}</button></nav>
    </header>

    <main id="main" class="hz-main">
      <aside class="hz-rail">
        <div class="hz-rail-sticky">
                  <article class="hz-wallet-panel">
          <div class="hz-panel-label">WALLET BALANCE <span class="hz-link-status" :class="{ 'hz-up': onTestnet }"><span class="hz-led" :class="{ 'hz-off': !onTestnet }"></span>{{ walletStatus }}</span></div>
          <div class="hz-balance">{{ onTestnet && walletBalance ? walletBalance : '\u2014' }} <small>test ETH</small></div>
          <div class="hz-balance-meta"><span>{{ walletAddress ? shortAddress(walletAddress) : 'Connect a wallet to view your balance' }}</span><button @click="openWallet">{{ walletAddress ? 'Manage wallet' : 'Choose wallet' }} <ArrowUpRight :size="14" /></button></div>
          <p v-if="walletAddress && !onTestnet" class="hz-help">Your wallet will request a network switch before sending a reward.</p>
          <p v-else-if="walletAddress && !walletBalance" class="hz-help">Balance is unavailable. Reconnect your wallet to retry.</p>
          <p v-else class="hz-help">Test ETH is used for rewards and network gas. It has no monetary value.</p>
        </article>
                  <article class="hz-rules-panel">
          <div class="hz-panel-label">AUTHOR REWARD RULES</div>
          <dl><div><dt>Creator receives</dt><dd>97<span>%</span></dd></div><div><dt>Claim window</dt><dd>90 <span>days</span></dd></div><div><dt>Unclaimed refund</dt><dd>100<span>%</span></dd></div></dl>
          <span class="hz-rule-note">3% fee only when claimed &middot; Gas separate</span>
        </article>
        </div>
      </aside>

      <section class="hz-data">
                <article class="hz-activity-panel">
          <div class="hz-panel-label"><Activity :size="15" /> YOUR REWARDS</div>
          <h2>Every reward.<br>One clear receipt.</h2>
          <p>Review sent, received and public activity in the rewards app.</p>
          <button class="hz-link" @click="openStudio">View rewards <ArrowUpRight :size="15" /></button>
        </article>

        <section id="why" class="hz-desk" aria-labelledby="reward-workspace-title">
                <div class="hz-heading"><div><span class="hz-kicker">CHOOSE YOUR ROUTE</span><h2 id="reward-workspace-title">Send a reward</h2></div><span class="hz-note">TEST ASSETS ONLY</span></div>
          <div class="hz-lanes">
                    <article class="hz-lane hz-lane-x"><div class="hz-lane-top"><span class="hz-lane-icon"><Link2 :size="23" /></span><span class="hz-chip">X AUTHOR REWARDS</span></div><h3>A post worth backing.</h3><p>Choose a public X post, add test ETH or tUSD, and leave a public note. The verified author claims to their wallet.</p><div class="hz-lane-assets"><span><b>&#x039E;</b> Test ETH</span><span><b>$</b> tUSD demo</span></div><button class="hz-btn hz-btn-primary hz-full" @click="openStudio">New reward <Plus :size="18" /></button><small>X author lookup and claims require the identity service.</small></article>
                    <article class="hz-lane"><div class="hz-lane-top"><span class="hz-lane-icon"><Wallet :size="23" /></span><span class="hz-chip">DIRECT WALLET PILOT</span></div><h3>Know their wallet?</h3><p>Send test ETH to an explicit recipient. Choose the claim deadline, then share the receipt to claim or refund.</p><div class="hz-lane-facts"><span><CircleCheck :size="15" /> 0% protocol fee</span><span><Clock3 :size="15" /> 1 hour to 30 days</span></div><button class="hz-btn hz-btn-ghost hz-full" @click="openWalletPilot">Open wallet pilot <ArrowUpRight :size="18" /></button><small>Confirm the address directly. This route does not verify X ownership.</small></article>
          </div>
        </section>

            <section id="how" class="hz-seq" aria-labelledby="workflow-title"><div class="hz-heading"><h2 id="workflow-title">From post to payout</h2><span class="hz-note">AUTHOR REWARDS</span></div><div class="hz-seq-steps"><article><span>1</span><div><h3>Find the post</h3><p>Paste a public X URL and check its author.</p></div></article><article><span>2</span><div><h3>Fund the reward</h3><p>Choose an asset, review the fee and approve in your wallet.</p></div></article><article><span>3</span><div><h3>Creator claims</h3><p>The author verifies X and their payout wallet. Unclaimed rewards are refundable after 90 days.</p></div></article></div></section>
            <section id="roadmap" class="hz-rack"><div class="hz-heading"><h2>Build status</h2><span class="hz-chip">TESTNET</span></div><div class="hz-rack-row"><CircleCheck :size="17" /><div><strong>Wallet rewards & receipts</strong><p>Direct rewards, claims and expiry refunds.</p></div></div><div class="hz-rack-row"><ShieldCheck :size="17" /><div><strong>Author verification</strong><p>Built. Availability depends on configured X access.</p></div></div><a class="hz-link" href="/docs/business-plan.html" target="_blank" rel="noopener noreferrer">Read the project plan <ArrowUpRight :size="14" /></a></section>
      </section>

      <aside class="hz-detail">
                <aside id="trust" class="hz-slip-guide"><div class="hz-slip-icon"><ReceiptText :size="26" /></div><h3>Follow the funds.</h3><p>Each reward gets a public receipt with its amount, recipient and expiry.</p><ol><li><span>01</span> Fund in your wallet</li><li><span>02</span> Share the receipt</li><li><span>03</span> Claim or refund onchain</li></ol><a :href="EXPLORER" target="_blank" rel="noopener noreferrer">Open testnet explorer <ExternalLink :size="14" /></a></aside>
            <section id="faq" class="hz-ask"><div class="hz-heading"><h2>Know before you send</h2></div><div class="hz-ask-list"><article v-for="(item, i) in faq" :key="item.q"><button :aria-expanded="activeFaq === i" :aria-controls="`faq-${i}`" @click="activeFaq = activeFaq === i ? null : i">{{ item.q }}<Minus v-if="activeFaq === i" :size="16" /><Plus v-else :size="16" /></button><div v-show="activeFaq === i" :id="`faq-${i}`"><p>{{ item.a }}</p></div></article></div></section>
      </aside>
    </main>

    <footer class="hz-base">
        <footer class="hz-base hz-wrap"><div class="hz-base-top"><a href="#" class="hz-brand"><img src="/logo.svg" alt="" />honoro</a><p>Reward the signal.</p><button @click="openWalletPilot">Wallet pilot <ArrowUpRight :size="14" /></button><button @click="openInfo('social')">@honoro_top <ArrowUpRight :size="14" /></button><a href="/docs/technical-overview.html" target="_blank" rel="noopener noreferrer">Docs <ArrowUpRight :size="14" /></a></div><div class="hz-base-bottom"><span>&copy; {{ new Date().getFullYear() }} Honoro</span><div><button @click="openInfo('privacy')">Privacy</button><button @click="openInfo('terms')">Testnet terms</button><span>Robinhood Chain testnet</span></div></div><p class="hz-base-note">Independent project. Not affiliated with X or Robinhood Chain. Test tokens have no monetary value. Contracts are experimental and unaudited.</p></footer>
    </footer>
  </div>

    <RewardStudio ref="studio" @connect="openWallet" @xRewards="openXStudio" />
  <XStudio ref="xStudio" @connect="openWallet" @walletPilot="openWalletPilot" />
  <RewardsApp ref="rewardsApp" @connect="openWallet" @walletPilot="openWalletPilot" />
    <dialog ref="walletDialog" class="hz-modal" @click="e => { if (e.target === walletDialog) walletDialog?.close() }"><div class="hz-modal-top"><span class="hz-kicker">YOUR ONCHAIN CONNECTION</span><button class="hz-icon" aria-label="Close wallet dialog" @click="walletDialog?.close()"><X :size="22" /></button></div><div class="hz-modal-icon"><Wallet :size="28" /></div><h2>{{ walletAddress ? 'You’re connected.' : 'Bring your wallet.' }}</h2><p>Use an Ethereum-compatible browser wallet to explore Robinhood Chain testnet.</p><template v-if="walletAddress"><div class="hz-address">{{ walletAddress }}</div><p v-if="walletBalance">{{ walletBalance }} test ETH</p><button class="hz-btn hz-btn-primary hz-full" @click="disconnectWallet(); walletDialog?.close()">Disconnect from Honoro</button><small>Disconnecting clears this app’s session. Manage site permissions in your wallet.</small></template><template v-else><button v-for="wallet in wallets" :key="wallet.id" class="hz-wallet-opt" :disabled="connecting" @click="selectWallet(wallet.id)"><Wallet :size="21" /><span>{{ wallet.name }}</span><ArrowUpRight :size="18" /></button><div v-if="!wallets.length" class="hz-no-wallet"><p>No browser wallet detected.</p><a class="hz-btn hz-btn-primary hz-full" href="https://metamask.io/download" target="_blank" rel="noopener noreferrer">Get an Ethereum wallet <ExternalLink :size="17" /></a><small>Install a wallet, then reload this page. Never share your recovery phrase or private key.</small></div></template><p v-if="walletError" role="alert" class="hz-feedback hz-bad">{{ walletError }}</p><div class="hz-modal-note"><ShieldCheck :size="15" /> Connecting does not move funds.</div></dialog>
  <dialog ref="infoDialog" class="hz-modal hz-modal-info" @click="e => { if (e.target === infoDialog) infoDialog?.close() }"><div class="hz-modal-top"><span class="hz-kicker">A LITTLE CLARITY</span><button class="hz-icon" aria-label="Close information" @click="infoDialog?.close()"><X :size="22" /></button></div><template v-if="infoType === 'privacy'"><h2>Privacy, plainly.</h2><p>The wallet pilot has no account database, analytics trackers or email collection. Wallet access requires your approval. The optional X service uses an HTTP-only session cookie and stores your public X identity temporarily in server memory.</p><p>Transactions, wallet addresses, amounts and post IDs are public onchain. The RPC provider and your wallet receive requests needed to operate the app. External links use their own privacy policies.</p><p>Public notes, post IDs and transfers are recorded onchain. Do not include private information. If enabled, X login requests only account and post read access; OAuth access tokens are discarded after identity lookup.</p></template><template v-else-if="infoType === 'terms'"><h2>A playground<br>with clear rules.</h2><p>Honoro is experimental software on Robinhood Chain testnet. Test ETH has no monetary value. Use only test funds.</p><p>Rewards are locked until the recipient claims or the expiry allows the sender to request a refund. The wallet pilot does not verify X ownership. Author mode relies on a trusted verification service; its signing key can authorize payouts. Its 3% claim fee and 90-day expiry are shown before funding.</p><p>Contracts are unaudited. Network availability and successful transactions are not guaranteed. No financial returns, affiliation, or production readiness are claimed.</p></template><template v-else><h2>Good voices.<br>Coming together.</h2><p>Our public identity is <strong>@honoro_top</strong> on X.</p><p>Honoro is an independent testnet prototype on Robinhood Chain: test assets only, no token, and no independent security audit.</p><a href="https://x.com/honoro_top" target="_blank" rel="noopener noreferrer" class="hz-btn hz-btn-primary hz-full">Open @honoro_top on X <ArrowUpRight :size="17" /></a></template></dialog>
</template>
