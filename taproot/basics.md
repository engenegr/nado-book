\newpage
## Taproot и подписи Шнорра {#sec:taproot_basics}

\EpisodeQR{2}

В этой главе мы впервые познакомимся с деревом Меркла, которое скрывает все различные условия расходования монет до тех пор, пока они не будут использованы. Это называется MAST. Далее мы объясним, как подписи Шнорра позволяют нам скрыть сам MAST, что еще больше повышает конфиденциальность. Мы рассмотрим более ранние предложения по MAST, в которых не было преимуществ подписей Шнорра, что, в свою очередь, хорошо иллюстрирует всю мощь Taproot. Наконец, мы отмечаем некоторые интересные вещи, которые позволяет Taproot.

### Абстрактные синтаксические деревья Меркла

В главе @sec:miniscript рассказывается, как программный форк Pay-to-Script-Hash (P2SH) 2012 года позволил скрыть содержимое скрипта до тех пор, пока монеты не будут израсходован. С точки зрения приватности это намного лучше, чем сразу выкладывать скрипт в блокчейн. Однако, что прискорбно, когда вы тратите монеты, все ограничения, которые были наложены на транзакцию, становятся видны всем.

В примере из той главы описывается скрипт, согласно которому вам нужно либо попросить маму подписать транзакцию, либо подождать два года, чтобы потратить монеты самостоятельно. Один из потенциальных недостатков отображения содержимого всего скрипта, включая резервное условие - злоумышленник узнает, что ему достаточно украсть только _ваши_ ключи, а затем через два года он може свободно тратить монеты. В самый первый раз, когда вы добавляете монеты в этот конкретный кошелек, злоумышленник ничего не знает о том, как их потратить (благодаря P2SH). И если вы воспользуетесь этим кошельком только один раз в жизни и потратите все сразу, то злоумышленник не узнает о дополнительном условии, пока не станет слишком поздно. Однако если вы используете кошелек более одного раза, то как только вы сделаете из него первую трату, вы обнародуете, что существует резервное условие. С этого момента вы под угрозой.

Этот сценарий в образовательных целях сделан немного гротескным, но смысл в том, что когда вы тратите деньги, было бы неплохо раскрывать только то решение, которое вы используете, а не все возможные варианты.

