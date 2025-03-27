# Серверный рендеринг (SSR)

:::tip Примечание
SSR конкретно относится к фронтенд-фреймворкам (например, React, Preact, Vue и Svelte), которые поддерживают запуск одного и того же приложения в Node.js, предварительный рендеринг в HTML и, наконец, гидратацию на клиенте. Если вы ищете интеграцию с традиционными серверными фреймворками, вместо этого проверьте [Руководство по интеграции с бэкендом](./backend-integration).

Следующее руководство также предполагает предварительный опыт работы с SSR в вашем выбранном фреймворке и будет фокусироваться только на деталях интеграции, специфичных для Vite.
:::

:::warning API низкого уровня
Это API низкого уровня, предназначенное для авторов библиотек и фреймворков. Если ваша цель - создать приложение, убедитесь, что вы проверили плагины и инструменты SSR высокого уровня в разделе [Awesome Vite SSR](https://github.com/vitejs/awesome-vite#ssr) сначала. Тем не менее, многие приложения успешно построены непосредственно поверх нативного API низкого уровня Vite.

В настоящее время Vite работает над улучшенным API SSR с [API окружения](https://github.com/vitejs/vite/discussions/16358). Проверьте ссылку для получения более подробной информации.
:::

## Примеры проектов

Vite предоставляет встроенную поддержку серверного рендеринга (SSR). [`create-vite-extra`](https://github.com/bluwy/create-vite-extra) содержит примеры настройки SSR, которые вы можете использовать как ссылки для этого руководства:

- [Vanilla](https://github.com/bluwy/create-vite-extra/tree/master/template-ssr-vanilla)
- [Vue](https://github.com/bluwy/create-vite-extra/tree/master/template-ssr-vue)
- [React](https://github.com/bluwy/create-vite-extra/tree/master/template-ssr-react)
- [Preact](https://github.com/bluwy/create-vite-extra/tree/master/template-ssr-preact)
- [Svelte](https://github.com/bluwy/create-vite-extra/tree/master/template-ssr-svelte)
- [Solid](https://github.com/bluwy/create-vite-extra/tree/master/template-ssr-solid)

Вы также можете создать эти проекты локально, [запустив `create-vite`](./index.md#scaffolding-your-first-vite-project) и выбрав `Others > create-vite-extra` под опцией framework.

## Структура исходного кода

Типичное SSR-приложение будет иметь следующую структуру исходных файлов:

```
- index.html
- server.js # основной сервер приложения
- src/
  - main.js          # экспортирует универсальный код приложения
  - entry-client.js  # монтирует приложение в DOM-элемент
  - entry-server.js  # рендерит приложение используя SSR API фреймворка
```

`index.html` должен ссылаться на `entry-client.js` и включать заполнитель, куда должен быть внедрен серверный рендеринг:

```html [index.html]
<div id="app"><!--ssr-outlet--></div>
<script type="module" src="/src/entry-client.js"></script>
```

Вы можете использовать любой заполнитель вместо `<!--ssr-outlet-->`, пока он может быть точно заменен.

## Условная логика

Если вам нужно выполнить условную логику на основе SSR vs. клиента, вы можете использовать

```js twoslash
import 'vite/client'
// ---cut---
if (import.meta.env.SSR) {
  // ... логика только для сервера
}
```

Это статически заменяется во время сборки, поэтому это позволит tree-shaking неиспользуемых веток.

## Настройка сервера разработки

При создании SSR-приложения вы, вероятно, хотите иметь полный контроль над вашим основным сервером и отделить Vite от production окружения. Поэтому рекомендуется использовать Vite в режиме middleware. Вот пример с [express](https://expressjs.com/) (v4):

```js{15-18} twoslash [server.js]
import fs from 'node:fs'
import path from 'node:path'
import { fileURLToPath } from 'node:url'
import express from 'express'
import { createServer as createViteServer } from 'vite'

const __dirname = path.dirname(fileURLToPath(import.meta.url))

async function createServer() {
  const app = express()

  // Создаем сервер Vite в режиме middleware и настраиваем тип приложения как
  // 'custom', отключая собственную логику обслуживания HTML Vite, чтобы родительский сервер
  // мог взять контроль
  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: 'custom'
  })

  // Используем экземпляр connect Vite как middleware. Если вы используете свой собственный
  // express router (express.Router()), вы должны использовать router.use
  // Когда сервер перезапускается (например, после того как пользователь изменяет
  // vite.config.js), `vite.middlewares` все еще будет тем же
  // ссылкой (с новым внутренним стеком Vite и middleware, внедренными плагинами).
  // Следующее действительно даже после перезапусков.
  app.use(vite.middlewares)

  app.use('*', async (req, res) => {
    // обслуживаем index.html - мы займемся этим следующим
  })

  app.listen(5173)
}

createServer()
```

Здесь `vite` - это экземпляр [ViteDevServer](./api-javascript#vitedevserver). `vite.middlewares` - это экземпляр [Connect](https://github.com/senchalabs/connect), который может быть использован как middleware в любом совместимом с connect Node.js фреймворке.

Следующий шаг - реализация обработчика `*` для обслуживания серверного HTML:

```js twoslash [server.js]
// @noErrors
import fs from 'node:fs'
import path from 'node:path'
import { fileURLToPath } from 'node:url'

/** @type {import('express').Express} */
var app
/** @type {import('vite').ViteDevServer}  */
var vite

// ---cut---
app.use('*', async (req, res, next) => {
  const url = req.originalUrl

  try {
    // 1. Читаем index.html
    let template = fs.readFileSync(
      path.resolve(__dirname, 'index.html'),
      'utf-8',
    )

    // 2. Применяем HTML трансформации Vite. Это внедряет клиент HMR Vite,
    //    и также применяет HTML трансформации из плагинов Vite, например, глобальные
    //    преамбулы из @vitejs/plugin-react
    template = await vite.transformIndexHtml(url, template)

    // 3. Загружаем серверную точку входа. ssrLoadModule автоматически трансформирует
    //    ESM исходный код для использования в Node.js! Здесь нет необходимости в бандлинге,
    //    и предоставляется эффективная инвалидация, похожая на HMR.
    const { render } = await vite.ssrLoadModule('/src/entry-server.js')

    // 4. рендерим HTML приложения. Это предполагает, что экспортируемая
    //    функция `render` из entry-server.js вызывает соответствующие SSR API фреймворка,
    //    например ReactDOMServer.renderToString()
    const appHtml = await render(url)

    // 5. Внедряем HTML, отрендеренный приложением, в шаблон.
    const html = template.replace(`<!--ssr-outlet-->`, () => appHtml)

    // 6. Отправляем отрендеренный HTML обратно.
    res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
  } catch (e) {
    // Если поймана ошибка, позволим Vite исправить стек-трейс, чтобы он соответствовал
    // вашему реальному исходному коду.
    vite.ssrFixStacktrace(e)
    next(e)
  }
})
```

Скрипт `dev` в `package.json` также должен быть изменен, чтобы использовать серверный скрипт вместо этого:

```diff [package.json]
  "scripts": {
-   "dev": "vite"
+   "dev": "node server"
  }
```

## Сборка для Production

Чтобы отправить SSR проект в production, нам нужно:

1. Создать клиентскую сборку как обычно;
2. Создать SSR сборку, которая может быть загружена напрямую через `import()`, чтобы нам не нужно было проходить через `ssrLoadModule` Vite;

Наши скрипты в `package.json` будут выглядеть так:

```json [package.json]
{
  "scripts": {
    "dev": "node server",
    "build:client": "vite build --outDir dist/client",
    "build:server": "vite build --outDir dist/server --ssr src/entry-server.js"
  }
}
```

Обратите внимание на флаг `--ssr`, который указывает, что это SSR сборка. Он также должен указывать SSR точку входа.

Затем, в `server.js` нам нужно добавить некоторую логику, специфичную для production, проверяя `process.env.NODE_ENV`:

- Вместо чтения корневого `index.html`, используем `dist/client/index.html` как шаблон, так как он содержит правильные ссылки на ресурсы клиентской сборки.

- Вместо `await vite.ssrLoadModule('/src/entry-server.js')`, используем `import('./dist/server/entry-server.js')` (этот файл является результатом SSR сборки).

- Перемещаем создание и все использование `vite` dev сервера за условные ветки только для dev, затем добавляем middleware для обслуживания статических файлов для обслуживания файлов из `dist/client`.

Обратитесь к [примерам проектов](#example-projects) для рабочей настройки.

## Генерация директив предзагрузки

`vite build` поддерживает флаг `--ssrManifest`, который сгенерирует `.vite/ssr-manifest.json` в директории вывода сборки:

```diff
- "build:client": "vite build --outDir dist/client",
+ "build:client": "vite build --outDir dist/client --ssrManifest",
```

Вышеуказанный скрипт теперь сгенерирует `dist/client/.vite/ssr-manifest.json` для клиентской сборки (Да, SSR манифест генерируется из клиентской сборки, потому что мы хотим сопоставить ID модулей с клиентскими файлами). Манифест содержит сопоставления ID модулей с их связанными чанками и файлами ресурсов.

Чтобы использовать манифест, фреймворки должны предоставить способ сбора ID модулей компонентов, которые использовались во время серверного рендеринга.

`@vitejs/plugin-vue` поддерживает это из коробки и автоматически регистрирует использованные ID модулей компонентов на связанном контексте Vue SSR:

```js [src/entry-server.js]
const ctx = {}
const html = await vueServerRenderer.renderToString(app, ctx)
// ctx.modules теперь Set ID модулей, которые использовались во время рендеринга
```

В production ветке `server.js` нам нужно прочитать и передать манифест в функцию `render`, экспортируемую из `src/entry-server.js`. Это предоставит нам достаточно информации для рендеринга директив предзагрузки для файлов, используемых асинхронными маршрутами! См. [исходный код демо](https://github.com/vitejs/vite-plugin-vue/blob/main/playground/ssr-vue/src/entry-server.js) для полного примера. Вы также можете использовать эту информацию для [103 Early Hints](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103).

## Предварительный рендеринг / SSG

Если маршруты и данные, необходимые для определенных маршрутов, известны заранее, мы можем предварительно отрендерить эти маршруты в статический HTML, используя ту же логику, что и в production SSR. Это также можно рассматривать как форму генерации статического сайта (SSG). См. [скрипт предварительного рендеринга демо](https://github.com/vitejs/vite-plugin-vue/blob/main/playground/ssr-vue/prerender.js) для рабочего примера.

## SSR Externals

Зависимости "экстернализируются" из системы трансформации модулей Vite по умолчанию при запуске SSR. Это ускоряет как dev, так и сборку.

Если зависимость должна быть трансформирована конвейером Vite, например, потому что функции Vite используются нетранспилированными в них, они могут быть добавлены в [`ssr.noExternal`](../config/ssr-options.md#ssr-noexternal).

Для связанных зависимостей они не экстернализируются по умолчанию, чтобы воспользоваться преимуществами HMR Vite. Если это нежелательно, например, для тестирования зависимостей, как если бы они не были связаны, вы можете добавить их в [`ssr.external`](../config/ssr-options.md#ssr-external).

:::warning Работа с алиасами
Если вы настроили алиасы, которые перенаправляют один пакет на другой, вы можете захотеть алиасировать фактические пакеты `node_modules` вместо этого, чтобы это работало для экстернализированных зависимостей SSR. И [Yarn](https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-alias), и [pnpm](https://pnpm.io/aliases/) поддерживают алиасирование через префикс `npm:`.
:::

## Логика плагинов, специфичная для SSR

Некоторые фреймворки, такие как Vue или Svelte, компилируют компоненты в разные форматы на основе клиента vs. SSR. Для поддержки условных трансформаций Vite передает дополнительное свойство `ssr` в объект `options` следующих хуков плагина:

- `resolveId`
- `load`
- `transform`

**Пример:**

```js twoslash
/** @type {() => import('vite').Plugin} */
// ---cut---
export function mySSRPlugin() {
  return {
    name: 'my-ssr',
    transform(code, id, options) {
      if (options?.ssr) {
        // выполнить трансформацию, специфичную для SSR...
      }
    },
  }
}
```

Объект options в `load` и `transform` является опциональным, rollup в настоящее время не использует этот объект, но может расширить эти хуки дополнительными метаданными в будущем.

:::tip Примечание
До Vite 2.7 это сообщалось хукам плагина с позиционным параметром `ssr` вместо использования объекта `options`. Все основные фреймворки и плагины обновлены, но вы можете найти устаревшие посты, использующие предыдущий API.
:::

## Целевая платформа SSR

Целевая платформа по умолчанию для SSR сборки - это окружение node, но вы также можете запустить сервер в Web Worker. Разрешение точек входа пакетов отличается для каждой платформы. Вы можете настроить целевую платформу как Web Worker, используя `ssr.target` установленный в `'webworker'`.

## Бандл SSR

В некоторых случаях, таких как окружения `webworker`, вы можете захотеть собрать вашу SSR сборку в один JavaScript файл. Вы можете включить это поведение, установив `ssr.noExternal` в `true`. Это сделает две вещи:

- Обработает все зависимости как `noExternal`
- Выбросит ошибку, если импортируются какие-либо встроенные модули Node.js

## Условия разрешения SSR

По умолчанию разрешение точек входа пакетов будет использовать условия, установленные в [`resolve.conditions`](../config/shared-options.md#resolve-conditions) для SSR сборки. Вы можете использовать [`ssr.resolve.conditions`](../config/ssr-options.md#ssr-resolve-conditions) и [`ssr.resolve.externalConditions`](../config/ssr-options.md#ssr-resolve-externalconditions) для настройки этого поведения.

## Vite CLI

CLI команды `$ vite dev` и `$ vite preview` также могут использоваться для SSR приложений. Вы можете добавить ваши SSR middleware в сервер разработки с помощью [`configureServer`](/guide/api-plugin#configureserver) и в сервер предпросмотра с помощью [`configurePreviewServer`](/guide/api-plugin#configurepreviewserver).

:::tip Примечание
Используйте post-хук, чтобы ваши SSR middleware выполнялись _после_ middleware Vite.
:::
