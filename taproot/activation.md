\newpage
## Варианты активации {#sec:taproot_activation}

\EpisodeQR{3}

Софтфорк Taproot был активирован 13 ноября 2021 года, примерно через год после появления окончательной редакции кода.^[<https://github.com/bitcoin/bitcoin/pull/19953>] Это произошло куда быстрее, чем активация SegWit, и куда менее драматично, но этот год не был совсем уж лишен событий. В этой главе рассказывается, как софтфорки активировались в прошлом, какие варианты активации рассматривались для Taproot и как он все-таки был активирован.

Мы посвятили этой теме пять эпизодов, и соответствующие QR-коды размещены в разных частях этой главы. Однако это далеко не однозначное сопоставление; ссылки приведены даже не в хронологическом порядке.

### Софтфорки: введение

По мере приближения развертывания Taproot вопрос о том, как активировать софт-форки, снова стал предметом дискуссий в биткоин-сообществе.

Софтфорки, если вы помните — это обратно совместимые изменения протокола. Другими словами, любой, кто обновился, будет пользоваться преимуществами, которые дает обновление, но те, кто не обновится, все равно останутся с корректно работающим софтом.

Помимо внедрения новых функций, софтфорк можно использовать для избавления от багов и потенциальных уязвимостей — по крайней мере, от некоторых из них. Это делается путем ужесточения правил, но без внезапной заморозки чьих-либо монет.

Простым примером таких более строгих правил является BIP 66,^[<https://en.bitcoin.it/wiki/BIP_0066>], который требует, чтобы любые новые подписи соответствовали строгому стандарту, в то время как ранее существовала некоторая (непреднамеренная) гибкость в способах кодирования подписей.^[<https://technicaldifficulties.io/2020/07/22/bip-66-unpacking-der-signatures/>]

Может показаться парадоксальным, что строгие правила допускают _больше_ возможностей, но в главе @sec:segwit, в разделе про будущие версии SegWit, мы объяснили, почему это работает. В текущей главе нас меньше интересует, как работают софтфорки, и вместо этого мы сосредоточимся на том, как их активируют.

Есть несколько разных способов внедрить софтфорк. Можно сделать это случайным образом, а можно внедрить его в код намеренно. Также можно объявить дату (день Д) или высоту блока, начиная с которой применяются новые правила. Наконец — и именно так это обычно сейчас работает — можно получать сигналы от майнеров и активировать софтфорк, как только будет достигнут определенный порог.

Но то, как принимается решение о развертывании софтфорка, возможно, даже важнее, чем механика активации. А кто вообще решает?

### Самые ранние софтфорки

Хотя на заре Биткоина этого термина еще не существовало, у него было много софтфорков, в основном связанных с закрытием дыр в безопасности у раннего прототипа.^[<https://blog.bitmex.com/bitcoins-consensus-forks/>] В 2013 году произошел даже случайный софтфорк, а в 2015-м случился случайный софтфорк, который едва не пропал из-за изменений OpenSSL (см. главу @sec:libsecp).

Самые ранние софтфорки в основном использовали в качестве метода активации высоту блока  — другими словами, «начиная с вот этого будущего блока должно применяться новое правило». В идеале об этом объявляют заблаговременно, чтобы у людей было достаточно времени для обновления. В случае «секретного» софтфорка разработчики могли просто настаивать на том, чтобы люди обновлялись, а уже затем объяснять причину.

![Неофициальная диаграмма софтфорка, активируемого по флагу на определенной высоте блока](taproot/flag.svg){ width=50% }

<!-- Отрегулируйте ширину вручную, чтобы примерно совпадал размер шрифта -->

Вероятно, самым печально известным софтфорком всех времен остается ограничение размера блока в один мегабайт, введенное Сатоши в 2010 году.^[Начиная с самого первого реелиза версии 0.1.0 от 9 января 2009 г. существовало применяемое к самым разным вещам ограничение в 32 МБ (`MAX_SIZE`). В их число входил размер блока, который проверялся в функции `CheckBlock()`. См. <https://satoshi.nakamotoinstitute.org/code/>. Затем, 15 июля 2010 года, Сатоши ввел параметр MAX_BLOCK_SIZE=1000000 и изменил программное обеспечение для майнеров, чтобы оно не создавало блоки большего размера, чем в <https://github.com/bitcoin/bitcoin/commit/a30b56ebe76ffff9f9cc8a6667186179413c6349>. Это еще не было софтфорком. Это было просто изменение программного обеспечения, используемого майнерами, которое они могли бы отменить, что не привелоо бы к созданию недействительных блоков. Спустя несколько месяцев, 7 сентября, Сатоши модифицировал связанную функцию AcceptBlock(), чтобы принудительно внедрить MAX_BLOCK_SIZE <https://github.com/bitcoin/bitcoin/commit/f1e1fb4bdef878c8fc1564fa418d44e7541a7e83>. Это уже был настоящий софтфорк, и он был выпущен в тот же день в версии 0.3.12. Оба коммита делали вид, что отвечают за совершенно не связанные между собой вещи. В настоящее время рецензенты кода осуждают коммиты, которые касаются чего-либо за пределами области, на изменение которой они претендуют, даже если это просто удаление пустой строки.] Софтфорк был выпущен 7 сентября 2010 года, а параметр `activation_height` для него был установлен на 79 400, и этот блок был добыт всего лишь неделю спустя.^[<https://bitcointalk.org/index.php?topic=999.msg12181>].

Решение Сатоши ввести это ограничение было не только односторонним, но и тайным. Вероятно, он счел более безопасным держать это изменение в секрете, потому что не хотел предупреждать потенциальных злоумышленников о зияющей дыре в безопасности, из-за которой массивные блоки могли остановить работу сети.^[Еще в 2017 году я провел эксперимент, в котором я взял более старые версии Bitcoin Core и измерил, сколько времени им потребовалось, чтобы синхронизировать состояние блокчейна. Современные узлы делали это во много раз быстрее благодаря различным усовершенствованиям, сделанным за прошедшие годы (см., например, главу @sec:assume). Но что еще более важно, Bitcoin Core v0.5, выпущенный в 2012 году, вообще не успевал за блокчейном. При подобной попытке он бы просто упал или ушел в ступор на несколько недель. Без ограничения размера блока, введенного Сатоши, любой в 2012 году мог создать огромные блоки, которые затем перегрузили бы доступное программное обеспечение узла (см. Приложение @sec:more_eps).]

Не то чтобы никто не отслеживал изменения в исходном коде, просто на форуме, где Сатоши анонсировал этот новый релиз, в тот момент обсуждался _еще один_ софтфорк, предлагаемый для активации с блока на той же высоте. В любом случае, никто не обращал на это особого внимания до тех пор, пока много лет спустя этот уменьшенный лимит не приобрел практическую значимость в виде увеличения платы за дефицитное место в блоке.

Но если не считать каких-то экзистенциальных чрезвычайных ситуаций, существует общее мнение, что сейчас это неприемлемый способ введения софтфорка. Если бы уже тогда случились дебаты об ограничении размера блока, возможно, драму, которая разразилась позже, можно было бы предотвратить.

### Как обеспечиваются софтфорки?

Выпуск новой версии программного обеспечения, которая активирует софтфорк на заданной высоте — это одно. Но если его не запустят правильные люди и не сделают это вовремя, на самом деле он не возымеет эффекта. Если софтфорк выпущен в лесу…

Что сделал Сатоши, так это объявил о новой версии на форуме, предполагая, что сообщество достаточно маленькое, и все успеют обновиться до момента активации, даже если до него осталась всего неделя. Как правило, факт обновления не подвергался проверке, потому что многие из этих первоначальных софтфорков были сделаны таким образом, намеренно или случайно, что только злоумышленник мог создавать блоки, нарушающие новое правило, а таких вокруг было немного.

Несмотря на это, даже в 2010 году уже росло понимание того, что самый безопасный способ реализовать софтфорк — это добиться, чтобы подавляющее большинство майнеров запустило последнюю версию. Это связано с тем, что узлы будут наращивать самую длинную цепочку, которую они считают действительной. Таким образом, даже если пользователь не обновил свой узел, пока большинство майнеров соблюдают правила, они будут создавать самую длинную цепочку, и узел пользователя будет следовать за ней. Таким образом, даже если майнер злонамеренно или случайно создаст недопустимый блок, большинство майнеров не будет его надстраивать, и недействительный блок вскоре станет устаревшим. ^ [См. главу @sec:eclipse для объяснения концепции устаревших блоков.]

Мы не знаем, сколько отдельных майнеров было в 2010 году, но вполне возможно, что они были в курсе этих обновлений. ставка до февраля 2010 года:^[Некоторые предполагают, что один майнер под ником Патоши, не обязательно Сатоши, контролировал более 50 процентов хэшрейта до февраля 2010 года: <https://whale-alert.medium.com/the-satoshi-fortune-e49cf73f9a9b>] При этом на майнинге нельзя было сделать деньги: первая продажа пиццы состоялась только в мае того же года.^[<https://bitcoinmagazine.com/culture/the-man-behind-bitcoin-pizza-day-is-more-than-a-meme-hes-a-mining-pioneer>] Даже в 2013г, когда произошел случайный софт-форк, достаточное количество майнеров запустило исправление всего за несколько часов.^[<https://bitcoinmagazine.com/technical/bitcoin-network-shaken-by-blockchain-fork-1363144448>] Но чрезвычайные ситуации — это не то же самое, что обычные софтфорки.

### Кто решает?

Первоначально Сатоши в одностороннем порядке решал, что нужно изменить, хотя в конечном итоге он не мог заставить кого-либо запускать новые версии, которые он выпустил, поэтому он не мог вносить совершенно произвольные спорные изменения.^[Лучше покинуть вечеринку, пока еще получаешь от нее удовольствие, и, возможно, именно так он и поступил: «Но по мере того, как росло недовольство его властью и возможностями, пользователи стали слишком часто осуждать Сатоши-администратора, Сатоши-бутылочное-горлышко, Сатоши-диктатора». <https://bitcoinmagazine.com/technical/what-happened-when-bitcoin-creator-satoshi-nakamoto-disappeared>] В данный момент разработка Биткойна ведется, если не залезать в дебри, в рамках процесса «приблизительного консенсуса», как описано в RFC 7282.^[<https://datatracker.ietf.org/doc/html/rfc7282>]

Любой может предложить изменение правил Биткоина. Но нужно не только убедить других в полезности изменения; также необходимо рассмотреть все технические возражения, которые выдвигаются против него. Например, если кто-то жалуется, что предложение нарушит работу его кошелька, автор предложения не может просто сказать «не повезло». Вместо этого он должен решить проблему. Может быть, он может предложить простое исправление для рассматриваемого кошелька, или изменить свое собственное предложение, чтобы оно ничего не ломало.

Это — наряду с несколькими другими требованиями — обычно означает множество переписок по техническим спискам рассылки, а также множество итераций по улучшению предложения. Многие предложения вообще не выдерживают этот процесс, потому что оказывается, что они вызывают слишком много проблем.

Предложения о хардфорке часто отклоняются только потому, что они ломают существующее программное обеспечение и требуют одновременного обновления от всех участников. То же самое предложение, вводимое как софтфорк, решает обе проблемы, поэтому его обычно предпочитают хард-форку. Вот почему небрежное предложение разработчика Bitcoin Core Люка Даша-младшего о том, что SegWit можно реализовать как софтфорк, изменило правила игры.^[<https://news.ycombinator.com/item?id=11230394>]

Помимо отсутствия неотвеченных возражений, также необходимо привлечь достаточно опытных разработчиков для рассмотрения предложения. Таких разработчиков очень немного, и отсутствие энтузиазма у них может привести к тому, что прекрасный софтфорк никогда не увидит свет. Или, что чаще, отсутствие энтузиазма у рецензента в сочетании с трудноразрешимыми техническими проблемами будет держать предложение в подвешенном состоянии.

Но если все пойдет хорошо и код предложения в конечном итоге будет объединен с кодом Bitcoin Core, остается вопрос, какую применить процедуру активации. Это происходит в рамках такого же процесса приблизительного консенсуса, как и обсуждение самого предложения; людям может понравиться предлагаемый софтфорк, но они, по всем вышеописанным причинам, будут возражать против активации в конкретную дату. Чтобы избежать ситуаций, когда предложения застревают в ходе обсуждения их активации, в идеале сообщество соглашается на единый механизм активации, который применяется для всех софтфорков. Но, конечно, это тоже серьезная проблема для сообщества.

### Signaling (BIP 9)

So if a flag day or block height isn’t the best way to activate soft forks, how can we do better? One idea was to have miners signal readiness in the blocks they create. Initially, this was done by increasing the block version number.^[BIP 34 <https://en.bitcoin.it/wiki/BIP_0034> in 2012 used version 2, and BIP 66 in 2015 used version 3. Once 95 percent of recent blocks contained this new version, the new rules would apply.] Signaling works as a coordination mechanism for the network to figure out that enough miners have upgraded.

The mechanism was improved and further formalized in BIP 9.^[<https://en.bitcoin.it/wiki/BIP_0009>] As part of the signaling mechanism embedded in the code, there’s a starting date (`starttime`) when miners begin signaling, and a deadline (`timeout`) at which point they give up if the `threshold` wasn’t reached. Tallying happens in rounds of 2,016 blocks, or about two weeks. If the threshold is reached in any round that ends before `timeout`, the new rules are active. This is easy for every node in the network to verify.

 ![BIP 9 flow](taproot/bip9.svg){ width=80% }

The significance of 2,016 is that it’s the number of blocks in a single difficulty adjustment period, or retarget period.^[<https://en.bitcoin.it/wiki/Difficulty>] In the diagram above, each arrow represents one signaling period. The looping arrows indicate when the state stays the same — for example, when a soft fork is `DEFINED` (meaning the node knows about it, but there’s no signaling yet), it’ll stay that way if the MTP^[MTP stands for Median Time Past, and it refers to the middle block of the last 11 blocks. This is a mechanism used to discourage miners from gaming the timestamp in each block.] is still below `starttime`. When it’s at or after `starttime`, the state jumps to `STARTED`. It stays there pending signaling. For each period — say every two weeks — we check if enough blocks are signaling. If so, we move to the next phase, which is `LOCKED_IN`. If not, and if `timeout` is reached, we move to `FAILED`. `LOCKED_IN` is a grace period where the new rules don’t yet apply, but after two weeks, the soft fork is `ACTIVE` and the rules do apply.

Signaling hands control of upgrade activation to miners for a predefined period. It requires a signaling readiness of 95 percent for a soft fork activation to succeed.

This mechanism has the additional benefit of allowing the deployment of multiple soft forks in parallel. Each gets assigned a specific signaling bit.

BIP 9 was used successfully to deploy the CSV and SegWit soft forks, and it could’ve been used for Taproot as well. The first deployment went smoothly, but as we’ll soon see, the second one involved much drama and took far longer than many people considered necessary. This led to worries that perhaps BIP 9 isn’t a future-proof deployment mechanism.^[Its use of timestamps rather than block heights also seemed to needlessly complicate things.] As a result, several other mechanisms were proposed, along with variations on those mechanisms.

### Always Verify Blocks!

Unfortunately, no amount of signaling is useful when miners don’t actually enforce the new rules. Remember that we need the majority of miners, i.e. the longest chain, to enforce the new rules to protect non-upgraded nodes (which in turn allows these upgrades to remain non-mandatory).

During the BIP 66 soft fork in 2015, it turned out that a large portion of miners, despite signaling readiness, weren’t verifying the new rules.^[<https://www.reddit.com/r/Buttcoin/comments/6dvkr6/short_writeup_of_the_bip66_disaster_is_this/>] As soon a transaction that violated the new rules appeared in a block, those miners failed to reject it and instead kept building on top of it.^[<https://bitcoin.org/en/alert/2015-07-04-spv-mining#cause>] At the same time, the miners that upgraded and checked the new rules did reject the block. They produced an alternative, valid block at the same height and kept building on top of that new branch. Eventually, their new branch overtook the invalid branch, causing the non-verifying miners to switch over to the valid branch. The first time, the invalid chain branched off for six blocks. The next day, it happened again, but only for three blocks, perhaps because more miners upgraded their software as they became aware of the problem.

Skipping block verification is risky for miners even without a soft fork,^[It’s even more reckless for miners to not even verify the _transactions_ they selected to put in their own blocks. But back in 2015, there was at least one small miner who did this: <https://www.reddit.com/r/Bitcoin/comments/3c305f/comment/csrt3dg/>] but under normal and benevolent circumstances, they may get away with it. However, in a soft fork where a majority of miners enforces the new rules, but a minority doesn’t, when one of these minority miners accidentally mines an invalid block, other such non-verifying miners will build on top of it. The upgraded majority will ignore all these blocks, and once their chain is longer, all these minority miners find their blocks stale. That’s what happened with BIP 66.

### The First Drama, and BIP 148/91

When it was time to deploy SegWit, after the code was ready, BIP 9 was used again because it worked fine before. But as months went by, in none of the biweekly retargeting periods was the required 95 percent block signaling threshold reached. Although one can’t tell by looking at the blockchain, it appeared that several large miners were using the BIP 9 readiness signal, or lack thereof, as a vote, in turn blocking the upgrade.

They weren’t raising any technical objections, so the previously established RFC 7282-style rough consensus was still intact. Was it just inertia and apathy? Was there a secret technical objection they had to the proposal?^[It was speculated that a technique called covert AsicBoost was giving certain miners an advantage over their competitors that they preferred not to disclose: <https://blog.bitmex.com/the-blocksize-war-chapter-14-asicboost/>] Or were miners frustrated with and lacking confidence in the developers, perhaps in part due to language and cultural barriers?^[Many miners were Chinese, and many developers were from Western countries.]

In any case, BIP 9 wasn’t supposed to be a referendum, and given how quickly miners were able to upgrade with previous soft forks, this stalemate made little sense and frustrated those who wanted to take advantage of the SegWit features we discussed in chapter @sec:segwit. One particularly frustrating aspect of this situation is that the lack of signaling was holding back the much demanded block size increase that SegWit offered.

At this point, most of the Bitcoin Core developers were perhaps just as frustrated, but they preferred to stay on the conservative side and simply wait for the `timeout` and some earlier success. When there’s no consensus on a new course of action, the default is to just do nothing. But that doesn’t have to stop anyone else.

Around early March of 2017, there was a group of people that said, “Hey, you know what? It’s time for a flag day.” They picked August 1, 2017, as a date and said that on that day, their nodes would enforce the new SegWit rules.

To be more precise, on that date, they would require all blocks to signal for SegWit. That signaling would in turn cause SegWit to activate. BIP 148 was the mechanism, and it was commonly referred to as a User-Activated Soft Fork (UASF).^[<https://en.bitcoin.it/wiki/BIP_0148>] The similarity of this acronym to a certain branch of the US military was helpful in its marketing.

The question is: Did it work? One can’t tell by just looking at the blockchain. When you look at historical blocks, you just see 95 percent signaled, and the soft fork was activated. Now, it’s of course very remarkable that this activation occurred right before August 1 and not some random other date that it could have happened. But there are no stale branches of non-signaling blocks that we could point to as evidence for a fight between miners and UASF nodes.

As we’ll explain in the `LOT=true` discussion below, it’s probably a good thing that no confrontation in the form of competing blockchain branches happened.

A few days after BIP 148 was published, an alternative to it was posted on the developer mailing list. It proposed activating SegWit, along with an additional doubling of the block size using a hard fork. We already pointed out above that, thus far, hard fork proposals haven’t survived the technical objections raised against them, e.g. the need for all node users and wallet software to upgrade at the same time, and the mandatory nature of such an upgrade.

But despite not being well received in a technical forum, a group of major companies in the space met in a New York hotel and announced the so-called New York Agreement.^[<https://dcgco.medium.com/bitcoin-scaling-agreement-at-consensus-2017-133521fe9a77>] They also added a lower activation signal threshold.

_The Blocksize War_ (the book mentioned at the end of chapter @sec:segwit) goes into more detail about where all that led (spoiler: They called off the hard fork at the last minute). For the purpose of this chapter, it’s interesting to briefly explain the idea of lowering the activation threshold, which enjoyed a more positive reception.

The next day, BIP 91 was proposed on the mailing list.^[<https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2017-May/014380.html>]. It worked as follows: If 80 percent of miners signaled in favor, then (two weeks later) all miners would need to signal. And once all miners signaled, the 95 percent threshold would be reached. This would cause SegWit to activate from the perspective of everyone involved.

Users that run the more conservative Bitcoin Core node would see the 95 percent signal and consider SegWit active. Users that run the more aggressive UASF node would see the same, provided it happened before August 1.

Similar to the situation with UASF, we can’t really tell what happened by just looking at blocks.^[SegWit signaled on bit 1, and the New York Agreement folks signaled on bit 4 (as did BIP 91). The bit 4 signal did cross the required threshold: <https://bitcoinmagazine.com/technical/bip-91-has-activated-heres-what-means-and-what-it-does-not>. However, that doesn’t prove that BIP 91 _caused_ SegWit to activate. That’s because things happened too quickly. During the retarget period, the BIP 91 status was `LOCKED_IN`, meaning it wasn’t yet enforcing its 100 percent signaling requirement. But right in that period, SegWit reached its 95 percent threshold on bit 1. So in the next period, BIP 91 was `ACTIVE` and SegWit was `LOCKED_IN`. Both required 100 percent signaling at that point. In reality, it’s likely that miners were simply setting bit 4, but not actually running any of the software that used it.]

So in the end, all we can say is it didn’t go wrong. Nobody called each other’s bluff. But what did come out of it is that everyone realized it was important to rethink how to actually activate soft forks. Having a situation where miners block activation without (publicly sharing) a good technical reason isn’t ideal. A bunch of contentious chain splits as different camps battle it out isn’t great either — it defeats the purpose of carefully engineering soft forks that maintain a well-functioning blockchain.

### Rethinking Activation, and the Introduction of BIP 8

Inspired by UASF, as well seeing the need to clean up BIP 9, BIP 8^[<https://en.bitcoin.it/wiki/BIP_0008>] was proposed.^[In case you’re wondering why a new proposal gets a lower BIP number, my guess is as follows: BIP numbers tend to be thematically grouped in ranges of 10, e.g. BIP 340–343 all relate to Taproot. BIP 1 and 2 are meta issues, and they explain, among other things, how Bitcoin ought to be upgraded. So it makes sense to include soft fork activation logic in the single-digit range, but fill it out in descending order.] It uses a signaling mechanism similar to BIP 9, but based on block height rather than timestamps. This makes it easier to reason about reorganizations, e.g. when the last block of a signaling period arrives just _before_ the `timeout`, a soft fork would activate, but then if an alternate branch of blocks appears and takes over, and this alternate branch arrives (has timestamps of) directly _after_ `timeout`, then — on that branch, that rewriting of history — it wouldn’t activate.

The downside of using a block height is that you can’t predict on which date the `timeout` is going to happen, because it depends on how fast blocks are produced.^[This is mainly a problem for developers who use the testnet to test e.g. wallet software ahead of the real activation. On the testnet, blocks don’t arrive in semi-regular 10-minute intervals, but can instead arrive in huge numbers. This makes it impossible to pick reasonable activation heights. For a more thorough explanation and solution, see appendix @sec:more_eps for the episode about Signet.]

![BIP 8 flow](taproot/bip8.svg)

So far, BIP 8 would just be a nice drop-in replacement for BIP 9. But where it really differentiates itself is in the UASF-style flag date for forced signaling. This is indicated by the `MUST_SIGNAL` phase in the diagram, which otherwise doesn’t differ much from what you saw above with BIP 9.

What this means is that the flag date doesn’t activate the soft fork itself. Rather, if, closer to the date, there’s a block that isn’t signaling support for the soft fork, that block will be rejected: That’s how it forces signaling toward the end.

Like with the UASF, this forced signaling only makes sense if there are _other_ nodes out there that don’t have such a flag date.^[For nodes without that setting, the horizontal line from `STARTED` to `MUST_SIGNAL` is never used, and instead the vertical line from `STARTED` to `FAILED` would occur, and it’s essentially BIP 9.] This is why the flag day setting itself is optional. What exactly does optional mean here? It could mean that there are two different downloads: one with the flag day enabled and one without, presumably released by two separate teams with different priorities. It could also mean that there’s a single download that allows the user to choose.

This allows for an escalation ladder: Perhaps most of the community starts out not using a flag date, and then as time goes by, more do.

In this sense, it formalizes the process that happened informally in 2017, basically saying: If you want to do something like a UASF, please do it in this specific way.

Just like its predecessor, UASF, this proposal isn’t without its problems. The following paragraphs outline how forced signaling is supposed to work.

_If you run a node with this feature enabled_, then when, after the flag day, a block that doesn’t include a signal is produced, your node will consider this block invalid, no matter how many other blocks are built on top of it. If some miners produce an alternative branch of blocks, even if it’s shorter, that do include the signal, your node will follow that alternative branch. Eventually, if and only if enough blocks are produced on this alternative branch, the soft fork is guaranteed to activate.

On the other hand, _if you run a node without this feature_, or for that matter, if you run older node software that doesn’t know about the soft fork, then you’ll continue to follow the longest chain, regardless of what its block signal is. If the longest chain happens to comply with mandatory signaling, your node will follow it. If it doesn’t comply, your node will also follow it. The scenario to worry about here is when, initially, the longest chain doesn’t signal, but after some time it gets overtaken by a chain that does. Since your node doesn’t care if blocks signal or not, it’ll happily switch over to this new branch.

In any scenario where two alternative chains exist, it’s unsafe for users whose node follows one branch to transact with users whose node follows the other branch. In fact, it’s unsafe for _anyone_ to use the blockchain at that point. On the other hand, as long as the only chain in existence complies with mandatory signaling, there’s nothing to worry about. This might remind some readers of the game theory around mutual assured destruction (MAD).

### To Argue a `LOT`

\EpisodeQR{29}

This setting to require mandatory signaling became known as `LOT`. We dedicated several episodes to the debate around it, not all of which made into this book. The transcript for the accompanying episode can be found here.^[This transcript was written by Michael Folkson. The site contains many other transcripts from technical Bitcoin podcasts, conference talks, and even group conversations. Many of them are written by Bryan Bishop aka Kanzure, who is quite possibly one of the fastest typists on Earth. <https://diyhpl.us/wiki/transcripts/bitcoin-magazine/2021-02-26-taproot-activation-lockinontimeout/>]

When it came to the Taproot activation process, there was a debate surrounding this `LOT` parameter, which stands for Lock-in On Timeout. If you refer to the BIP 8 diagram above, `lockinontimeout` appears in the horizontal arrow. When set to `false` we transition to `FAILED` after the soft fork times out. But when it’s set to `true`, then in the very last 2,106-block retargeting period, we go to `MUST_SIGNAL`, and that’s always followed by `LOCKED_IN`. In other words, we lock in right before (“on”) the timeout, hence the name.

In other words, in the case of both `LOT=false` and `LOT=true`, miners can signal for an upgrade for one year. Then, if the specified threshold percentage — in the case of Taproot, 90 percent — is met, it’ll activate. However, if it isn’t met, two things could happen. With `LOT=false`, the Taproot upgrade will expire, but a new activation period could be implemented if the community decides to try again with a new software version. With `LOT=true`, nodes will begin only accepting blocks that signal for the upgrade, which forces the activation.

Initially, in early 2021, there wasn’t yet a Bitcoin Core release that would activate Taproot. Such a release was still pending community debate on what the appropriate activation mechanism should be. So this is a different context than during the UASF debate in 2017, where there _was_ an existing Bitcoin Core BIP 9 deployment in the `STARTED` stage. Meanwhile, in early 2021, the discussion revolved around whether Bitcoin Core should switch from its usual BIP 9 deployment system to BIP 8, and if so, if `LOT` should be set to `true` or `false`.

The switch from BIP 9 to BIP 8 wasn’t very controversial, as long as it stuck to the more conservative `LOT=false` incarnation. But it’s not a no-brainer, because there’s always a risk when making _any_ change, especially to something as critical as the code that decides which rules apply to blocks. So it still raised the question: Is a switch to BIP 8 with `LOT=false` worth it?

It might be worth it to remain compatible with software from an independent group that insists on `LOT=true` (compatibility shouldn’t be construed as endorsement). But this debate was never settled.^[There was pull request that implemented a transition from BIP 9 to BIP 8 in Bitcoin Core. This is a generic transition and not Taproot specific. However, it contained `LOT=true` code, which added complexity and triggered objections. A pure `LOT=false` version might have made it through review. <https://github.com/bitcoin/bitcoin/pull/19573>]

The real controversy revolved around `LOT=true`.

A lot of people supported `LOT=true` because it made it so miners couldn’t have a veto. The counterargument to that is that miners don’t have a veto anyway; they can merely delay an orderly activation. This is because node owners can always switch to a new version of the node software that has a flag day, bypassing signaling altogether. But they’ll have to wait.^[If the community doesn’t wait until the timeout and activates a soft fork with a flag day, then anyone who doesn’t upgrade to the new flag day software will think the soft fork failed to activate. Mandatory signaling avoids this need to wait, but at a cost.] If activation is delayed by a lack of signaling with `LOT=false`, the upgrade will expire after a year. After that, we could deploy any new upgrade mechanism, and potentially one without any signaling.

Another option could even be to start with `LOT=false`, wait half a year, and then say, “This is taking too long. Let’s take a little more risk and set `LOT=true`.”

The debate itself arguably ended up slowing down Taproot activation: There was no reason to believe miners would object to Taproot the way some did to SegWit, and there was no political drama around Taproot itself. It was very conceivable that a BIP 9 or BIP 8 with `LOT=false` activation would’ve gone smoothly, as it happened in the pre-SegWit era. But the debate created a stalemate, because Bitcoin Core generally doesn’t ship functionality that’s deemed controversial, and at that point, all activation options were controversial to at least some people.

It wasn’t a stalemate because some people _preferred_ `LOT=true`. Remember the RFC 7282 rough consensus process. As long as they didn’t _object_ to `LOT=false` (or BIP 9), then their preference for `LOT=true` could be dismissed. Because in that case, what would have remained were objections to `LOT=true` and no objections to `LOT=false`, and so you’d have moved forward with the thing nobody objected to.

But some people went beyond a mere preference for `LOT=true`. They claimed `LOT=false` was unsafe,^[<https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018498.html>] i.e. they objected to it. So we ended up with two proposals that both had objections. That these objections were advocated for by a very small number of developers was immaterial. They needed to be addressed, e.g. shown to be incorrect or solved somehow. And that process could and did drag on for a while.

### The Chain Split Scenario

The main objection to `LOT=true` was the same as the one raised against UASF: It could cause a chain split. Remember the reference above to MAD. An often-heard argument _for_ `LOT=true` is that a chain split is so terrible — especially for miners who wouldn’t be able to sell their new coins, it won’t happen. They believe this is sufficient deterrence for miners to simply signal. Such game theory is beyond the scope of this book, but we can clarify what such a chain split would be like and why it’s bad.

The following is one such example of how, if one part of the network runs `LOT=true` and another part runs `LOT=false` or an older version of the software, bad things could happen in the form of a chain split.

Let’s say you’re running the `LOT=true` version of Bitcoin Core and you want Taproot to activate. But the scenario here is that the rest of the world — including most miners — isn’t doing this. The day arrives and you see a block that isn’t signaling correctly but you want it to signal correctly, so you say “This block is now invalid, so I’m not going to accept this block. Instead, I’m just going to wait until another miner comes with a block that does meet my criteria.” Maybe that happens once in every 10 blocks, so you’re seeing new blocks, but they’re coming in very, very slowly.

So far, you might think this is merely annoying. But now let’s put some money on the line. What if you’re trying to receive a payment?

Let’s say somebody who runs a node with `LOT=false` sends you 1 BTC. So in this scenario, they’re on a branch that’s growing 10 times faster than the branch your node is seeing. Let’s also say their blocks aren’t completely full, so the sender uses a low fee rate. The transaction confirms quickly in their branch. But you’re on this shorter, slower-moving branch, and all those transactions have to be squeezed into fewer blocks. Those blocks are completely full. In your branch, the transaction doesn’t confirm. It’s just sitting there in the mempool.

![In this example, blocks on the fast moving side of the chain split require a fee of at least 1 satoshi per vbyte.^[<https://bitcoin.stackexchange.com/a/89418/4948>] Due to congestion on the slower moving side, its minimum fee rate is 50 sat/vbyte. A transaction paying 3 sat/vbyte only confirms on one side of the split.](taproot/split.svg)

That’s actually a relatively benevolent scenario. You’ve learned that you shouldn’t accept unconfirmed transactions. You’ll have a disagreement with your counterparty, you’ll say, “It hasn’t confirmed,” and they’ll say, “It has confirmed.” Assuming you’re aware of this chain split, you might realize what’s going on.

If you had anticipated this situation, you would’ve asked your counterparty to pay a much higher fee than their node suggests (and maybe deduct it from the amount). Otherwise, you could’ve used something called Child Pays For Parent (CPFP) by taking their unconfirmed low fee transaction and spending it back to yourself with a very high fee transaction. The combined fee is then enough for miners to include both.

A much worse scenario is when your counterparty is trying to send you coins that don’t exist on your branch. How can that be? One possibility is that they’re spending a coin that descends from a coin that was created by a miner on their branch. Every block contains a coinbase transaction, which sends the block subsidy (6.25 BTC at the time of writing), plus any transaction fees collected, to the miner. Each branch will have a different sequence of miners producing its blocks. This means each branch has unique coinbase transactions that don’t exist on the other branch. That, in turn, means any transaction spending from such a coinbase transaction can’t exist in the other branch.^[This can be used to generate something so called UTXO Fairy Dust. By deliberately spending a piece of that dust in a transaction, you can guarantee it won’t replay on the other branch.]

The general phenomenon described here is called transaction replay. In the case of a hard fork, it’s often desirable to _prevent_ transactions on one side of the fork from appearing (being replayed) on the other side. If this sounds fascinating, then you may like the author’s presentation on the topic.^[<https://sprovoost.nl/2017/11/10/a-short-history-of-replay-protection-2bd8b288cf94/>] Otherwise, just understand that whether you want to prevent replay or guarantee it, it’s a pain.

It’s possible that the coins on both branches will have a different market price. In that case, the above examples become even more complicated, because you and your counterparty can’t agree on what 1 BTC is worth.

In any case, translated to the RFC 7282 rough consensus process: If you propose something that creates the possibility of transaction replay, you should address how to deal with it.

But it gets worse.

Continuing with the above scenario of a short branch with `LOT=true` and a long branch with `LOT=false`, perhaps over time, the market price for coins on `LOT=true` increases. This price increase attracts miners, and this increase in mining activity increases the pace of block creation on this branch. In turn, it slows it down on the `LOT=false` branch.

This can lead to a tipping point if `LOT=true` overtakes the `LOT=false` branch. Your node chugs along just fine. It was following the `LOT=true` branch already when it was shorter, and it continues to do so now that it’s longer.

But your friend in the other branch is about to have a very traumatic experience. Their node detects the longer branch. From its point of view, that branch is perfectly valid; the mandatory signaling on it doesn’t violate any rules. So it’s going to switch over!

This is very bad. Any transactions your friend had sent or received on their branch, if they don’t also appear on your branch, will disappear. More accurately: Those transactions will either return to mempool, from which they could later end up getting confirmed again, or they could disappear entirely if they descend from a coinbase transaction.

Anything like a big reorganization^[<https://en.bitcoin.it/wiki/Chain_Reorganization>] will cause mayhem, even without any malicious actors. Let’s say a miner deposited coins on an exchange and your friend withdrew some coins from that same exchange. Exchanges generally don’t allocate specific coins to specific users, so there’s a chance the exchange used some of the miner coins to pay your friend. That means your friend never received that money and has to complain to the exchange. But if this happens on a large enough scale, the exchange is probably going to be insolvent.

Again translated to the RFC 7282 rough consensus process: Is it really enough to simply claim this scenario won’t happen because of incentives to prevent it? Is it so unlikely that not even a contingency plan is needed to handle it? Some cities have nuclear shelters despite the MAD game theory, though others have indeed repurposed them as shopping malls. It seems like more of a political debate than a technical discussion.

Finally, it’s worth pointing out that all the problems that `LOT=false` users are subjected to in world with `LOT=true` clients are also encountered by users who don’t upgrade at all. Avoiding mandatory upgrades is also something to consider.

### `LOT=true` Client, Rogue?

\EpisodeQR{36}

This time around, the first software download release with the Taproot activation code wasn’t Bitcoin Core. Instead, two developers decided to independently release a modified version of Bitcoin Core, which included BIP 8 and the `LOT=true` behavior.^[<https://www.reddit.com/r/Bitcoin/comments/mruopv/bitcoincorebased_bip8_lottrue_taproot_activation/>]

\noindent
With open source software, anyone is free to release any variation of the software they want. Similarly, everyone is free to download whichever variation they want. However, in addition to the general objections to `LOT=true` above, there are other practical matters to think about when downloading such an alternative implementation. We cover these in the episode above. In particular, it’s important to make sure you’re not accidentally downloading malware (see chapter @sec:guix).

### The Speedy Trial Proposal

\EpisodeQR{31}

To get out of this stalemate, Speedy Trial came to the rescue. It proposed^[<https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018583.html>] the following: “Rather than discussing whether or not there’s going to be signaling and having lots of arguments about it, let’s just try it quickly.” The proposed timeline^[<https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018594.html>] suggested the signaling would start in early May, last three months (until August), and then be activated three months later, in November.

Spoiler: As we mentioned at the beginning of the chapter, Taproot indeed activated on November 13, 2021.

![Speedy trial flow.](taproot/speedy_trial.svg)

The diagram above is quite simple to understand. Compared to BIP 9 above, it only introduces one new concept: a minimum activation height. This adds a delay in the transition from `LOCKED_IN` to `ACTIVE`.

Although it’s conceptually almost the same as BIP 9, the dates picked for Taproot are quite different than what would’ve been picked before. The waiting period before signaling (`DEFINED`), as well as the signaling period (`STARTED`), are much shorter than usual (months instead of a full year). This way, we could know the result faster.

Knowing the result quickly is great, but the rush runs the risk of miners using fake signals rather than actually upgrading. So to add a margin of safety, the transition from `LOCKED_IN` to `ACTIVE` was increased from the usual single period (two weeks) to a fixed block height, which was expected to be reached in November 2021. That was the only code change required (a much smaller change than BIP 8).

So the “speedy” part refers to figuring out miner readiness, or at least to figuring out if there was any previously unknown miner objection, or just apathy. The rest of the process was slower, and it behaved a bit more like a flag day. Once the signal threshold was reached, the soft fork was set in stone, meaning it would happen, at least if people ran the full nodes.

This process made it so Taproot would activate six months after the initial release of the software, assuming 90 percent of miners were signaling. If that threshold wasn’t met, the proposal would’ve expired, and activation options would’ve been discussed more, albeit with more data to back up decision making.

Speedy Trial seemed to sufficiently address the objections to BIP 9. From the objectors’ point of view, because it was so fast, their own plans for BIP 8 wouldn’t be delayed.

With the controversy (temporarily) out of the way, more developers came out of the woodwork and started writing code that could actually get Speedy Trial done.^[Mainly <https://github.com/bitcoin/bitcoin/pull/21377>, <https://github.com/bitcoin/bitcoin/pull/21686>, and a BIP 8-based alternative that was briefly considered: <https://github.com/bitcoin/bitcoin/pull/21392>] In turn, because there were more developers from different angles cooperating on it and getting things done a little bit more quickly, it demonstrated that Speedy Trial was a good idea. When you have some disagreement, then people start procrastinating, not reviewing things, or not writing things. But if people begin working on something quickly and it’s making progress, that’s a vague indicator that it was a good choice.

### We Have Taproot `LOCKED_IN`!

\EpisodeQR{40}

Bitcoin Core v0.21.1 with the Speedy Trial code was released on May 1, 2021.^[<https://bitcoincore.org/en/2021/05/01/release-0.21.1/>]

\noindent
The first retargeting period started a week before that release on April 24, 2021, and the threshold wasn’t reached. The second retargeting period also didn’t reach the threshold, but the third time was a charm. The 90 percent signaling threshold was reached on June 12, 2021, with `LOCKED_IN` happening a few days later.^[<https://sports.yahoo.com/locked-bitcoin-taproot-upgrade-gets-120837972.html>] It lasted until the November activation.

Remember that the signal for a soft fork (BIP 9, BIP 8, or Speedy Trial) is just a bit flag in the block header. Miners can and do use custom software to set this bit. At the same time, miners run full nodes that actually enforce the consensus rules. But if they don’t upgrade their own nodes, then their outdated nodes will simply ignore the flag, and their nodes won’t enforce the new rules. For that to happen, they need to actually upgrade their node software.

In general, it’s preferred if miners actually upgrade their nodes and don’t fake signal. That’s one reason why the timeout in BIP 9 was so long. But because Speedy Trial happened on such short notice, some may have considered it too risky to upgrade their software. Others ran into practical issues performing the upgrade. Mining pool operator Alejandro De La Torre described some of the practical issues he encountered in the field on a podcast episode.^[<https://stephanlivera.com/episode/277/>]

The accompanying episode goes into further detail about what, once Taproot activation became inevitable, needed to happen before it could ultimately be used on the Bitcoin network safely. We also explain how upcoming Bitcoin Core releases will handle the Taproot upgrade, especially with respect to its wallet software. At the time of writing, there’s some basic Taproot wallet support, but it’s still a work in progress.

### Moving Forward

Because Speedy Trial was successful, it’s possible we can use it as a template for soft fork activation moving forward. Or, we could interpret the lack of drama as an argument to just stick with BIP 9 or a `LOT=false` version of BIP 8. Perhaps some aspects of `LOT=true` deployment can be made safer.

Even if it’s inherently unsafe, it could make sense to continue developing it further, having the code already in place in case it’s ever needed. Perhaps the Bitcoin Core software could have generic support for it, even if the project itself recommends against using it. The best time to think about such matters is when they’re not yet needed.

### Burying Soft Forks

\EpisodeQR{54}

After all is said and done and a soft fork has activated, what do you do with the activation code? Is it merely a scaffold that can be removed once the new rules are active? Or is the activation mechanism itself a permanent part of the rules?

\noindent
As was done with previous soft forks, it looks like a future Bitcoin Core release will “bury” the Taproot activation. This means the node will treat the Taproot rules as if they’ve been active since Bitcoin’s very beginning. This is possible because, when applying these rules retroactively, only one historical block does not conform to them. This block can be grandfathered in.^[<https://github.com/bitcoin/bitcoin/pull/23536>]

In the episode we explain what the benefits are of burying a soft fork, in particular pointing out how it helps developers when they review the Bitcoin Core codebase or when they perform tests on it.

After that, we outline a potential edge case scenario where burying soft forks could, in a worst-case scenario, split the Bitcoin blockchain between upgraded and non-upgraded nodes. Bitcoin Core developers generally don’t consider this edge case — a very long block re-org — to be a realistic problem and/or believe that this would be such a big problem that a buried soft fork would be a minor concern comparatively. However, as we explain, not everyone agrees with this assessment entirely.