Именно здесь на помощь приходят абстрактные синтаксические деревья Меркла (Merklized Abstract Syntax Trees или MAST^[<https://bitcoinops.org/en/topics/mast/>]).

Дерево Меркла^[Не путать с деревом Меркеля: <https://www.reddit.com/r/ProgrammerHumor/comments/qzwjm3/please_dont_confuse_these_two/>] — это дерево хэшей скриптов, и оно определяет различные способы потратить биткоин. Представьте его вверх ногами, с корнем вверху и листьями внизу. Итак, если у вас есть список из четырех условий, вы строите дерево, начиная с этих четырех условий (или хэшей) внизу. Затем вы объединяете их в пары, в результате чего получается две группы хэшей. Далее они опять хэшируются вверх по дереву, пока не останется один хэш наверху, корень, который вы и публикуете: он используется для генерации адреса, на который отправляются монеты.

Позже, когда вы захотите их потратить, вы говорите: «Вот часть дерева, которую я хочу потратить», а затем используете этот скрипт. Вы также передаете хэш скрипта соседа, потому что скрипты идут парами; также вы даете хэш каждой другой точки дерева. Раскрывая скрипт и хэш соседнего листа, вы доказываете, что не изменяли скрипт. Это называется доказательством Меркла, которое мы более подробно объясняли в главе @sec:utreexo.

![Дерево Меркла для MAST. Чтобы доказать существование скрипта 1, вам необходимо предоставить доказательство Меркла, состоящее из трех отмеченных элементов.](taproot/mast.svg)

В нашем примере с четырьмя условиями дерево имеет три уровня в высоту. Поскольку вы уже раскрываете используемый скрипт, нет необходимости раскрывать его хэш. Остается открыть только два хэша: соседний лист вашего скрипта внизу и сосед его родителя (где дерево имеет ширину в два хэша). Сам родительский хэш не нужно раскрывать, потому что любой может вычислить его из двух предоставленных вами хэшей. В зависимости от скриптов для этого обычно требуется меньше данных, чем для исходных четырех скриптов, а все остальное вы держите в секрете. Имея 1024 скрипта, вам нужно поместить в блокчейн только тот, который вы использовали, плюс девять хэшей.

Деревья Меркла сейчас широко распространены: они используются в блоках, для обмена файлами, в BitTorrent и т. д. Их использование позволяет вам делиться только теми частями записанной в виде дерева сущности, которые вам нужны. В контексте Биткоина это может быть скрипт, который вы используете, чтобы потратить монеты, тогда как в контексте BitTorrent это может быть конкретное двухсекундное видео; ваш компьютер может принимать множество коротких фрагментов и уверенно хранить их на жестком диске, зная, что вы скачиваете действительно кусок фильма, а не какие-то мусорные данные. В обоих сценариях остальное остается хэшированным, и вы просто добавляете дополнительные данные, чтобы доказать, что они — скрипт или фрагмент видеофайла — находятся где-то в этом дереве.

Помимо сохранения секретности, использование MAST также менее затратно, потому что вам не нужно включать все возможные скрипты в блокчейн. Это особенно актуально для больших деревьев с большим количеством скриптов, которые могут понадобиться для «смарт-контракта». Блокчейн — дефицитный ресурс, а включение всего стоит дорого. Потратив монеты, вы должны раскрыть скрипт, который требует комиссий. С момента появления P2SH скрипт открывает именно тот, кто тратит. Без MAST тот, кто тратит монеты, должен был бы раскрыть все возможные скрипты, но с MAST ему нужно раскрыть только тот скрипт, который фактически использовался.

#### Скрытие MAST

Хотя деревья Меркла — хорошее решение, было бы еще лучше, если бы вы могли скрыть сам MAST. В идеале никто не должен знать даже то, что вы используете MAST.

Вернемся к предыдущему примеру про вас и вашу маму. В этом смарт-контракте, если вы и ваша мама соглашаетесь потратить деньги, вам не нужно ждать два года. В большинстве смарт-контрактов, какими бы сложными ни были различные возможные скрипты, когда все участники соглашаются потратить деньги, они могут также обойтись без скриптов и просто создать единую совместную подпись.

Было бы неплохо иметь способ выразить это, используя только подпись — без скриптов или всего дерева. Это можно сделать, подправив ваш открытый ключ, как описано в BIP 341.^[Taproot (SegWit v1): <https://en.bitcoin.it/wiki/BIP_0341>] Вместо того, чтобы сказать: «Отправьте эти монеты на мой открытый ключ», вы бы сказали: «Отправьте их на мой открытый ключ, плюс на открытый ключ моей мамы, плюс на этот MAST-ключ».

<!-- Описание BIP 341 намеренно многословно, чтобы предотвратить наложение QR-кода -->

Эта подгонка ключей немного сложнее, чем просто взять и добавить их все, из-за множества криптографических тонкостей, но по сути, в результате вы можете складывать ключи, а также можете складывать подписи, и это выглядит для внешнего мира, как будто это просто обычная подпись. В результате внутри можно спрятать много интересного — но без подписей Шнорра этот процесс крайне сложен.

### Шнорр {#sec:schnorr}

Chapter @sec:libsecp talks about libsecp256k1, and in May 2021, BIP 340^[Schnorr: <https://en.bitcoin.it/wiki/BIP_0340>] support was merged into libsecp256k1. This added Schnorr signatures to Bitcoin Core.

В главе @sec:libsecp рассказывается о библиотеке libsecp256k1, а в мае 2021 года в нее добавилась поддержка BIP 340^[Шнорр: <https://en.bitcoin.it/wiki/BIP_0340>]. Это добавило в Bitcoin Core подписи Шнорра.

Цифровые подписи Шнорра были впервые созданы немецким математиком Клаусом-Петером Шнорром. Он создал алгоритм подписи Шнорра, который затем запатентовал. Этот алгоритм мог бы очень пригодиться Биткоину, равно как и другим проектам с открытым исходным кодом, которые появились еще раньше, но из-за патента людям пришлось найти другой способ, чтобы воспользоваться преимуществами этих подписей.

So a bunch of lawyers, engineers, and cryptographers joined forces and tried to figure out if there was a way to maim Schnorr’s algorithm so far that it would legally not fall under the patent, but still work. The result was a signature algorithm called Elliptic Curve Digital Signature Algorithm (ECDSA), which is the elliptic curve algorithm that Bitcoin currently uses and that the libsecp library implements. Although both Schnorr and ECSDA use public and private keys to create digital signatures, the latter involves a slightly more complicated process.

Поэтому множество юристов, инженеров и криптографов объединили усилия и попытались выяснить, есть ли способ покалечить алгоритм Шнорра настолько, чтобы он юридически не подпадал под действие патента, но все же работал. Результатом стал алгоритм подписи под названием Алгоритм цифровой подписи на эллиптических кривых (Elliptic Curve Digital Signature Algorithm или ECDSA), который представляет собой алгоритм для той эллиптической кривой, которую сейчас использует Биткоин и который реализуется в библиотеке libsecp. Хотя оба алгоритма - и Шнорра, и ECSDA - используют открытые и закрытые ключи для создания цифровых подписей, последний подразумевает несколько более сложный процесс.

Хотя ECDSA представляет собой изрядно запутанную версию подписей Шнорра, он был стандартизирован в 2005 году, и внедрен не менее чем в дюжину криптографических библиотек, включая OpenSSL. Итак, когда Сатоши нужно было выбрать криптографическую кривую для Биткоина, он выбрал ECDSA именно потому, что она не была запатентована и уже присутствовала в OpenSSL.

В целом, алгоритм Шнорра проще, чем ECDSA. Они используют одну и ту же эллиптическую кривую, но для создания подписи с ней приходится производить несколько иные вычисления. Так что это также означает, что правки для алгоритма Шнорра не так сложны, как, скажем, для первоначальной версии libsecp.

Первоначальная версия libsecp должна была представлять эллиптическую кривую, включая все операции, которые вы можете выполнять с кривой, такие как сложение и умножение, а затем реализовать алгоритм подписи ECDSA. Но для новой библиотеки вам просто нужно выполнить алгоритм подписи Шнорра, после чего можно пропустить или удалить всю ненужную математику.

Переход от ECDSA к алгоритму Шнорра это не очень большое изменение; при этом не меняется и не вводится совершенно новая эллиптическая кривая. Скорее, это переход к другому — и более простому — способу подписи.

Чем меньше изменений разработчикам придется внести в криптографическую библиотеку, тем лучше. Это означает меньшее количество мест, где могут быть допущены критические ошибки. Это также означает, что сообществу нужно проводить меньше проверок кода. На кону стоит почти триллион долларов, и любая ошибка, связанная с цифровыми подписями в Биткоине, может иметь катастрофические последствия, поэтому важность того, что требуется лишь простенькое изменение, трудно переоценить.

Вдобавок ко всему, поскольку алгоритм Шнорра добавляется как софтфорк, его использование полностью добровольно. ECDSA никуда не денется.

### Но почему Шнорр?

Simplicity is great, but all the hard work for the more complicated ECDSA had already been done. Why bother changing things? Even before Taproot, people wanted to add Schnorr because of all the things it enabled. But at some point, Bitcoin Core contributor and former Blockstream CTO Gregory Maxwell came up with a clever way of using Schnorr in combination with MAST.

Basically, because you can add anything to a public key, you can also add a script to a public key, because a script is essentially just a number and a private key is essentially just a number, and numbers can be added. Converting an elliptic curve private key to a public key also happens to be commutative. That let’s you do this:

```
public_key(private key + hash) ==
public_key(private_key) + public_key(hash)
```

Let’s now use an example of a backup system that you’d need in case you forget your private key. You start out with two keys: a primary key and a backup key. You keep your primary key on a very secure and tamperproof hardware wallet at home. Your backup key could be a note in a remote safe. If your house burns down, or if the hardware bricked itself after three incorrect PIN attempts, or if it’s stolen, you effectively lose the primary private key. So you’d fetch your backup key.

The backup key is what goes _in_ the MAST. If you never use it, nothing on the blockchain will indicate that you even have this backup. Under normal circumstances, you’d only use the primary key without revealing the MAST or the backup key in it.

But how does this hiding of the MAST work? Well that’s where Schnorr comes in. Schnorr lets you take this MAST and hide it inside your public key. Your wallet adds the root hash of the MAST to your private key, and then it calculates the corresponding, tweaked, public key. That tweaked key is what you put on the blockchain. To the outside world, it looks like any old public key.

And then, when you sign an actual transaction, you sign for this tweaked public key. Anyone else doesn’t see any difference between a tweaked public key and one that isn’t tweaked; they’re both perfectly valid public keys. Again, whether or not you tweaked your public key with this MAST structure, it looks the same to the rest of the world.

Only when you need to use your backup key is it time to reveal the MAST structure. Instead of using the tweaked key in your transaction, you reveal the original untweaked key, and you reveal the script (or one of your scripts, if you have a more complicated setup with more scripts in the tree).

Then, any person verifying, i.e. everyone who runs a full node, will take that script, calculate the hash, and add it up to the public key. They’ll see that this matches the tweaked key that was already on the blockchain, which proves you didn’t just make up a new script. The new script reveals to the world what your backup public key is, and they’ll check if your signature was indeed made using the private key for that public key.

Using this approach of a public key tweaked with a MAST is very space efficient. It improves privacy overall, because there’s no difference between transactions that pay to an individual with a simple single-key wallet and those that pay to an exchange with a super fancy multi-signature setup. It all looks the same, unless any of the backup conditions are used.

In the earlier example of you and your mom, if you accept Bitcoin with your mom this way, the first step is for the two of you to combine your public keys.^[The art of combining public keys and making joint signatures deserves a chapter of its own. It’s an important feature that Schnorr enables. But Taproot doesn’t do this for you. That’s up to wallet software and this is still a work in progress. The MuSig2 protocol is the latest proposal for how future wallet software can do this in a provably secure manner: <https://eprint.iacr.org/2020/1261>] Next, you generate a MAST with at least one leaf: the script specifying that after two years, you can spend the coins alone.^[It might also contain a second leaf that allows you and your mom to bypass the MuSig2 protocol and instead provide two individual signatures. This isn’t as good in terms of privacy, and it incurs higher fees, but it’s easier in some circumstances.]

Under normal circumstances, when you want to spend some coins, you call your mom and produce a joint signature. The coin you spend from specifies the public key, which has been tweaked with the MAST Merkle root. So you tweak your private key before producing a signature with it. What you publish will look like a regular signature for everyone else (because it is).

However, if there’s a scenario where one of you can’t sign and those two years go by, at the end you can reveal that it was actually a tweaked public key.^[Under the hood, every Taproot spend involves a tweaked key, using an empty MAST if there are no script leafs.] The rest of the world can look at that and say, “Yep. That adds up. The math makes total sense. That was what you were always doing; we just never were able to see it. Yep, two years have passed, so you’re allowed to spend this money now on your own.”

As a result, the condition is only revealed if you actually use it. Otherwise, it’ll be a secret forever, unless somebody hacks your wallet.

The ability to have multiple conditions and only reveal one of those conditions is what MAST enables. The ability to combine public keys with other keys and hashes is what Schnorr enables.^[<https://bitcoinmagazine.com/articles/the-power-of-schnorr-the-signature-algorithm-to-increase-bitcoin-s-scale-and-privacy-1460642496>]. But, this magic is combined, like Captain Planet, and now you can hide the MAST.

### Earlier MASTs

To appreciate Taproot even more, let’s take a brief excursion back in time.

The first MAST proposal, BIP 114,^[MAST: <https://en.bitcoin.it/wiki/BIP_0114>] introduced a new SegWit version. It offered privacy benefits similar to the Taproot Merkle tree proposal, and it only revealed the spending condition or script that was used.

Instead of introducing a new SegWit version, the second MAST proposal, BIP 116,^[`MERKLEBRANCHVERIFY`: <https://en.bitcoin.it/wiki/BIP_0116>] added a new opcode, `MERKLEBRANCHVERIFY`, to the existing script system. While the privacy was the same, the implementation varied.

However, there are downsides to both of these earlier MAST proposals:

1. As soon as you spend it, everyone can see that a MAST tree existed, even if they can’t see the full contents of the tree.
2. In the case where everyone agrees, you can’t just ignore the script and put signatures on the chain: You still have to pick a “we all agree” script from the MAST tree and satisfy it, which uses precious blockchain bytes.

By tapping the MAST root onto a Schnorr public key, so to speak, you fix these issues, as explained above.

### But Wait, There’s More…

While it’s true that some of the things you can do with Taproot were already technically possible (but more complicated), there are also some things Taproot unlocks.

For example, M-of-N signatures, or multi signatures, can now be done without a script, because Taproot enables protocols for combining them. This was possible before with threshold signatures in ECDSA,^[<https://eprint.iacr.org/2020/1390.pdf>], but like everything before Schnorr, it was complicated, and now it’s slightly easier.

To the outside world, a threshold signature looks like a single public key and a single signature. For M-of-M signatures, e.g. 2-of-2, the MuSig2 algorithm can be used. For the more general M-of-N, there’s no recommended algorithm yet. This isn’t a problem, because the algorithm for combining keys and signatures doesn’t need to be baked into the protocol; the Bitcoin protocol just needs to support Schnorr. When someone comes up with a new way to combine signatures, the result will look like a single signature — not just to humans, but also to nodes. And single signatures can be verified.

Another cool feature is how Taproot can make the Lightning network,^[<https://lightning.network/>] which is a layer 2 payment protocol, more private. Payments on Lightning involve passing a hash around, which is the same for all intermediate hops. This is a potential privacy concern, because someone with access to many nodes on the network could reconstruct the route a given payment took. With Schnorr, these hashes can be replaced with elliptic curve points that are different for each hop.^[<https://bitcoinops.org/en/topics/ptlc/>]

Additionally, Lightning uses channels, which are coins protected by two signatures, and with Taproot:

1. Those two signatures can be combined into one signature (e.g. using MuSig2).
2. The scripts Lightning uses to enforce good behavior can be hidden in the MAST, only to be revealed in the case of misbehavior.

In other words, if both sides of the channel agree on an operation, it looks like a normal transaction to outsiders. But if they disagree, there are a lot of additional timeout conditions, which can be nicely hidden inside the MAST.

However, most people won’t necessarily notice much of a difference, except that privacy will be slightly better. As these advanced options come along, they’ll use them but not notice them. That said, taking advantage of this functionality makes things cheaper, easier, and more private.
