# Appendice 3: Usare Bitcoin Regtest

> :information_source: **NOTA:** Questa sezione è stata recentemente aggiunta al corso ed è una bozza iniziale che potrebbe essere ancora in attesa di revisione. Lettore avvisato.

La maggior parte di questo corso presume che tu utilizzi Mainnet o Testnet. Tuttavia, queste non sono le uniche opzioni. Durante lo sviluppo di applicazioni Bitcoin, potresti voler mantenere le tue applicazioni isolate da queste blockchain pubbliche. Per farlo, puoi creare una blockchain da zero utilizzando Regtest, che ha un altro grande vantaggio rispetto a Testnet: puoi scegliere quando creare nuovi blocchi, quindi hai il controllo completo sull'ambiente.

## Avviare Bitcoind su Regtest

Dopo aver [configurato Bitcoin-Core VPS](02_0_Configurare_Bitcoin-Core_VPS.md) o [compilato dal codice sorgente](A2_0_Compilare_Bitcoin_dal_Codice_Fonte.md), sei ora in grado di usare regtest. Per avviare il tuo `bitcoind` su regtest e creare una blockchain privata, usa il seguente comando:



```
$ bitcoind -regtest -daemon -fallbackfee=1.0 -maxtxfee=1.1
```

Gli argomenti `-fallbackfee=1.0 -maxtxfee=1.1` eviteranno l'errore `Fee estimation failed. Fallbackfee is disabled`.

Su regtest, di solito non ci sono abbastanza transazioni quindi bitcoind non può fornire una stima affidabile e, senza di essa, il wallet non creerà transazioni a meno che non venga esplicitamente impostata la tariffa.

### Resettare la Blockchain Regtest

Se lo desideri, puoi in seguito riavviare il tuo Regtest con una nuova blockchain.

I wallet regtest e lo stato della blockchain (chainstate) sono salvati nella sottodirectory regtest della directory di configurazione di Bitcoin:


```
user@mybtc:~/.bitcoin# ls
bitcoin.conf  regtest  testnet3
```


Per iniziare una nuova Blockchain usando regtest, tutto ciò che devi fare è eliminare la cartella `regtest` e riavviare Bitcoind:


```
$ rm -rf regtest
```

## Generare un Wallet Regtest

Prima di generare blocchi, è necessario caricare un wallet utilizzando `loadwallet` o crearne uno nuovo con `createwallet`. Dalla versione 0.21, Bitcoin Core non crea automaticamente nuovi wallet all'avvio.

L'argomento `descriptors=true` crea un wallet descrittore nativo, che memorizza le informazioni scriptPubKey utilizzando descrittori di output. Se è `false`, creerà un wallet legacy, dove le chiavi sono usate per generare implicitamente scriptPubKeys e indirizzi.



```
$ bitcoin-cli -regtest -named createwallet wallet_name="regtest_desc_wallet" descriptors=true
```


## Generare Blocchi

Puoi generare (minare) nuovi blocchi su una catena regtest utilizzando il metodo RPC `generate` con un argomento per quanti blocchi generare. Ha senso utilizzare questo metodo solo su regtest; a causa dell'alta difficoltà è molto improbabile che produca nuovi blocchi su mainnet o testnet:



```
$ bitcoin-cli -regtest -generate 101
[
  "57f17afccf28b9296048b6370312678b6d8e48dc3a7b4ef7681d18ed3d91c122",
  "631ff7b8135ce633c774828be3b8505726459eb65c339aab981b10363befe5a7",
  ...
  "1162dbfe025c7da94ee1128dc26d518a94508f532c19edc0de6bc673a909d02c",
  "20cb2e815c3d42d6a117a204a0b5e726ab641c826e441b5b3417aca33f2aba48"
]
```
> :warning: AVVISO. Nota che devi aggiungere l'argomento `-regtest` dopo ogni comando `bitcoin-cli` per accedere correttamente al tuo ambiente Regtest. Se preferisci, puoi includere un comando `regtest=1` nel tuo file `~/.bitcoin/bitcoin.conf`.

Poiché un blocco deve avere 100 conferme prima che quella ricompensa possa essere spesa, generi 101 blocchi, fornendo accesso alla transazione coinbase dal blocco #1. Poiché questa è una nuova blockchain che utilizza le regole predefinite di Bitcoin, i primi blocchi pagano una ricompensa di blocco di 50 bitcoin. A differenza di mainnet, in modalità regtest solo i primi 150 blocchi pagano una ricompensa di 50 bitcoin. La ricompensa si dimezza dopo 150 blocchi, quindi paga 25, 12,5, e così via...

