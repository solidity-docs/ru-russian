# Солидити контрактно-ориентированный язык программирования

[![Matrix Chat](https://img.shields.io/badge/Matrix%20-chat-brightgreen?style=plastic&logo=matrix)](https://matrix.to/#/#ethereum_solidity:gitter.im)
[![Gitter Chat](https://img.shields.io/badge/Gitter%20-chat-brightgreen?style=plastic&logo=gitter)](https://gitter.im/ethereum/solidity)
[![Solidity Forum](https://img.shields.io/badge/Solidity_Forum%20-discuss-brightgreen?style=plastic&logo=discourse)](https://forum.soliditylang.org/)
[![Twitter Follow](https://img.shields.io/twitter/follow/solidity_lang?style=plastic&logo=twitter)](https://twitter.com/solidity_lang)
[![Mastodon Follow](https://img.shields.io/mastodon/follow/000335908?domain=https%3A%2F%2Ffosstodon.org%2F&logo=mastodon&style=plastic)](https://fosstodon.org/@solidity)

Вы можете связаться с нами в чатах Gitter и Matrix, упомянуть нас в Twitter или создать новую тему на форуме Solidity. Мы рады любым вопросам, обратной связи и предложениям! 

Solidity статически типизированный, контрактно-ориентированный, высокоуровневый язык для реализации смарт-контрактов в среде блокчейна Эфириума.

Чтобы с чего-то начать и получить хороший обзор, пожалуйста, ознакомьтесь с нашим официальным сайтом [Solidity Language Portal](https://soliditylang.org).

## Содержание

- [Определение](#background)
- [Сборка и установка](#build-and-install)
- [Пример](#example)
- [Документация](#documentation)
- [Разработка](#development)
- [Поддержка](#maintainers)
- [Лицензия](#license)
- [Безопасность](#security)

## Определение

Solidity - это статически типизированный фигурно-скобочный язык, созданный для разработки смарт контрактов, которые исполняются
на виртуальной машине Эфириума. Смарт контракты - это программы, которые исполняются внутри пиринговой сети, где ни у кого нет
исключительного права на исполнение, и таким образом контракты позволяют создавать токены стоимости, владения, голосования и других
типов логики.

Во время деплоя контрактов, Вы должны использовать последнюю релизную версию Солидити, потому что критические изменения, так же как и новые функции и исправления выпускаются регулярно. В данный момент мы используем нумерацию версий типа 0.x [чтобы отразить высокую скорость изменений](https://semver.org/#spec-item-4)

## Сборка и установка

Инструкции по сборке и установке компилятора Солидити можно найти в документации [Solidity documentation](https://docs.soliditylang.org/en/latest/installing-solidity.html#building-from-source).


## Пример

Простейшая программа "Привет мир" на языке Солидити еще более бесполезна, чем на других языках, но тем не менее:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.6.0 <0.9.0;

contract HelloWorld {
    function helloWorld() external pure returns (string memory) {
        return "Hello, World!";
    }
}
```

Для того, чтобы начать работу с Солидити, вы можете использовать [Remix](https://remix.ethereum.org/), который
представляет собой браузерную интегрированную среду разработки. Ниже приведено несколько примеров контрактов:

1. [Голосование](https://docs.soliditylang.org/en/latest/solidity-by-example.html#voting)
2. [Слепой аукцион](https://docs.soliditylang.org/en/latest/solidity-by-example.html#blind-auction)
3. [Безопасная удаленная покупка](https://docs.soliditylang.org/en/latest/solidity-by-example.html#safe-remote-purchase)
4. [Канал микроплатежей](https://docs.soliditylang.org/en/latest/solidity-by-example.html#micropayment-channel)

## Документация

Документация Солидити размещена по адресу: [Read the docs](https://docs.soliditylang.org).

## Разработка

Солидити всё ещё в процессе разработки. Любая помощь приветствуется! 
Пожалуйста изучите [Гид разработчика] [Developers Guide](https://docs.soliditylang.org/en/latest/contributing.html),
если вы хотите помочь.

Вы можете найти наши текущие приоритеты по исправлению ошибок и добавлению функций для предстоящих релизов в разделе проектов: [projects section](https://github.com/ethereum/solidity/projects).

## Поддержка
* [@axic](https://github.com/axic)
* [@chriseth](https://github.com/chriseth)

## Лицензия
Солидити лицензирован на следующих условиях [GNU General Public License v3.0](LICENSE.txt).

Часть третье-стороннего кода имеет собственные условия лицензирования [own licensing terms](cmake/templates/license.h.in).

## Безопасность

Политику безопасности можно найти здесь[found here](SECURITY.md).
