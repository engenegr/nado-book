\newpage
## Script, P2SH и Miniscript {#sec:miniscript}

\EpisodeQR{4}

В этой главе мы поговорим о Miniscript и о том, как он значительно упрощает использование Bitcoin Script. Мы расскажем, как работает Script, как с его помощью можно делать более сложные и даже абсурдные вещи, и как появился Miniscript, чтобы сделать транзакции менее сложными и более безопасными. Кроме того, в главе будет рассказано о том, какова политика этого языка, и как она может упростить создание сценариев.

### Ограничения

Скрипты — это о том, как блокчейн Биткоина ограничивает возможности траты некой конкретной монеты: когда вы хотите получить биткоин, вы сообщаете человеку, отправляющему его, какие правила применяются к транзакции. Например, без каких-либо ограничений любой может посмотреть, что находится в вашем кошельке. Обычно вы добавляете ограничение, что только вы можете тратить эти монеты. Или, если быть более точным, «эти монеты могут быть потрачены только в том случае, если транзакция включает в себя подпись, сделанную с помощью определенного открытого ключа, а именно моего ключа».

Таким образом, вы должны сообщить человеку, отправляющему биткоины, либо ваш открытый ключ, либо его хэш. Для этого и нужны адреса, как мы объясняли в главе @sec:address. Затем отправитель должен поместить их в блокчейн с пометкой о том, что только владелец этого открытого ключа может тратить биткоины.

Однако, хотя это, безусловно, самый распространенный тип ограничения, существует множество других типов ограничений, и вы даже можете указать несколько различных вариантов для того, кто тратит, например: «Я могу потратить это, но для этого моей маме также нужно подписать транзакцию. Но через два года, может быть, я смогу подписать ее и в одиночку». В таком сценарии, если вы хотите потратить деньги, вам нужно указать, какой вариант вы используете, и выполнить только определенные критерии для этого варианта.

### Как работает Script