L'output è l'hash del blocco di ogni blocco generato.

> :book: ***Cos'è una transazione coinbase?*** Una coinbase è la transazione senza input creata quando un nuovo blocco viene minato e data al minatore. È così che nuovi bitcoin entrano nell'ecosistema. Il valore delle transazioni coinbase diminuisce nel tempo. Su mainnet, si dimezza ogni 210.000 blocchi e termina completamente con il 6.929.999° blocco, attualmente previsto per il 22° secolo. A partire da maggio 2020, la ricompensa coinbase è di 6,25 BTC.

### Verificare il Tuo Bilancio

Dopo aver minato blocchi e ottenuto le ricompense, puoi verificare il bilancio sul tuo wallet:



```
$ bitcoin-cli -regtest getbalance
50.00000000
```

## Usare il Regtest

Ora dovresti essere in grado di utilizzare questo saldo per qualsiasi tipo di interazione sulla tua Blockchain privata, come inviare transazioni Bitcoin secondo [Capitolo 4](04_0_Inviare_Transazioni_Bitcoin.md).

È importante notare che per completare qualsiasi transazione, dovrai generare (minare) nuovi blocchi, in modo che le transazioni possano essere incluse.

Ad esempio, per creare una transazione e includerla in un blocco, dovresti prima usare il comando `sendtoaddress`:


```
$ bitcoin-cli -regtest sendtoaddress [address] 15.1
e834a4ac6ef754164c8e3f0be4f34531b74b768199ffb244ab9f6cb1bbc7465a
```


L'output è l'hash della transazione inclusa nella blockchain. Puoi verificare i dettagli utilizzando il `gettransaction`:


