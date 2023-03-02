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

В главе @sec:libsecp рассказывается о библиотеке libsecp256k1, а в мае 2021 года в нее добавилась поддержка BIP 340^[Шнорр: <https://en.bitcoin.it/wiki/BIP_0340>]. Это добавило в Bitcoin Core подписи Шнорра.

Цифровые подписи Шнорра были впервые созданы немецким математиком Клаусом-Петером Шнорром. Он создал алгоритм подписи Шнорра, который затем запатентовал. Этот алгоритм мог бы очень пригодиться Биткоину, равно как и другим проектам с открытым исходным кодом, которые появились еще раньше, но из-за патента людям пришлось найти другой способ, чтобы воспользоваться преимуществами этих подписей.

Поэтому множество юристов, инженеров и криптографов объединили усилия и попытались выяснить, есть ли способ покалечить алгоритм Шнорра настолько, чтобы он юридически не подпадал под действие патента, но все же работал. Результатом стал алгоритм подписи под названием Алгоритм цифровой подписи на эллиптических кривых (Elliptic Curve Digital Signature Algorithm или ECDSA), который представляет собой алгоритм для той эллиптической кривой, которую сейчас использует Биткоин и который реализуется в библиотеке libsecp. Хотя оба алгоритма - и Шнорра, и ECSDA - используют открытые и закрытые ключи для создания цифровых подписей, последний подразумевает несколько более сложный процесс.

Хотя ECDSA представляет собой изрядно запутанную версию подписей Шнорра, он был стандартизирован в 2005 году, и внедрен не менее чем в дюжину криптографических библиотек, включая OpenSSL. Итак, когда Сатоши нужно было выбрать криптографическую кривую для Биткоина, он выбрал ECDSA именно потому, что она не была запатентована и уже присутствовала в OpenSSL.

В целом, алгоритм Шнорра проще, чем ECDSA. Они используют одну и ту же эллиптическую кривую, но для создания подписи с ней приходится производить несколько иные вычисления. Так что это также означает, что правки для алгоритма Шнорра не так сложны, как, скажем, для первоначальной версии libsecp.

Первоначальная версия libsecp должна была представлять эллиптическую кривую, включая все операции, которые вы можете выполнять с кривой, такие как сложение и умножение, а затем реализовать алгоритм подписи ECDSA. Но для новой библиотеки вам просто нужно выполнить алгоритм подписи Шнорра, после чего можно пропустить или удалить всю ненужную математику.

Переход от ECDSA к алгоритму Шнорра это не очень большое изменение; при этом не меняется и не вводится совершенно новая эллиптическая кривая. Скорее, это переход к другому — и более простому — способу подписи.

Чем меньше изменений разработчикам придется внести в криптографическую библиотеку, тем лучше. Это означает меньшее количество мест, где могут быть допущены критические ошибки. Это также означает, что сообществу нужно проводить меньше проверок кода. На кону стоит почти триллион долларов, и любая ошибка, связанная с цифровыми подписями в Биткоине, может иметь катастрофические последствия, поэтому важность того, что требуется лишь простенькое изменение, трудно переоценить.

Вдобавок ко всему, поскольку алгоритм Шнорра добавляется как софтфорк, его использование полностью добровольно. ECDSA никуда не денется.

### Но почему Шнорр?

Простота — это здорово, но вся тяжелая работа по более сложной ECDSA уже сделана. Зачем утруждаться, что-то меняя? Еще до Taproot люди хотели добавить алгоритм Шнорра ради всех его возможностей. Но однажды участник Bitcoin Core и бывший технический директор Blockstream Грегори Максвелл придумал умный способ использования подписей Шнорра в сочетании с MAST.

По сути, поскольку вы можете добавить к открытому ключу что угодно, вы также можете добавить к нему скрипт, потому что скрипт — это, по сути, просто число, и закрытый ключ — это тоже просто число, а числа можно добавлять. Преобразование закрытого ключа в открытый ключ при помощи эллиптической кривой также оказывается коммутативным. Это означает вот что:

```
public_key(private key + hash) ==
public_key(private_key) + public_key(hash)
```

Давайте теперь рассмотрим в качестве примера систему резервного копирования, которая вам понадобится, если вы забудете свой закрытый ключ. На старте у вас два ключа: первичный ключ и резервный ключ. Вы храните свой первичный ключ в очень безопасном и защищенном от несанкционированного доступа аппаратном кошельке дома. Резервный ключ может быть, скажем, записан на бумажке в удаленном сейфе. Если ваш дом сгорит, или если аппаратный кошелек заблокируется после трех неправильных попыток ввода PIN-кода, или если он будет украден, вы фактически потеряете первичный закрытый ключ. И тогда вы просто используете свой резервный ключ.

Резервный ключ — это то, что _входит в MAST_. Если вы никогда его не используете, ничто в блокчейне не выдаст, что у вас есть резервный ключ. В обычных обстоятельствах вы бы использовали только первичный ключ, не раскрывая MAST или резервный ключ в нем.

Но как работает это сокрытие MAST? А вот тут-то и появляется алгоритм Шнорра. Шнорр позволяет вам взять этот MAST и спрятать его внутри вашего открытого ключа. Ваш кошелек добавляет корневой хэш MAST к вашему закрытому ключу, а затем вычисляет соответствующий измененный открытый ключ. Этот измененный ключ — вы как раз и помещаете в блокчейн. Для внешнего мира это выглядит как самый обычный открытый ключ.

И затем, когда вы подписываете фактическую транзакцию, вы ставите подпись для на этого измененного открытого ключа. Никто другой не видит никакой разницы между измененным открытым ключом и исходным; они оба - полностью валидные открытые ключи. Опять же, вне зависимости от того, изменили ли вы свой открытый ключ с помощью структуры MAST, он выглядит для остального мира совершенно однотипно.

Лишь когда вам нужно использовать свой резервный ключ, приходит время раскрыть структуру MAST. Вместо использования измененного ключа в вашей транзакции вы раскрываете исходный ключ и раскрываете скрипт (или один из скриптов, если у вас более сложная настройка с большим количеством скриптов в дереве).

Затем любой проверяющий, то есть каждый, кто запускает полный узел, возьмет этот скрипт, рассчитает хэш и добавит его к открытому ключу. Увидев, что результат соответствует измененному ключу, который уже был в блокчейне, проверяющий убедится, что вы не просто создали новый скрипт. Новый скрипт открывает миру ваш резервный открытый ключ, и владелец полной ноды проверяет, действительно ли ваша подпись была сделана с использованием закрытого ключа именно для этого открытого ключа.

Использование подхода с открытым ключом, настроенным с помощью MAST, очень эффективно использует пространство. Также это улучшает конфиденциальность в целом, потому что нет никакой разницы между персональными переводами с помощью простого кошелька с одним ключом, и транзакциями, которые отправляют монеты на биржу с супер-причудливой настройкой с несколькими подписями. Все выглядит одинаково, пока не используются какие-либо резервные условия.

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
