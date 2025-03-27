# API HMR

:::tip Примечание
Это клиентский API HMR. Для обработки обновлений HMR в плагинах см. [handleHotUpdate](./api-plugin#handlehotupdate).

Ручной API HMR в первую очередь предназначен для авторов фреймворков и инструментов. Как конечный пользователь, HMR, вероятно, уже обрабатывается для вас в стартовых шаблонах конкретного фреймворка.
:::

Vite раскрывает свой ручной API HMR через специальный объект `import.meta.hot`:

```ts twoslash
import type { ModuleNamespace } from 'vite/types/hot.d.ts'
import type {
  CustomEventMap,
  InferCustomEventPayload,
} from 'vite/types/customEvent.d.ts'

// ---cut---
interface ImportMeta {
  readonly hot?: ViteHotContext
}

interface ViteHotContext {
  readonly data: any

  accept(): void
  accept(cb: (mod: ModuleNamespace | undefined) => void): void
  accept(dep: string, cb: (mod: ModuleNamespace | undefined) => void): void
  accept(
    deps: readonly string[],
    cb: (mods: Array<ModuleNamespace | undefined>) => void,
  ): void

  dispose(cb: (data: any) => void): void
  prune(cb: (data: any) => void): void
  invalidate(message?: string): void

  on<T extends keyof CustomEventMap>(
    event: T,
    cb: (payload: InferCustomEventPayload<T>) => void,
  ): void
  off<T extends keyof CustomEventMap>(
    event: T,
    cb: (payload: InferCustomEventPayload<T>) => void,
  ): void
  send<T extends keyof CustomEventMap>(
    event: T,
    data?: InferCustomEventPayload<T>,
  ): void
}
```

## Обязательная условная защита

Прежде всего, убедитесь, что все использование API HMR защищено условным блоком, чтобы код мог быть удален при сборке в production:

```js
if (import.meta.hot) {
  // Код HMR
}
```

## Поддержка IntelliSense для TypeScript

Vite предоставляет определения типов для `import.meta.hot` в [`vite/client.d.ts`](https://github.com/vitejs/vite/blob/main/packages/vite/client.d.ts). Вы можете создать `env.d.ts` в директории `src`, чтобы TypeScript подхватывал определения типов:

```ts
/// <reference types="vite/client" />
```

## `hot.accept(cb)`

Для самопринятия модулем обновлений используйте `import.meta.hot.accept` с callback-функцией, которая получает обновленный модуль:

```js twoslash
import 'vite/client'
// ---cut---
export const count = 1

if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    if (newModule) {
      // newModule равен undefined, если произошла ошибка синтаксиса
      console.log('обновлено: count теперь ', newModule.count)
    }
  })
}
```

Модуль, который "принимает" горячие обновления, считается **границей HMR**.

HMR Vite фактически не заменяет изначально импортированный модуль: если модуль-граница HMR реэкспортирует импорты из зависимости, то он отвечает за обновление этих реэкспортов (и эти экспорты должны использовать `let`). Кроме того, импортеры выше по цепочке от модуля-границы не будут уведомлены об изменении. Эта упрощенная реализация HMR достаточна для большинства случаев использования в разработке, при этом позволяя нам пропустить дорогостоящую работу по генерации прокси-модулей.

Vite требует, чтобы вызов этой функции появлялся как `import.meta.hot.accept(` (с учетом пробелов) в исходном коде для того, чтобы модуль принимал обновления. Это требование статического анализа, который Vite выполняет для включения поддержки HMR для модуля.

## `hot.accept(deps, cb)`

Модуль также может принимать обновления от прямых зависимостей без перезагрузки самого себя:

```js twoslash
// @filename: /foo.d.ts
export declare const foo: () => void

// @filename: /example.js
import 'vite/client'
// ---cut---
import { foo } from './foo.js'

foo()

if (import.meta.hot) {
  import.meta.hot.accept('./foo.js', (newFoo) => {
    // callback получает обновленный модуль './foo.js'
    newFoo?.foo()
  })

  // Также можно принять массив модулей-зависимостей:
  import.meta.hot.accept(
    ['./foo.js', './bar.js'],
    ([newFooModule, newBarModule]) => {
      // callback получает массив, где только обновленный модуль
      // не равен null. Если обновление не удалось (например, ошибка синтаксиса),
      // массив пуст
    },
  )
}
```

## `hot.dispose(cb)`

Самопринимающий модуль или модуль, который ожидает быть принятым другими, может использовать `hot.dispose` для очистки любых постоянных побочных эффектов, созданных его обновленной копией:

```js twoslash
import 'vite/client'
// ---cut---
function setupSideEffect() {}

setupSideEffect()

if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    // очистка побочного эффекта
  })
}
```

## `hot.prune(cb)`

Регистрирует callback, который будет вызван, когда модуль больше не импортируется на странице. По сравнению с `hot.dispose`, это можно использовать, если исходный код сам очищает побочные эффекты при обновлениях, и вам нужно очищать только когда он удаляется со страницы. Vite в настоящее время использует это для импортов `.css`.

```js twoslash
import 'vite/client'
// ---cut---
function setupOrReuseSideEffect() {}

setupOrReuseSideEffect()

if (import.meta.hot) {
  import.meta.hot.prune((data) => {
    // очистка побочного эффекта
  })
}
```

## `hot.data`

Объект `import.meta.hot.data` сохраняется между разными экземплярами одного и того же обновленного модуля. Его можно использовать для передачи информации от предыдущей версии модуля к следующей.

Обратите внимание, что переназначение самого `data` не поддерживается. Вместо этого вы должны изменять свойства объекта `data`, чтобы информация, добавленная из других обработчиков, сохранялась.

```js twoslash
import 'vite/client'
// ---cut---
// правильно
import.meta.hot.data.someValue = 'hello'

// не поддерживается
import.meta.hot.data = { someValue: 'hello' }
```

## `hot.decline()`

В настоящее время это пустая операция и существует для обратной совместимости. Это может измениться в будущем, если появится новое применение. Чтобы указать, что модуль не поддерживает горячее обновление, используйте `hot.invalidate()`.

## `hot.invalidate(message?: string)`

Самопринимающий модуль может во время выполнения понять, что он не может обработать обновление HMR, и поэтому обновление нужно принудительно распространить на импортеры. Вызовом `import.meta.hot.invalidate()` сервер HMR сделает недействительными импортеры вызывающего модуля, как если бы вызывающий модуль не был самопринимающим. Это выведет сообщение как в консоль браузера, так и в терминал. Вы можете передать сообщение, чтобы дать контекст о том, почему произошла инвалидация.

Обратите внимание, что вы всегда должны вызывать `import.meta.hot.accept`, даже если планируете сразу после этого вызвать `invalidate`, иначе клиент HMR не будет слушать будущие изменения самопринимающего модуля. Чтобы четко выразить ваше намерение, мы рекомендуем вызывать `invalidate` внутри callback-функции `accept`, как показано ниже:

```js twoslash
import 'vite/client'
// ---cut---
import.meta.hot.accept((module) => {
  // Вы можете использовать новый экземпляр модуля, чтобы решить, нужно ли инвалидировать.
  if (cannotHandleUpdate(module)) {
    import.meta.hot.invalidate()
  }
})
```

## `hot.on(event, cb)`

Слушать событие HMR.

Следующие события HMR автоматически отправляются Vite:

- `'vite:beforeUpdate'` когда обновление вот-вот будет применено (например, модуль будет заменен)
- `'vite:afterUpdate'` когда обновление только что было применено (например, модуль был заменен)
- `'vite:beforeFullReload'` когда вот-вот произойдет полная перезагрузка
- `'vite:beforePrune'` когда модули, которые больше не нужны, вот-вот будут удалены
- `'vite:invalidate'` когда модуль инвалидирован с помощью `import.meta.hot.invalidate()`
- `'vite:error'` когда происходит ошибка (например, ошибка синтаксиса)
- `'vite:ws:disconnect'` когда соединение WebSocket потеряно
- `'vite:ws:connect'` когда соединение WebSocket (пере)установлено

Пользовательские события HMR также могут отправляться из плагинов. Подробнее см. [handleHotUpdate](./api-plugin#handlehotupdate).

## `hot.off(event, cb)`

Удаляет callback из слушателей событий.

## `hot.send(event, data)`

Отправляет пользовательские события обратно на сервер разработки Vite.

Если вызывается до подключения, данные будут буферизованы и отправлены после установки соединения.

Подробнее см. [Клиент-серверная коммуникация](/guide/api-plugin.html#client-server-communication), включая раздел [Типизация пользовательских событий](/guide/api-plugin.html#typescript-for-custom-events).

## Дополнительное чтение

Если вы хотите узнать больше о том, как использовать API HMR и как он работает под капотом. Проверьте эти ресурсы:

- [Hot Module Replacement is Easy](https://bjornlu.com/blog/hot-module-replacement-is-easy)