```
$ bitcoin-cli -regtest gettransaction e834a4ac6ef754164c8e3f0be4f34531b74b768199ffb244ab9f6cb1bbc7465a
{
  "amount": 0.00000000,
  "fee": -0.00178800,
  "confirmations": 0,
  "trusted": false,
  "txid": "e834a4ac6ef754164c8e3f0be4f34531b74b768199ffb244ab9f6cb1bbc7465a",
  "walletconflicts": [
  ],
  "time": 1513204730,
  "timereceived": 1513204730,
  "bip125-replaceable": "unknown",
  "details": [
    {
      "account": "",
      "address": "mjtN3C97kuWMgeBbxdB7hG1bjz24Grx2vA",
      "category": "send",
      "amount": -15.10000000,
      "label": "",
      "vout": 1,
      "fee": -0.00178800,
      "abandoned": false
    },
    {
      "account": "",
      "address": "mjtN3C97kuWMgeBbxdB7hG1bjz24Grx2vA",
      "category": "receive",
      "amount": 15.10000000,
      "label": "",
      "vout": 1
    }
  ],
  "hex": "020000000f00fe2c7b70b925d0d40011ce96f8991fee5aba9537bd1b6913b37c37b041a57c00000000494830450221009ad02bfeee2a49196a99811ace20e2e7fefd16d33d525884edbc64bf6e2b1db502200b94f4000556391b0998932edde3033ba2517733c7ddffb87d91f6b756629fe201feffffff06a9301a2b39875b68f8058b8e2ad0b658f505e44a67e1e1d039140ae186ed1f0000000049483045022100c65cd13a85af6fcfba74d2852276a37076c89a7642429aa111b7986eea7fd6c7022012bbcb633d392ed469d5befda8df0a6b96e1acfa342f559877edebc2af7cb93401feffffff434b6f67e5e068401553e89f739a3edc667504597f29feb8edafc2b081cc32d90000000049483045022100b86ecc43e602180c787c36465da7fc8d1e8bfba23d6f49c37190c20889f2dfa0022032c3aec3ceefbb7a33c040ef19090cacbfd6bc9c5cd8e94252eb864891c6f34501feffffff4c65b43f8568ce58fc4c55d24ba0742e9878a031fdfae0fadac7247f42cc1f8e0000000049483045022100d055acfce852259dde051dc61792f94277d094c5da96752f925582b8e739868f02205e69add76e6b001073ad6b7df5f32a681fc8513ee0f6e126ee1c2d45149bd91d01feffffff5a72d60b58300974c5d4731e29b437ea61b87b6733bb3ca6ce5548ef8887d05b0000000049483045022100a7f5b2ee656a5a904fb27f982210de6858dfb165777ec969a77ea1c2c82975a4022001a1a563dbc3714047ec855f7aee901e756b851e255f35435e85c2ba7b0abd8401feffffff60d68e9d5650d55bc9e0b2a65ed27a3b9bceac4955760aa1560408854c3b148d000000004948304502210081a6f0c8232c52f3eaca825965077e88b816e503834989be4afb3f44f87eb98202207ae8becb99efe379fb269f477e7bb70d117dcb83e106c53b7addaa9715029da101feffffff63e2239425aad544f6e1157d5ee245d2500d4e9e9daf8049e0a38add6246da890000000049483045022100e0ab1752e8fbb244b63f7dd5649f2222e0dc42fae293b758e0c28082f77560b60220013f72fbe50acf4af197890b4d18fa89094055ed66f9226a6b461cc4ff560f8e01feffffff6aad4151087f4209ace714193dd64f770305dfb89470b79cca538b88253fbbef0000000049483045022100fee4a5f7ec6e8b55bd6aa0e93b5399af724039171d998b926e8095b70953d5f202203db0d4ef9d1bd57aeff0fe3d47d4358ec0559135dac8107507741eef0638279201feffffff7ddbca5854e25e6a2dfeacfe5828267cd1ef5d86e1da573fe2c2b21b45ecd6ce0000000049483045022100bf45241525592df4625642972dbc940ef74771139dd844bc6a9517197d01488c02203c99ca98892cc2693e8fbb9a600962eec84494fb8596acf0d670822624e497c901feffffff8672949de559e76601684c4ac3731599fd965d0c412e7df9f8ec16038d4420a60000000049483045022100b5a9bd3c6718c6bd2a8300bbd1d9de0ff1c5d02aeb6a659c52bb88958e7e3b0302207f710db1ef975c22edf54e063169aae31bbe470166cc0e5c34fd27b730b8e7d001feffffff8e006b0bb8cef2c5c2a11c8c2aa7d3ba01cb4386c7f780c45bc1014142b425f00000000048473044022046dc9db8daeb09b7c0b9f48013c8af2d0a71f688adaa8d91b40891768c852d4a02204fa15da6d58851191344a56c63bf51a540ec03f73117a3446230bb58a8a4bcce01feffffffbad05b8f86182b9b7c9c5aaa9ce3dc8d08a76848e49a2d9b8dcfb0f764bb26ca000000004847304402200682379dc36cb486309eac4913f41ac19638525677edad45ca8d9a2b0728b12f02203fb44f8a46cbc4c02f5699d7d4d9cd810bdf7e7c981b421218ccbcb7b73845f501feffffffd35228fe9ef0a742eacffc4a13f15ed7ba23854e6cb49d5010810ac11b5bdf690000000048473044022030045b882500808bd707f4654becc63de070818c82716310d39576decdd724e3022034d3b41cb5e939f0011bb5251be7941b6077fde5f4eff59afd8e49a2844288f701fefffffff5ae4cbd4ae8d68b5a34be3231cdc88b660447175f39cf7a86397f37641d4aa70000000049483045022100afe16f0de96a8629d6148f93520d690f30126c37e7f7f05300745a1273d7eb7202200933f6b371c4ea522570f3ec2aee9be2b59730b634e828f543bcdb019cf4749901fefffffff633f61ac61683221cc3d2665cf4bcf193af1c8ffe9d3d756ba83cc5eb7643250000000049483045022100ef0b8853c94d60634eff2fc1d4d75872aacb0a2d3242308b7ee256b24739c614022069fe9be8288bdd635871c263c46be710c001729d43f6fbc1350ed1a693c4646301feffffff0250780000000000001976a91464ed7fb2fe0b06f4cad0d731b122222e3e91088a88ac80c5005a000000001976a9142fed0f02d008f89f6a874168e506e2d4f9bcbfb888acd32b0000"
}
```


Tuttavia, devi ora finalizzarla creando blocchi sulla blockchain.
La maggior parte delle applicazioni richiede sei conferme di blocchi per considerare la transazione come irreversibile. Se questo è il tuo caso, puoi minare altri sei blocchi nella tua catena regtest:



```
$ bitcoin-cli -regtest -generate 6
[
  "33549b2aa249f0a814db4a2ba102194881c14a2ac041c23dcc463b9e4e128e9f",
  "2cc5c2012e2cacf118f9db4cdd79582735257f0ec564418867d6821edb55715e",
  "128aaa99e7149a520080d90fa989c62caeda11b7d06ed1965e3fa7c76fa1d407",
  "6037cc562d97eb3984cca50d8c37c7c19bae8d79b8232b92bec6dcc9708104d3",
  "2cb276f5ed251bf629dd52fd108163703473f57c24eac94e169514ce04899581",
  "57193ba8fd2761abf4a5ebcb4ed1a9ec2e873d67485a7cb41e75e13c65928bf3"
]
```



## Test con NodeJS

Quando sei su regtest, sei in grado di simulare casi limite e attacchi che potrebbero verificarsi nel mondo reale, come il double spend.

Come discusso altrove in questo corso, l'uso di librerie software potrebbe darti accesso più sofisticato ad alcuni comandi RPC. In questo caso, [bitcointest di dgarage](https://github.com/dgarage/bitcointest) per NodeJS può essere utilizzato per simulare una transazione da un wallet all'altro; puoi controllare [la loro guida](https://www.npmjs.com/package/bitcointest) per simulazioni di attacchi più specifiche, come il double spend.

Vedi il [Capitolo 18.3](18_3_Accedere_a_Bitcoind_con_NodeJS.md) per le informazioni più aggiornate sull'installazione di NodeJS, poi aggiungi `bitcointest`:



```
$ npm install -g bitcointest
```


Dopo aver installato `bitcointest`, puoi creare un file `test.js` con il seguente contenuto:


```
file: test.js

const { BitcoinNet, BitcoinGraph } = require('bitcointest');
const net = new BitcoinNet('/usr/local/bin', '/tmp/bitcointest/', 22001, 22002);
const graph = new BitcoinGraph(net);

try {

  console.log('Launching nodes...');
  
  const nodes = net.launchBatchS(4);
  const [ n1, n2 ] = nodes;
  net.waitForNodesS(nodes, 20000);

  console.log('Connected!');
  const blocks = n1.generateBlocksS(110);
  console.info('Generated 110 blocks');

  console.log(`n2.balance (before) = ${n2.getBalanceS()}`);

  const sometxid = n1.sendToNodeS(n2, 100);
  console.log(`Generated transaction = ${sometxid}`);
  n1.generateBlocksS(110);
  n2.waitForBalanceChangeS(0);

  const sometx = n2.getTransactionS(sometxid);
  console.log(`n2.balance (after) = ${n2.getBalanceS()}`);


} catch (e) {
  console.error(e);
  net.shutdownS();
  throw e;
}
```


Come mostrato, questo genererà blocchi e una transazione:


```
$ node test.js
Launching nodes...
Connected!
Generated 110 blocks
n2.balance (before) = 0
Generated transaction = 91e0040c26fc18312efb80bad6ec3b00202a83465872ecf495c392a0b6afce35
n2.after (before) = 100

```


## Riepilogo: Usare Bitcoin Regtest

Un ambiente regtest per Bitcoin funziona proprio come qualsiasi ambiente testnet, tranne per il fatto che hai la possibilità di generare blocchi rapidamente e facilmente.

> :fire: ***Qual è il potere del regtest?*** Il più grande potere del regtest è che puoi minare blocchi rapidamente, permettendoti di accelerare la blockchain, per testare transazioni, timelocks e altre funzionalità su cui altrimenti dovresti aspettare a lungo. Tuttavia, l'altro potere è che puoi eseguirlo privatamente, senza connetterti a una blockchain pubblica, permettendoti di testare idee proprietarie prima di rilasciarle nel mondo.

## Cosa viene dopo?

Se hai visitato questa Appendice mentre lavoravi su un'altra parte del corso, dovresti tornarci.

Altrimenti, hai raggiunto la fine! Altre persone che hanno seguito questo corso sono diventate sviluppatori ed ingegneri Bitcoin professionisti, inclusi alcuni che hanno contribuito a [Blockchain Commons](https://www.blockchaincommons.com/). Ti incoraggiamo a fare lo stesso! Basta iniziare a lavorare su un po' di codice Bitcoin con ciò che hai imparato.
