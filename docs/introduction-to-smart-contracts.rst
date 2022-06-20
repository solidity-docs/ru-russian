###############################
Введение в смарт-контракты
###############################

.. _simple-smart-contract:

***********************
Простой смарт-контракт
***********************

Позвольте нам начать с простого примера, в котором мы присваиваем значение переменной и
предоставляем к ней доступ для других контрактов. Ничего страшного, если вам что-то
будет непонятьно в этом примере, мы разберем детали позднее. 

Пример создания контракта с общедоступной переменной.
===============

.. Пример кода:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public view returns (uint) {
            return storedData;
        }
    


В первой строке указано, что исходный код предоставляется по лицензии
GPL версии 3.0 Важно указывать спецификатор лицензии в машинно-читаемом формате
в случаях, когда планируется сделать написанный код публично доступным.

Следующая строка указывает, что данный код написан для языка Солидити версии 0.4.16,
или более новой, за ислючением версий выше 0.9.0. 

Это сделано, чтобы акцентировать внимание на том, что данный контракт не совместим с новой(фундаментально отличающейся) 
версией компилятора, в котором данный код может вести себя по другому.

:Примечание:`Pragmas<pragma>` широко используемая инструкция для компиляторов, 
указывающая каким образом следует интерпретировать исходный код (подробнее `pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_).

Контракт(смарт-контракт) с позиции языка Солидити - это массив кода(его *функции*) и
данные(его *состояния*), которые размещены по определенному адресу в блокчейне Эфириума.
Строка ``uint storedData`` объявляет состояние переменной ``storedData`` типа ``uint``
(*u*\nsigned - беззнаковое, *int*\eger -  число, *256* битное). Вы можете рассматривать этот тип
данных, как одну ячейку базы данных, которую вы можете запрашивать и изменять с помощью вызова
функций программного кода, который управляет базой данных. В этом примере, контракт
объявляет функции ``set`` и ``get``, которые могу быть использованы для изменения
и запроса значения переменной.

Для получения доступа к элементу(например, к переменной состояния) данного контракта, вам не нужно,
как обычно добавлять префикс ``this``, вы просто обращаетесь к нему напрямую через его имя. В отличии от
некоторых других языков программирования, исключение префикса не является особенностью стиля, а приводит
к принципиально другому подходу в реализации доступа к элементам, но подробнее об этом позже.

Этот контракт ничего не делает, лишь(благодаря принципам работый инфраструктуры Эфириума) позволяет кому бы то ни было 
хранить едиственное число, к которому может получить доступ любой желающий, 
и исключает(какую-либо практическую) возможность уклониться от его публикации в открытый доступ.
Любой может вызвать функцию ''set'' снова и присвоить переменной другое значение, переписав ваше число,
но оно все равно будет сохранено в истории блокчейна. Позже, вы увидите, как можно наложить ограничения доступа
к функции таким образом, что только вы будете иметь право изменять это число.

.. Внимание::
    Будьте аккуратны с использованием текста в Unicode кодировке, так как одинаково выглядящие
    (или даже полностью одинаковые) символы могут иметь разный код в интерпретации Unicode и
    из-за этого шифруются как совершенно не идентичные байтовые массивы.
    
.. Замечание::
    Все идентификаторы(названия контрактов, имена функций и переменных) должны строго использовать
    систему кодирования ASCII. Также возможно хранить данные в переменных типа строка с использованием
    кодировки UTF-8. 
    
.. содержание:: ! Производная валюта(Сабкарренси)

Пример производной криптовалюты
===================

Следующий контракт представляет из себя простейшую форму криптовалюты.
Контракт позволяет создавать новые монеты только создателю контракта(схема создания может быть самая разнообразная).
Любой может отправить монеты кому угодно без необходимости регистрации с помощью имени пользователя и пароля,
все что вам нужно это пара ключей Эфириума.

.. Пример кода:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract Coin {
        // Ключевое слово "public" окрывает к переменной публичный
        // доступ из других контрактов
        address public minter;
        mapping (address => uint) public balances;

        // Событие(Event) позволяет клиентам реагировать на определенные,
        // объявленные вами, изменения в контракте
        event Sent(address from, address to, uint amount);

        // Код Constructor выполняется только в момент создания контракта
        constructor() {
            minter = msg.sender;
        }

        // Даная функция отправляет количество, определенное в переменной amount, вновь созданых монет на определеный адрес
        // Может быть вызвана только создателем контракта
        function mint(address receiver, uint amount) public {
            require(msg.sender == minter);
            balances[receiver] += amount;
        }

        // Обработчики ошибок позволяют вам вывести пользователю информацию
        // о причинах провала операции. Ответ возвращается к тому, кто вызвал функцию.
        error InsufficientBalance(uint requested, uint available);

        // Отправляет определенное в переменной amount количество существующих монет
        // с кошелька того, кто вызвал эту функцию на любой другой адрес.
        function send(address receiver, uint amount) public {
            if (amount > balances[msg.sender])
                revert InsufficientBalance({
                    requested: amount,
                    available: balances[msg.sender]
                });

            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

В этом контракте есть некоторые новые концепции, давайте рассмотрим их одну за другой.

Строка ''address public minter;'' объявляет постоянную переменную типа адрес 'address<address>'.
Тип ``address``(адрес) представляет собой 160-битное значение, с которым запрещено проводить какие-либо 
арифметические операции. Эта переменная подходит для хранения адресов контрактов или хэш значений публичной
части пары ключей от каких-либо внешних аккаунтов.

Ключевое слово ''public''(публичный) автоматически генерирует функцию, которая позволяет вам получать доступ
к текущему значению постоянной переменной запросами из-за пределов контракта. Без этого ключевого слова, другие
контракты не смогут получить доступ к этой переменной. Код функции, сгенерированный компилятором, представляет нечто подобное:
(пока что не обращайте внимание на ``external`` и ``view``)

.. Пример кода:: solidity

    function minter() external view returns (address) { return minter; }

Вы можете самостоятельно добавить в контракт подобную функцию, но в итое вы получите функцию и постоянную переменную с
таким же именем. Вам не нужно этого делать, комрилятор сделает это за вас.

.. содержание:: Мэппинг(Сопоставление)

Следующая линия ``mapping (address => uint) public balances;`` также создает
переменную публичного состояния, но это более сложный тип данных. 
mapping <mapping-types> тип сопоставляет адреса к типу беззнаковое число `unsigned integers <integers>`
The :ref:`mapping <mapping-types>` type maps addresses to :ref:`unsigned integers <integers>`.

Мэппинг может рассматриваться как Хэш таблицы <https://en.wikipedia.org/wiki/Hash_table>, которые
виртуально инициализированны таким образом, что каждый возможный ключ существует с самого начала
и мэппирован к значению, чье байтовое представление будет состоять из одних нулей(0). Однако,
это невозможно, получить список всех ключей мэппирования, как и список всех значений. Запишите, то что
вы добавили к мэппингу или используйте этот тип в контексте, где знание списка значений мэппинга не является
необходимым. Или даже лучше, сохраните список или используйте более подходящий тип данных.

Функция getter (Отсылка `getter function<getter-functions>`) создается ключевым словом ``public``
и является более сложным случаем мэппинга. Выглядит это примерно так:

.. Пример кода:: solidity

    function balances(address account) external view returns (uint) {
        return balances[account];
    }


Вы можете использовать эту функцию, чтобы получать баланс какого-либо одного аккаунта.


.. Содержание:: Событие(event)
    
Строка ``event Sent(address from, address to, uint amount);`` объявляет
событие (event), которое выпускается в последней линии функции ``send``.
Клиенты Эфириума, такие как веб-приложения, могут слушать такие события,
выпускаемые в блокчейне за небольшую цену. Как только событие произошло,
слушающее событие приложение получает аргументы ``from``(от кого), ``to``(кому) 
and ``amount``(количество), с помощью которых можно отследить транзации.

Чтобы слушать это событие, вы можете использовать следующий код JavaScript,
который использует библиотеку web3.js <https://github.com/ethereum/web3.js/>,
чтобы создать объект контракта ``Coin``, и любой пользовательский интерфейс
автоматически вызывающий сгенерированную  функцию ``balances`` указанную выше: 

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Coin transfer: " + result.args.amount +
                " coins were sent from " + result.args.from +
                " to " + result.args.to + ".");
            console.log("Balances now:\n" +
                "Sender: " + Coin.balances.call(result.args.from) +
                "Receiver: " + Coin.balances.call(result.args.to));
        }
    })

.. Содержание:: Монета(coin)

constructor - это специальная функция, которая выполняется во время создания контракта и после 
этого уже не может быть вызвана. В этом случая, она навсегда сохраняет адрес создателя контракта.
Переменная ``msg`` (вместе с ``tx`` и ``block``) является специальной глабальной переменной, которая
содержит свойства, которые позволяют взаимодействовать с блокчейном. ``msg.sender`` это всегда адрес
с которого вызывается текущая(внешняя) функция.

Функция которая создает и устанавливает контракт, а также пользователь, которые это делает
и сам контракт могут вызывать функции ``mint`` и ``send``.

Функция ``mint`` отправляет вновь созданные монеты на другой адрес. Метод функции require определяет
условия для выполнения функции, и если условия не выполняются, то выполнение отменяется. В это примере,
``require(msg.sender == minter);`` - эта строчка подразумевает, что только создатель контракта может
вызывать функцию ``mint``. В основном, создатель может выпустить столько токенов, сколько он хочет,
но с другой стороны это может привести к феномену, называемому "overflow"(переполнение). Обратите внимание,
что из-за проверки арифметических соотношений транзакция может отмениться, если выражение
``balances[receiver] += amount;``(баланс получателя = инкремент на определенное число) переполняет значение
переменной, например, когда увеличение баланса кошелька получателя на произвольное точное число, приводит
значение баланса кошелька к такому числу, которое превышеает максимально возможное значение в
беззнаковом 256 битном числе минус 1. Такая ситуация также возмжона для строки ``balances[receiver] += amount;``
в функции ``send``.

Функция ошибка(error) позволяет вам сопроводить ошибку большим количеством подробностей о том, почему
условие или операция из контракта не выполнились. Ошибки используются вместе с 
откатом состояния(revert statement). Откат состояния без каких-либо дополнительных условий
отменяет и восстанавливает все изменения, в изначальное состояние, подобно функции ``require``,
а также  позволяет сопроводить это указанием имени ошибки и дополнительной информацией,
которую получает вызвавший функцию, которая привела к ошибке(в конечном итоге
информация об ошибке выводится на фронт-энд приложения или в блок эксплорер). Сопровождение
ошибки дополнительной информацией позволяет облегчить отладку и правильно на нее отреагировать.

Функция ``send``(отправить) может быть использована любым пользователем(у кого уже есть
немного этих монет), чтобы отправить монеты кому-нибудь еще. Если у отправляющего недостаточно 
монет для отправки заявленной суммы, то условие ``if`` устанавливается в ``True``. В результате будет вызвана функция
``revert``, которая откатит все изменения инициированные пользователем, и отправка монет вернет ошибку 
``InsufficientBalance``, указывающую на недостаточность средств для отправки.

.. Замечание::
    Если вы используете этот контракт для отправки монет на определенный адрес, 
    то вы ничего не увидите, если попробуете получить данные об адресе через блокчейн
    эксплорер, потому что запись, которой вы отправляете монеты и измененные балансы
    сохраняются в данных конкретного контракта. Используя события, вы можете создать
    свой "блокчейн эксплорер", который будет отслеживать переводы и балансы вашей новой
    монеты, но вы должны будете отслеживать адреса контрактов вашей монеты, а не адреса
    владельцев монет.
 
.. основы блокчейна:

*****************
Основы блокчейна
*****************

Блокчейны как концепт довольно просты для понимания программистами. Причина в том, что основные
сложности(майнинг, хэширование <https://en.wikipedia.org/wiki/Cryptographic_hash_function>, эллиптическая криптография
<https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>, пиринговые сети <https://en.wikipedia.org/wiki/Peer-to-peer> и прочее) 
для понимания здесь в определенных особенностях и ожиданиях от платформы. Как только
вы принимаете эти особенности, как данность, вам не нужно больше беспокоиться о тех технологиях,
которые лежат "под капотом" - или вам обязательно знать как Amazon AWS работает внутри, чтобы использовать его?

.. Содержание:: Переводы(Транзакции)

Транзакции
============

Блокчейн - это глобально распределенная база транзакций. Это значит, что каждый может 
прочитать записи в базе с помощью участия в сети. Если вы хотите поменять что-то в 
базе данных, вы должны создать так называемую транзакцию, которую должны принять остальные
участники сети. Слово "транзакция" подразумевает, что изменение, которое вы хотите сделать
(допустим вы хотите изменить два значения в одно и то же время) либо не будет выполнена вообще
или будет полностью принята. К тому же, пока ваша транзакция применяется для изменения базы
данных, никакая другая транзакция не может на нее повлиять.

A blockchain is a globally shared, transactional database.
This means that everyone can read entries in the database just by participating in the network.
If you want to change something in the database, you have to create a so-called transaction
which has to be accepted by all others.
The word transaction implies that the change you want to make (assume you want to change
two values at the same time) is either not done at all or completely applied. Furthermore,
while your transaction is being applied to the database, no other transaction can alter it.

As an example, imagine a table that lists the balances of all accounts in an
electronic currency. If a transfer from one account to another is requested,
the transactional nature of the database ensures that if the amount is
subtracted from one account, it is always added to the other account. If due
to whatever reason, adding the amount to the target account is not possible,
the source account is also not modified.

Furthermore, a transaction is always cryptographically signed by the sender (creator).
This makes it straightforward to guard access to specific modifications of the
database. In the example of the electronic currency, a simple check ensures that
only the person holding the keys to the account can transfer money from it.

.. index:: ! block

Blocks
======

One major obstacle to overcome is what (in Bitcoin terms) is called a "double-spend attack":
What happens if two transactions exist in the network that both want to empty an account?
Only one of the transactions can be valid, typically the one that is accepted first.
The problem is that "first" is not an objective term in a peer-to-peer network.

The abstract answer to this is that you do not have to care. A globally accepted order of the transactions
will be selected for you, solving the conflict. The transactions will be bundled into what is called a "block"
and then they will be executed and distributed among all participating nodes.
If two transactions contradict each other, the one that ends up being second will
be rejected and not become part of the block.

These blocks form a linear sequence in time and that is where the word "blockchain"
derives from. Blocks are added to the chain in rather regular intervals - for
Ethereum this is roughly every 17 seconds.

As part of the "order selection mechanism" (which is called "mining") it may happen that
blocks are reverted from time to time, but only at the "tip" of the chain. The more
blocks are added on top of a particular block, the less likely this block will be reverted. So it might be that your transactions
are reverted and even removed from the blockchain, but the longer you wait, the less
likely it will be.

.. note::
    Transactions are not guaranteed to be included in the next block or any specific future block,
    since it is not up to the submitter of a transaction, but up to the miners to determine in which block the transaction is included.

    If you want to schedule future calls of your contract, you can use
    a smart contract automation tool or an oracle service.

.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

****************************
The Ethereum Virtual Machine
****************************

Overview
========

The Ethereum Virtual Machine or EVM is the runtime environment
for smart contracts in Ethereum. It is not only sandboxed but
actually completely isolated, which means that code running
inside the EVM has no access to network, filesystem or other processes.
Smart contracts even have limited access to other smart contracts.

.. index:: ! account, address, storage, balance

.. _accounts:

Accounts
========

There are two kinds of accounts in Ethereum which share the same
address space: **External accounts** that are controlled by
public-private key pairs (i.e. humans) and **contract accounts** which are
controlled by the code stored together with the account.

The address of an external account is determined from
the public key while the address of a contract is
determined at the time the contract is created
(it is derived from the creator address and the number
of transactions sent from that address, the so-called "nonce").

Regardless of whether or not the account stores code, the two types are
treated equally by the EVM.

Every account has a persistent key-value store mapping 256-bit words to 256-bit
words called **storage**.

Furthermore, every account has a **balance** in
Ether (in "Wei" to be exact, ``1 ether`` is ``10**18 wei``) which can be modified by sending transactions that
include Ether.

.. index:: ! transaction

Transactions
============

A transaction is a message that is sent from one account to another
account (which might be the same or empty, see below).
It can include binary data (which is called "payload") and Ether.

If the target account contains code, that code is executed and
the payload is provided as input data.

If the target account is not set (the transaction does not have
a recipient or the recipient is set to ``null``), the transaction
creates a **new contract**.
As already mentioned, the address of that contract is not
the zero address but an address derived from the sender and
its number of transactions sent (the "nonce"). The payload
of such a contract creation transaction is taken to be
EVM bytecode and executed. The output data of this execution is
permanently stored as the code of the contract.
This means that in order to create a contract, you do not
send the actual code of the contract, but in fact code that
returns that code when executed.

.. note::
  While a contract is being created, its code is still empty.
  Because of that, you should not call back into the
  contract under construction until its constructor has
  finished executing.

.. index:: ! gas, ! gas price

Gas
===

Upon creation, each transaction is charged with a certain amount of **gas**
that has to be paid for by the originator of the transaction (``tx.origin``).
While the EVM executes the
transaction, the gas is gradually depleted according to specific rules.
If the gas is used up at any point (i.e. it would be negative),
an out-of-gas exception is triggered, which ends execution and reverts all modifications
made to the state in the current call frame.

This mechanism incentivizes economical use of EVM execution time
and also compensates EVM executors (i.e. miners / stakers) for their work.
Since each block has a maximum amount of gas, it also limits the amount
of work needed to validate a block.

The **gas price** is a value set by the originator of the transaction, who
has to pay ``gas_price * gas`` up front to the EVM executor.
If some gas is left after execution, it is refunded to the transaction originator.
In case of an exception that reverts changes, already used up gas is not refunded.

Since EVM executors can choose to include a transaction or not,
transaction senders cannot abuse the system by setting a low gas price.

.. index:: ! storage, ! memory, ! stack

Storage, Memory and the Stack
=============================

The Ethereum Virtual Machine has three areas where it can store data:
storage, memory and the stack.

Each account has a data area called **storage**, which is persistent between function calls
and transactions.
Storage is a key-value store that maps 256-bit words to 256-bit words.
It is not possible to enumerate storage from within a contract, it is
comparatively costly to read, and even more to initialise and modify storage. Because of this cost,
you should minimize what you store in persistent storage to what the contract needs to run.
Store data like derived calculations, caching, and aggregates outside of the contract.
A contract can neither read nor write to any storage apart from its own.

The second data area is called **memory**, of which a contract obtains
a freshly cleared instance for each message call. Memory is linear and can be
addressed at byte level, but reads are limited to a width of 256 bits, while writes
can be either 8 bits or 256 bits wide. Memory is expanded by a word (256-bit), when
accessing (either reading or writing) a previously untouched memory word (i.e. any offset
within a word). At the time of expansion, the cost in gas must be paid. Memory is more
costly the larger it grows (it scales quadratically).

The EVM is not a register machine but a stack machine, so all
computations are performed on a data area called the **stack**. It has a maximum size of
1024 elements and contains words of 256 bits. Access to the stack is
limited to the top end in the following way:
It is possible to copy one of
the topmost 16 elements to the top of the stack or swap the
topmost element with one of the 16 elements below it.
All other operations take the topmost two (or one, or more, depending on
the operation) elements from the stack and push the result onto the stack.
Of course it is possible to move stack elements to storage or memory
in order to get deeper access to the stack,
but it is not possible to just access arbitrary elements deeper in the stack
without first removing the top of the stack.

.. index:: ! instruction

Instruction Set
===============

The instruction set of the EVM is kept minimal in order to avoid
incorrect or inconsistent implementations which could cause consensus problems.
All instructions operate on the basic data type, 256-bit words or on slices of memory
(or other byte arrays).
The usual arithmetic, bit, logical and comparison operations are present.
Conditional and unconditional jumps are possible. Furthermore,
contracts can access relevant properties of the current block
like its number and timestamp.

For a complete list, please see the :ref:`list of opcodes <opcodes>` as part of the inline
assembly documentation.

.. index:: ! message call, function;call

Message Calls
=============

Contracts can call other contracts or send Ether to non-contract
accounts by the means of message calls. Message calls are similar
to transactions, in that they have a source, a target, data payload,
Ether, gas and return data. In fact, every transaction consists of
a top-level message call which in turn can create further message calls.

A contract can decide how much of its remaining **gas** should be sent
with the inner message call and how much it wants to retain.
If an out-of-gas exception happens in the inner call (or any
other exception), this will be signaled by an error value put onto the stack.
In this case, only the gas sent together with the call is used up.
In Solidity, the calling contract causes a manual exception by default in
such situations, so that exceptions "bubble up" the call stack.

As already said, the called contract (which can be the same as the caller)
will receive a freshly cleared instance of memory and has access to the
call payload - which will be provided in a separate area called the **calldata**.
After it has finished execution, it can return data which will be stored at
a location in the caller's memory preallocated by the caller.
All such calls are fully synchronous.

Calls are **limited** to a depth of 1024, which means that for more complex
operations, loops should be preferred over recursive calls. Furthermore,
only 63/64th of the gas can be forwarded in a message call, which causes a
depth limit of a little less than 1000 in practice.

.. index:: delegatecall, callcode, library

Delegatecall / Callcode and Libraries
=====================================

There exists a special variant of a message call, named **delegatecall**
which is identical to a message call apart from the fact that
the code at the target address is executed in the context (i.e. at the address) of the calling
contract and ``msg.sender`` and ``msg.value`` do not change their values.

This means that a contract can dynamically load code from a different
address at runtime. Storage, current address and balance still
refer to the calling contract, only the code is taken from the called address.

This makes it possible to implement the "library" feature in Solidity:
Reusable library code that can be applied to a contract's storage, e.g. in
order to implement a complex data structure.

.. index:: log

Logs
====

It is possible to store data in a specially indexed data structure
that maps all the way up to the block level. This feature called **logs**
is used by Solidity in order to implement :ref:`events <events>`.
Contracts cannot access log data after it has been created, but they
can be efficiently accessed from outside the blockchain.
Since some part of the log data is stored in `bloom filters <https://en.wikipedia.org/wiki/Bloom_filter>`_, it is
possible to search for this data in an efficient and cryptographically
secure way, so network peers that do not download the whole blockchain
(so-called "light clients") can still find these logs.

.. index:: contract creation

Create
======

Contracts can even create other contracts using a special opcode (i.e.
they do not simply call the zero address as a transaction would). The only difference between
these **create calls** and normal message calls is that the payload data is
executed and the result stored as code and the caller / creator
receives the address of the new contract on the stack.

.. index:: selfdestruct, self-destruct, deactivate

Deactivate and Self-destruct
============================

The only way to remove code from the blockchain is when a contract at that
address performs the ``selfdestruct`` operation. The remaining Ether stored
at that address is sent to a designated target and then the storage and code
is removed from the state. Removing the contract in theory sounds like a good
idea, but it is potentially dangerous, as if someone sends Ether to removed
contracts, the Ether is forever lost.

.. warning::
    Even if a contract is removed by ``selfdestruct``, it is still part of the
    history of the blockchain and probably retained by most Ethereum nodes.
    So using ``selfdestruct`` is not the same as deleting data from a hard disk.

.. note::
    Even if a contract's code does not contain a call to ``selfdestruct``,
    it can still perform that operation using ``delegatecall`` or ``callcode``.

If you want to deactivate your contracts, you should instead **disable** them
by changing some internal state which causes all functions to revert. This
makes it impossible to use the contract, as it returns Ether immediately.


.. index:: ! precompiled contracts, ! precompiles, ! contract;precompiled

.. _precompiledContracts:

Precompiled Contracts
=====================

There is a small set of contract addresses that are special:
The address range between ``1`` and (including) ``8`` contains
"precompiled contracts" that can be called as any other contract
but their behaviour (and their gas consumption) is not defined
by EVM code stored at that address (they do not contain code)
but instead is implemented in the EVM execution environment itself.

Different EVM-compatible chains might use a different set of
precompiled contracts. It might also be possible that new
precompiled contracts are added to the Ethereum main chain in the future,
but you can reasonably expect them to always be in the range between
``1`` and ``0xffff`` (inclusive).
