---
description: Как в разных языках программирования есть свои способы выражения концепций, так и в React есть свои идиомы - или правила - для выражения паттернов таким образом, чтобы их было легко понять и получить качественные приложения
---

# Правила React

<big>
Как в разных языках программирования есть свои способы выражения концепций, так и в React есть свои **идиомы** - или **правила** - для выражения паттернов таким образом, чтобы их было легко понять и получить качественные приложения.
</big>

!!!note "Мыслим как React"

    Чтобы узнать больше о выражении пользовательского интерфейса с помощью React, мы рекомендуем прочитать [Мыслим как React](../../learn/thinking-in-react.md).

В этом разделе описаны правила, которые необходимо соблюдать для написания идиоматического кода React. Написание идиоматического кода React поможет вам создавать хорошо организованные, безопасные и композитные приложения. Эти свойства делают ваше приложение более устойчивым к изменениям и облегчают работу с другими разработчиками, библиотеками и инструментами.

Эти правила известны как **Правила React**. Они являются правилами, а не просто рекомендациями, в том смысле, что если их нарушить, то в вашем приложении, скорее всего, будут ошибки. Ваш код также становится неидиоматичным, его сложнее понять и осмыслить.

Мы настоятельно рекомендуем использовать [Strict Mode](../react/StrictMode.md) вместе с плагином [ESLint](https://www.npmjs.com/package/eslint-plugin-react-hooks) для React, чтобы помочь вашей кодовой базе следовать правилам React. Следуя правилам React, вы сможете найти и устранить эти ошибки и сохранить работоспособность вашего приложения.

## Компоненты и хуки должны быть чистыми {#components-and-hooks-must-be-pure}

[Чистота компонентов и хуков](./components-and-hooks-must-be-pure.md) - это ключевое правило React, которое делает ваше приложение предсказуемым, легким для отладки и позволяет React автоматически оптимизировать ваш код.

-   [Компоненты должны быть идемпотентными](./components-and-hooks-must-be-pure.md#components-and-hooks-must-be-idempotent) - предполагается, что компоненты React всегда возвращают один и тот же результат относительно своих входов - props, state и context.
-   [Побочные эффекты должны выполняться вне рендера](./components-and-hooks-must-be-pure.md#side-effects-must-run-outside-of-render) - Побочные эффекты не должны выполняться во время рендера, так как React может рендерить компоненты несколько раз, чтобы создать наилучший пользовательский опыт.
-   [Пропсы и состояние неизменяемы](./components-and-hooks-must-be-pure.md#props-and-state-are-immutable) - Пропсы и состояние компонента являются неизменяемыми моментальными снимками по отношению к одному рендеру. Никогда не изменяйте их напрямую.
-   [Возвращаемые значения и аргументы хуков неизменяемы](./components-and-hooks-must-be-pure.md#return-values-and-arguments-to-hooks-are-immutable) - После передачи значений в хук их не следует изменять. Как и пропсы в JSX, значения становятся неизменяемыми при передаче в хук.
-   [Значения неизменяемы после передачи в JSX](./components-and-hooks-must-be-pure.md#values-are-immutable-after-being-passed-to-jsx) - Не изменяйте значения после того, как они были использованы в JSX. Переместите мутацию до создания JSX.

## React вызывает компоненты и хуки {#react-calls-components-and-hooks}

[React отвечает за рендеринг компонентов и хуков, когда это необходимо для оптимизации пользовательского опыта](./react-calls-components-and-hooks.md) Это декларативно: вы указываете React, что рендерить в логике вашего компонента, а React решает, как лучше отобразить это для пользователя.

-   [Никогда не вызывайте функции компонентов напрямую](./react-calls-components-and-hooks.md#never-call-component-functions-directly) - Компоненты должны использоваться только в JSX. Не вызывайте их как обычные функции.
-   [Никогда не передавайте хуки как обычные значения](./react-calls-components-and-hooks.md#never-pass-around-hooks-as-regular-values) - Хуки должны вызываться только внутри компонентов. Никогда не передавайте их как обычные значения.

## Правила использования хуков {#rules-of-hooks}

Хуки определяются с помощью функций JavaScript, но они представляют собой особый тип многократно используемой логики пользовательского интерфейса с ограничениями на то, где они могут быть вызваны. При их использовании необходимо следовать [Правилам использования хуков](./rules-of-hooks.md).

-   [Вызывать хуки только на верхнем уровне](./rules-of-hooks.md#only-call-hooks-at-the-top-level) - Не вызывайте хуки внутри циклов, условий или вложенных функций. Вместо этого всегда используйте Hooks на верхнем уровне вашей React-функции, перед любыми ранними возвратами.
-   [Вызывать хуки только из функций React](./rules-of-hooks.md#only-call-hooks-from-react-functions) - Не вызывайте хуки из обычных функций JavaScript.

<small>:material-information-outline: Источник &mdash; <https://react.dev/reference/rules></small>
