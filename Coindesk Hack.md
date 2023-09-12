# Lessons of a $37M Attack: How a Ukrainian Payment Processor Was Hacked

## CoinsPaid, a crypto payment processing company with Ukrainian roots, fell a victim to a social engineering attack, thought to have emanated from Lazarus, a North Korean hacking group.

Exploiting DeFi protocols has long become  [crypto’s most popular type of crime](https://www.coindesk.com/business/2022/07/27/defi-has-become-crypto-crimes-main-arena-crystal-blockchain-says/), while traditional exchange hacks have become far less frequent. But cybercriminals haven’t lost all interest in the good old digital robbery.

The recent hack of a crypto payment processor CoinsPaid shows that the most industrious cybercriminal groups in the world are still willing to spend formidable resources on breaking into centralized entities.

CoinsPaid, a Ukrainian firm registered in Estonia,  [reported](https://coinspaid.com/tpost/0zx28tmj51-press-release)  being hacked on July 22, with estimated crypto losses of $37.3 million. According to the CEO Max Krupyshev, the company ended up refunding clients from its own funds. Those customers likely include online casinos, which  [according to a Blockchain Intelligence Group](https://blockchaingroup.io/coinspaid-likely-new-victim-in-a-series-of-hacks-by-the-lazarus-group/?utm_campaign=Blogs&utm_content=258255679&utm_medium=social&utm_source=twitter&hss_channel=tw-3146954264), are widespread users of CoinsPaid.

In a detailed  [explanation](https://coinspaid.com/tpost/k4r6jt90p1-the-coinspaid-hack-explained)  of the incident published Monday, CoinsPaid said that, judging by the thieves’ on-chain behavior, they were very likely the North Korean Lazarus Group or affiliated with it. To siphon money out of CoinsPaid, the attackers used wallets that included the one spotted in another recent attack attributed to Lazarus – the  [Atomic Wallet hack](https://www.coindesk.com/tech/2023/06/05/atomic-wallet-users-hacked-for-35m-worth-of-bitcoin-ether-tether-and-other-tokens/)  in June, Blockchain Intelligence Group  [wrote](https://blockchaingroup.io/coinspaid-likely-new-victim-in-a-series-of-hacks-by-the-lazarus-group/?utm_campaign=Blogs&utm_content=258255679&utm_medium=social&utm_source=twitter&hss_channel=tw-3146954264).

The attackers had been targeting CoinsPaid for months before finally pulling off the theft, CoinsPaid said. Fishing and social engineering attempts started in March, including a request from someone posing as a fellow Ukrainian crypto processing startup, who was asking CoinsPaid developers about the firm’s technical infrastructure, the blog post said. The attackers also tried to bribe CoinsPaid staff and engaged in distributed denial-of-service (DDOS) attacks aimed at the company’s servers.

## Fishing for the gullible employees

Then, in July, several employees received lucrative job offerings from LinkedIn accounts posing as recruiters from other crypto companies, including the exchange Crypto.com. “For instance, some of our team members received job offers with compensation ranging from 16,000-24,000 USD a month,” the blog post said.

After making an initial contact, the fake recruiters asked the employees to install JumpCloud, a platform for user authentication that was reportedly also  [hacked](https://techcrunch.com/2023/07/20/north-korea-backed-hackers-breached-jumpcloud-to-target-cryptocurrency-clients/)  by Lazarus in July, or other software, presumably to perform a test task. Several employees took the bait and installed malicious software, after which the attackers got access to CoinsPaid’s infrastructure.

During late-European hours on a Friday night on July 21, the attackers got access to CoinsPaid’s blockchain node and requested a large withdrawal of Tron-based USDT, bitcoin and several ERC20 tokens running on the Ethereum blockchain, Pavel Kashuba, CoinsPaid chief financial officer, told CoinDesk in an interview. The active phase of the attack took about four hours 23 minutes, he said.

While the thieves got free access to the company’s servers, they did not compromise the private keys for CoinsPaid’s wallets, CEO Max Krupyshev told CoinDesk: “As soon as we switched off our servers, the transfers stopped.” He added that, when the firm spun off new wallets with the same keys, those weren’t drained, confirming that the keys were safe.

## Can’t block this

The firm lost money anyway. Most of the stolen funds, in a form of USDT on the Tron blockchain, were swapped for the USDT on Avalanche via cross-chain bridges and then sent to a decentralized exchange SwftSwap, Krupyshev said. Attackers also used decentralized exchanges Uniswap and SunSwap, as well as centralized exchanges Binance, Huobi, Kucoin, Bybit, Bitget and MEXC, according to the post-mortem blog post.

Bitcoin was laundered via the Sindbad mixer, which, according to the blockchain intel firm Elliptic, is the  [most popular mixer for North Korean hackers](https://www.coindesk.com/business/2023/02/13/sanctioned-mixer-blender-re-launched-as-sinbad-elliptic-says/).

CoinsPaid said, although they notified the centralized exchanges as soon as they saw funds moving there, labeling crime-related addresses and taking action by exchanges is a process too slow to keep up with the hackers, who were cashing out in a matter of minutes.

Kashuba expressed frustration that law enforcement agencies go slowly in convincing exchanges to freeze criminal accounts. “You need to block the money but that money is already gone,” he said.

The bottom line is exchanges need to pay attention to digital hygiene and adequate training for the staff, Kashuba said. And that goes for all kinds of cybercrime.