Script^[<https://en.bitcoin.it/wiki/Script>] — это язык, основанный на стеке, поэтому думайте о нем как о стопке тарелок. На стопку можно ставить тарелки, можно снимать верхнюю тарелку, но нельзя манипулировать тарелками посередине.

Стек работает иначе, чем обычная память, где вы можете читать и записывать произвольные адреса (например, жесткий диск или ОЗУ — память с произвольным доступом). Стек легче реализовать и представить. ^ [Напротив, смарт-контракты Ethereum имеют стек, а также обычную память и даже долгосрочное хранилище. Как следствие, разработчикам гораздо труднее представлять себе его поведение. <https://dlt-repo.net/storage-vs-memory-vs-stack-in-solidity-ethereum/>].

Наиболее часто используемый (до SegWit, см. главу @sec:segwit) биткоин-скрипт выглядит следующим образом:

- `OP_DUP` (как в дубликате)
- `OP_HASH160` (который дважды берет хэш SHA-256, а затем хэш RIPEMD-160)
- `pubKeyHash` (хэш приватного ключа)
- `OP_EQUAL_VERIFY`
- `OP_CHECKSIG`

Значение для `pubKeyHash` генерируется кошельком получателя путем выполнения двойного хэширования SHA-256, за которым следует RIPEMD-160, и результат вставляется в приведенный выше скрипт. Как объяснялось в главе @sec:address, _биткоин-адрес_ содержит только `pubKeyHash`; остальное подразумевается. На самом деле именно кошелек отправителя генерирует полный скрипт перед его публикацией в блокчейне.

В блокчейне этот скрипт заканчивается выходом транзакции. Выход транзакции, то есть монета, состоит из этого скрипта, которым она заблокирована (`scriptPubKey`) и суммы. Теперь, если вы хотите потратить монеты, вы создаете вход транзакции, который дает указание блокчейну добавить определенные вещи в стек _перед_ выполнением вышеприведенного скрипта.

Интерпретатор Биткоина увидит, что вы поместили в стек, и начнет выполнять программу, начиная с выхода. В этом случае то, что вы помещаете в стек — это ваша подпись и ваш открытый ключ, потому что исходный скрипт не имел вашего открытого ключа; у него был только его хэш.

Продолжая вышеприведенный пример, мы начнем со стопки, состоящей из двух тарелок. Тарелка внизу это ваша подпись, а сверху тарелка с вашим открытым ключом, а дальше скрипт говорит `OP_DUP`. Это производит операцию pop, т. е. скрипт берет верхний элемент из стека — верхнюю тарелку, которая является открытым ключом — и дублирует его. Затем оригинал и дубликат помещаются в стек. Итак, теперь у вас две тарелки с публичным ключом наверху стека, а ваша подпись по-прежнему внизу.
  
Следующая инструкция — «OP_HASH160». Она извлекает один из этих двух дубликатов открытых ключей из стека и хэширует его, а затем помещает этот хэш в стек.

Подпись все еще внизу, потом публичный ключ, а потом хэш публичного ключа (три тарелки).

Следующая операция — «pubKeyHash», которая помещает хэш вашего открытого ключа в стек. Итак, теперь хэш вашего открытого ключа дважды повторен на вершине стека.

Операция OP_EQUAL_VERIFY извлекает оба этих хэша из стека и проверяет, совпадают ли они.

В стеке остается ваша подпись и ваш открытый ключ, поэтому `OP_CHECKSIG` проверяет подпись с помощью вашего открытого ключа, после чего стек оказывается пустым.

Именно так, в двух словах, работает программа на языке Script, и в ходе ее выполнения вы можете делать сколь угодно сложные вещи.

### Script Hash and P2SH

В общем, всякий раз, когда вы хотите получить от кого-то монеты, вы должны точно указать, какой скрипт использовать. В приведенном выше примере все, что нужно — это предоставить хэш открытого ключа в стандартном формате адреса, и кошелек отправителя создаст корректный скрипт.

Но в более сложном примере, приведенном ранее, с альтернативными условиями, такими как наличие подписи родителя через несколько лет, сообщать об этом становится неудобно. Даже если бы для подобного существовал стандарт адреса, ради учета всех возможных ограничений это был бы чрезвычайно длинный адрес.

К счастью, есть альтернатива передаче контрагенту (отправителю) полного скрипта — вы можете передать ему хэш скрипта, который всегда имеет одинаковую длину, причем точно такую же длину, что и обычный адрес.

В 2012 году был введен стандарт Pay-to-Script-Hash (P2SH).^[<https://en.bitcoin.it/wiki/BIP_0016>] Эти виды транзакций позволяют вам использовать в качестве адреса отправления хэш скрипта (такой адрес должен начинаться с 3), вместо отправки на хэш открытого ключа (такой адрес начинался с 1).

Находящийся на другом конце должен скопипастить этот адрес в свой биткоин-кошелек и отправить на него деньги. Теперь, когда вы хотите потратить эти деньги, вам нужно открыть блокчейну фактический скрипт, который ваш кошелек далее автоматически обработает. Поскольку все, что вам для этого нужно, это хэш, человеку, который отправляет вам деньги, не нужно заботиться о том, что на самом деле скрывается за этим хэшем. Только когда вы тратите монеты, вам нужно раскрыть ограничения. С точки зрения приватности это намного лучше, чем сразу вставлять скрипт в блокчейн. В главе @sec:taproot_basics объясняется, как Taproot идет еще дальше.

Как и в случае с обычными адресами P2PKH, то, что вы сообщаете отправителю - это просто хэш скрипта. Прежде чем кошелек отправителя поместит его в блокчейн, он добавляет в начало OP_HASH160 и в конец OP_EQUAL. Так что это, по сути, скрипт внутри скрипта. Внешний скрипт, который кошелек помещает в блокчейн, сообщает блокчейну, что существует внутренний скрипт, который должен быть раскрыт _и исполнен_ получателем, и тогда с него можно тратить деньги.

Это последнее требование на самом деле не следует из скрипта в блокчейне, для которого требуется только совпадение хэша скрипта. Вот почему новый тип адреса P2SH появился с софт-форком, чтобы гарантировать, что, когда такой скрипт находится внутри скрипта, он также выполняется. Обычно это означает, что плательщик помещает в стек не только скрипт, но и ингредиенты, необходимые для выполнения скрипта, такие как подпись.

### Really Absurd Things

Script is a programming language that was introduced in Bitcoin, though it resembles a preexisting language known as Forth. It also seems to have been cobbled together as an afterthought. In fact, a lot of the operations that were part of the language were removed almost immediately, because there were all sorts of ways that you could just crash a node or do other bad things.^[
Unfortunately, with Bitcoin, you can’t just start with a draft language and then clean it up later. But this only became clear once developers realized the only safe way to upgrade Bitcoin is through very carefully crafted soft forks. Every change has to be backward compatible and not break any existing script. But developers can’t always know the intention of scripts that are already out there, and worse still, as explained above, most scripts are hashed, so they could contain anything.
<!-- The double \ is needed to separate paragraphs in a footnote. Without it everything but the first paragraph disappears entirely. -->
\
\
As a result, it’s been a complete nightmare to make sure upgrades to the Script language don’t do anything surprising or bad. If it turns out that existing nodes can be negatively impacted, e.g. crashed, by some obscure script, developers have to very carefully work around that issue; they have to fix the problem without accidentally making coins unspendable and without introducing new bugs, including in any unknown (hashed) script potentially out there.
\
\
Worse still, because Bitcoin is a live system and users can’t be forced to all update at once, an ideal fix should not tip off an attacker as to what the issue is. But at the same time, it’s an open source and transparent system, where changes can’t go through without public justification. This makes Responsible Disclosure (<https://github.com/bitcoin/bitcoin/blob/master/SECURITY.md>) very complicated. So it’s really best to go above and beyond to avoid such problems in first place.
]

The script’s language is diverse enough to allow for weird stuff. If you just want somebody to send money to you, you only need this very simple standard script explained above: `OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG`.

But let’s say you’re collaborating, and you want to do a multi-signature, or multisig. To spend coins, two signatures need to provided, rather than just one. Now you could just use `OP_CHECKMULTISIG`, but let’s say that didn’t yet exist. Instead, you could take the script for one signature from above and more or less duplicate it, like so: `<KEY_A> OP_CHECKSIGVERIFY <KEY_B> OP_CHECKSIG`. In this example you’re B, the second key that’s checked (we also don’t bother with hashing the public keys).

Essentially, if you start with those two public keys and two signatures on the stack, and you run the script one instruction at a time, then if A and B put a valid signature on the stack, it’s all good. This would be a poor man’s multisig.

However, a malicious actor could insert an op code called `OP_RETURN` in the middle: `<KEY_A> OP_CHECKSIGVERIFY OP_RETURN <KEY_B> OP_CHECKSIG`.

This `OP_RETURN` code instructs the blockchain to stop evaluating the program — in other words, skipping the signature check for B, your signature.

If you naively looked at this script, you might think that your signature is checked at the end, and so the rest of the script isn’t relevant. If you had a vigilant electronic lawyer (i.e. a person or computer program that does due diligence on transactions), who would properly check that this “smart contract” does what it says it does, they might say, “Careful there, your signature isn’t getting checked.” This hypothetical electronic lawyer should see that `OP_RETURN` “fine print” and warn you. But the problem is there are countless ways in which scripts can go wrong, which is why we need a standardized way of dealing with these scripts.

In an interview with Bitcoin Magazine,^[<https://bitcoinmagazine.com/technical/miniscript-how-blockstream-engineers-are-making-bitcoin-programming-easyer>] Andrew Poelstra said, “There are opcodes in Bitcoin Script which do really absurd things, like, interpret a signature as a true/false value, branch on that; convert that boolean to a number and then index into the stack, and rearrange the stack based on that number. And the specific rules for how it does this are super nuts.”

This quote exemplifies the complexity of potential ways to mess around with script.

To return to the plate analogy, you’d take a hammer and smash one, and then you’d confuse two and paint one red and then it would still work, if you do it correctly. It’s completely absurd.

So that’s the long^[If you can’t get enough of this, watch Andrew Poelstra’s two-hour presentation at London Bitcoin Devs, where he goes on and on and on about the problems in script: <https://www.youtube.com/watch?v=_v1lECxNDiM>] and short of the problem with scripts: It’s easy to make mistakes or hide bugs and make all sorts of complex arrangements that people might or might not notice. And then your money goes places you don’t want it to go. We’ve already seen in other projects, famously with the Ethereum DAO hack and resulting hard fork,^[<https://ogucluturk.medium.com/the-dao-hack-explained-unfortunate-take-off-of-smart-contracts-2bd8c8db3562>] how bad things can get if you have a very complicated language that does things you’re not completely expecting. But Bitcoin dodged many bullets in the early days, and despite its relative simplicity,^[<https://blockstream.com/2018/11/28/en-simplicity-github/>] it still requires vigilance.

### Enter Miniscript

Miniscript^[<https://medium.com/blockstream/miniscript-bitcoin-scripting-3aeff3853620>] is a project that was designed by a few Blockstream engineers: Pieter Wuille, Andrew Poelstra, and Sanket Kanjalkar. It’s “a language for writing (a subset of) Bitcoin Scripts in a structured way, enabling analysis, composition, generic signing and more.” You can see examples and try it yourself at <https://bitcoin.sipa.be/miniscript>.

Miniscript consists of a few dozen script fragments, each a sequence of op codes. These fragments can be combined. If individual script op codes are like an alphabet then Miniscript fragments are like words. By building a script that only uses these words, rather than just any combination of letters from the alphabet, you lose some Script features, but you gain certain guarantees about safety and correct behavior.

A simple example of Miniscript is `pkh(A)` — which consists of only a single fragment. It’s the equivalent of the standard P2PKH script analyzed above (`OP_DUP OP_HASH160 <pubKeyHashA> OP_EQUALVERIFY OP_CHECKSIG`). The poor man’s multisig above requires several Miniscript fragments: `and_v(v:pk(pubKeyA),pk(pubKeyB))`.

Miniscript makes sure there’s no funny stuff in the fine print. It removes some of the foot guns,^[An unsafe piece of code that causes users to shoot themselves in the foot. Early Bitcoin developer Gregory Maxwell was using this term as early as 2012, see e.g. <https://github.com/bitcoin/bitcoin/pull/1889#issuecomment-9638527>, but it may be older] but it also allows you to do very cool stuff safely. In particular, it lets you do things like `AND`. So you can say condition `A` must be true `AND` condition `B` must be true, and you can do things like `OR`. And whatever’s inside the `OR` or inside the `AND` can be arbitrarily complex.

In contrast, with Bitcoin Script, you have `if` and `else` statements, but if you’re not careful, those `if` and `else` statements won’t do what you think they’re going to do, because there’s complexity hidden after these statements.

Meanwhile, with Miniscript, the templates make sure you’re only doing things that are actually doing what you think they’re doing. Let’s say you’re a company and you offer a semi-custodial wallet solution, where you have one of the keys of the user and the user has the other has two keys. You don’t have a majority of the keys, but maybe there’s a five-year timeout where you do have control in case the user dies or something else happens.

This would be like a multisig set up. Normally, when you set up a multisig, everybody gives their public key^[Usually everyone would provide not just one public key, but a whole series of public keys, by using an extended public key, or xpub] for example, and you create a simple script that has three keys and three people sign. But the problem is, because you’re a big business that offers a service, you have some really complicated internal accounting department and you maybe want to have five different signatures by specific people with varied levels of complexity.

There’s a lot of complex stuff you can do with it, and all the complex stuff should count as one key.

The problem with that is how does the customer know the script is OK? They’d have to hire their own electronic lawyer to check that the script doesn’t have any little gimmicks in it.

Miniscript allows you to check that. A futuristic wallet could show you a little pie chart, saying “You’re this one piece of the pie, and there’s this other piece of the pie that’s really complicated, but you don’t have to worry about it. It’s not going to do anything sneaky.”

### Policy Language

A policy language is a way to express your intentions. It’s easier than writing a Miniscript directly, let alone writing Bitcoin Script directly. A compiler then does the hard work.

Our earlier example of a poor man’s multisig was actually found this way. Starting with a policy `and(pk(KEY_A),pk(KEY_B))`, the compiler produced `and_v(v:pk(KEY_A),pk(KEY_B))`, which is equivalent to the script `<KEY_A> OP_CHECKSIGVERIFY <KEY_B> OP_CHECKSIG`. It turns out this actually produces a lower fee transaction than `<KEY_A> <KEY_B> 2 OP_CHECKMULTISIG`. This is the kind of optimization a human might overlook, which is what compilers are good for.

Basically, you write a policy language, which is like a higher-level programming language, which the compiler turns into low level op codes. These are instructions like the ones we described above for popping things off the stack and duplicating them. Miniscript also lives at that very low level, even if it’s slightly more readable and a lot safer. It’s the compiler’s job to take a high level language like Policy Language and turn into the most efficient low-level code.

In the case of multisig, you might say, “I just want two out of two signatures. I don’t care how you do that.” The compiler knows there are multiple ways to execute the intention. And then, the question is, which of them will be picked? The answer to that depends on the transaction weight and the fees that might be involved.

However, you can also tell the compiler, “OK, I think most of the time it’s condition A, but only 10 percent of the time it’s condition B.” The compiler would then calculate the fee for condition A, multiply by 9, add the fee for condition B and divide the total by 10 in order to get the average expected fee. It can optimize for typical use cases, worst case scenarios, all these things, and it then spits out a Miniscript which can then be transpiled to Bitcoin Script.^[The technical term for going from Miniscript to Script — or for transforming source code from any language into another similar one — is transpiling, which can basically be done in two directions. So you can go from Miniscript to Script, or from Script to Miniscript, but you can’t trivially go back to a policy language. However, using automated analysis tools, you can often still figure out what policy language was used to produce a given piece of Miniscript.]

With Taproot (see chapter @sec:taproot_basics), rather than splitting different conditions using _and_ / _or_, they can be split into a Merkle tree of scripts. You don’t have to worry about how to build the Merkle tree, as the compiler takes care of that. In principle, each leaf can also contain _and_ / _or_ statements. Does it make sense to do that? Or is it better to stick to one condition per leaf? Who knows? A future Miniscript compiler can just try all permutations and decide what’s optimal.

### Limitations

All this said, there are some limitations when you’re using policy language or Miniscript in general.

To ensure Miniscript and its corresponding Bitcoin Script can be safely reasoned about, it does not provide access to the full power of script. Sometimes, however, doing things safely results in a script that’s unacceptably long and expensive to execute. In that case, a human may be able to construct a better solution. In regard to the example Poelstra mentioned of how transactions for the Lightning network deal with time locks, hashes, or nonces, there are some optimizations. As he put it: “Oh, you do some weird switching of the stack and you interpret things, not the way they were.” You put a public key on it, but you interpret it as a number — those kind of weird tricks.

Those might be very hard to reason about, and a human might be able to do it, but the Miniscript compiler wouldn’t, which means the compiler would end up with potentially longer Lightning scripts. Perhaps one day Miniscript can be expanded so it can also find these shortcuts. But the Miniscript developers have to be careful, because they really want to make sure there’s nothing in Miniscript that brings back the scary properties of the underlying language.

Another limitation is policy language is just one of several tools needed to make very complicated multisig wallets a practical reality. There are still questions left to answer, such as: How exactly do you do this setup? What are you emailing to each other? Are you emailing your keys or are you emailing something a little bit more abstract that you agree on first and then you exchange keys? These are practical things that aren’t solved inside a Miniscript.

At the time of writing, integrating Miniscript into the Bitcoin Core wallet is still very much a work in progress.^[<https://github.com/bitcoin/bitcoin/pull/24149>]
