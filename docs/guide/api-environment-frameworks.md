# API окружений для фреймворков

:::warning Экспериментально
API окружений является экспериментальным. Мы будем поддерживать API стабильным во время Vite 6, чтобы позволить экосистеме экспериментировать и строить на его основе. Мы планируем стабилизировать эти новые API с потенциальными breaking changes в Vite 7.

Ресурсы:

- [Обсуждение обратной связи](https://github.com/vitejs/vite/discussions/16358), где мы собираем отзывы о новых API.
- [PR API окружений](https://github.com/vitejs/vite/pull/16471), где новые API были реализованы и проверены.

Пожалуйста, поделитесь с нами вашими отзывами.
:::

## Окружения и фреймворки

Неявное окружение `ssr` и другие не-клиентские окружения по умолчанию используют `RunnableDevEnvironment` во время разработки. Хотя это требует, чтобы среда выполнения была такой же, как та, в которой запущен сервер Vite, это работает аналогично с `ssrLoadModule` и позволяет фреймворкам мигрировать и включать HMR для их SSR dev-сценария. Вы можете защитить любое выполнимое окружение с помощью функции `isRunnableDevEnvironment`.

```ts
export class RunnableDevEnvironment extends DevEnvironment {
  public readonly runner: ModuleRunner
}

class ModuleRunner {
  /**
   * URL для выполнения.
   * Принимает путь к файлу, путь сервера или id относительно корня.
   * Возвращает инстанцированный модуль (то же, что и в ssrLoadModule)
   */
  public async import(url: string): Promise<Record<string, any>>
  /**
   * Другие методы ModuleRunner...
   */
}

if (isRunnableDevEnvironment(server.environments.ssr)) {
  await server.environments.ssr.runner.import('/entry-point.js')
}
```

:::warning
`runner` оценивается сразу при первом обращении к нему. Обратите внимание, что Vite включает поддержку source map, когда `runner` создается путем вызова `process.setSourceMapsEnabled` или переопределения `Error.prepareStackTrace`, если это недоступно.
:::

## По умолчанию `RunnableDevEnvironment`

Учитывая сервер Vite, настроенный в режиме middleware, как описано в [руководстве по настройке SSR](/guide/ssr#setting-up-the-dev-server), давайте реализуем SSR middleware с использованием API окружений. Обработка ошибок опущена.

```js
import fs from 'node:fs'
import path from 'node:path'
import { fileURLToPath } from 'node:url'
import { createServer } from 'vite'

const __dirname = path.dirname(fileURLToPath(import.meta.url))

const server = await createServer({
  server: { middlewareMode: true },
  appType: 'custom',
  environments: {
    server: {
      // по умолчанию модули выполняются в том же процессе, что и сервер vite
    },
  },
})

// В TypeScript вам может потребоваться привести это к RunnableDevEnvironment или
// использовать isRunnableDevEnvironment для защиты доступа к runner
const environment = server.environments.node

app.use('*', async (req, res, next) => {
  const url = req.originalUrl

  // 1. Читаем index.html
  const indexHtmlPath = path.resolve(__dirname, 'index.html')
  let template = fs.readFileSync(indexHtmlPath, 'utf-8')

  // 2. Применяем HTML-трансформации Vite. Это внедряет клиент HMR Vite,
  //    а также применяет HTML-трансформации из плагинов Vite, например, глобальные
  //    преамбулы из @vitejs/plugin-react
  template = await server.transformIndexHtml(url, template)

  // 3. Загружаем точку входа сервера. import(url) автоматически преобразует
  //    исходный код ESM для использования в Node.js! Нет необходимости в сборке,
  //    и предоставляется полная поддержка HMR.
  const { render } = await environment.runner.import('/src/entry-server.js')

  // 4. Рендерим HTML приложения. Это предполагает, что экспортируемая
  //    функция `render` из entry-server.js вызывает соответствующие API SSR фреймворка,
  //    например, ReactDOMServer.renderToString()
  const appHtml = await render(url)

  // 5. Внедряем HTML, отрендеренный приложением, в шаблон.
  const html = template.replace(`<!--ssr-outlet-->`, appHtml)

  // 6. Отправляем отрендеренный HTML обратно.
  res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
})
```

## SSR, независимый от среды выполнения

Поскольку `RunnableDevEnvironment` может использоваться только для запуска кода в той же среде выполнения, что и сервер Vite, он требует среду выполнения, которая может запустить сервер Vite (среду выполнения, совместимую с Node.js). Это означает, что вам нужно будет использовать чистый `DevEnvironment`, чтобы сделать его независимым от среды выполнения.

:::info Предложение `FetchableDevEnvironment`

В первоначальном предложении был метод `run` в классе `DevEnvironment`, который позволил бы потребителям вызывать импорт на стороне runner с помощью опции `transport`. Во время нашего тестирования мы обнаружили, что API не был достаточно универсальным, чтобы начать его рекомендовать. В данный момент мы ищем отзывы о [предложении `FetchableDevEnvironment`](https://github.com/vitejs/vite/discussions/18191).

:::

`RunnableDevEnvironment` имеет функцию `runner.import`, которая возвращает значение модуля. Но эта функция недоступна в чистом `DevEnvironment` и требует, чтобы код, использующий API Vite, и пользовательские модули были разделены.

Например, следующий пример использует значение пользовательского модуля из кода, использующего API Vite:

```ts
// код, использующий API Vite
import { createServer } from 'vite'

const server = createServer()
const ssrEnvironment = server.environment.ssr
const input = {}

const { createHandler } = await ssrEnvironment.runner.import('./entry.js')
const handler = createHandler(input)
const response = handler(new Request('/'))

// -------------------------------------
// ./entrypoint.js
export function createHandler(input) {
  return function handler(req) {
    return new Response('hello')
  }
}
```

Если ваш код может выполняться в той же среде выполнения, что и пользовательские модули (то есть, он не полагается на API, специфичные для Node.js), вы можете использовать виртуальный модуль. Этот подход устраняет необходимость доступа к значению из кода, использующего API Vite.

```ts
// код, использующий API Vite
import { createServer } from 'vite'

const server = createServer({
  plugins: [
    // плагин, который обрабатывает `virtual:entrypoint`
    {
      name: 'virtual-module',
      /* реализация плагина */
    },
  ],
})
const ssrEnvironment = server.environment.ssr
const input = {}

// используем раскрытые функции каждой фабрики окружений, которая запускает код
// проверяем, что предоставляет каждая фабрика окружений
if (ssrEnvironment instanceof RunnableDevEnvironment) {
  ssrEnvironment.runner.import('virtual:entrypoint')
} else if (ssrEnvironment instanceof CustomDevEnvironment) {
  ssrEnvironment.runEntrypoint('virtual:entrypoint')
} else {
  throw new Error(`Неподдерживаемая среда выполнения для ${ssrEnvironment.name}`)
}

// -------------------------------------
// virtual:entrypoint
const { createHandler } = await import('./entrypoint.js')
const handler = createHandler(input)
const response = handler(new Request('/'))

// -------------------------------------
// ./entrypoint.js
export function createHandler(input) {
  return function handler(req) {
    return new Response('hello')
  }
}
```

Например, для вызова `transformIndexHtml` на пользовательском модуле можно использовать следующий плагин:

```ts {13-21}
function vitePluginVirtualIndexHtml(): Plugin {
  let server: ViteDevServer | undefined
  return {
    name: vitePluginVirtualIndexHtml.name,
    configureServer(server_) {
      server = server_
    },
    resolveId(source) {
      return source === 'virtual:index-html' ? '\0' + source : undefined
    },
    async load(id) {
      if (id === '\0' + 'virtual:index-html') {
        let html: string
        if (server) {
          this.addWatchFile('index.html')
          html = fs.readFileSync('index.html', 'utf-8')
          html = await server.transformIndexHtml('/', html)
        } else {
          html = fs.readFileSync('dist/client/index.html', 'utf-8')
        }
        return `export default ${JSON.stringify(html)}`
      }
      return
    },
  }
}
```

Если ваш код требует API Node.js, вы можете использовать `hot.send` для связи с кодом, использующим API Vite, из пользовательских модулей. Однако имейте в виду, что этот подход может работать не так же после процесса сборки.

```ts
// код, использующий API Vite
import { createServer } from 'vite'

const server = createServer({
  plugins: [
    // плагин, который обрабатывает `virtual:entrypoint`
    {
      name: 'virtual-module',
      /* реализация плагина */
    },
  ],
})
const ssrEnvironment = server.environment.ssr
const input = {}

// используем раскрытые функции каждой фабрики окружений, которая запускает код
// проверяем, что предоставляет каждая фабрика окружений
if (ssrEnvironment instanceof RunnableDevEnvironment) {
  ssrEnvironment.runner.import('virtual:entrypoint')
} else if (ssrEnvironment instanceof CustomDevEnvironment) {
  ssrEnvironment.runEntrypoint('virtual:entrypoint')
} else {
  throw new Error(`Неподдерживаемая среда выполнения для ${ssrEnvironment.name}`)
}

const req = new Request('/')

const uniqueId = 'a-unique-id'
ssrEnvironment.send('request', serialize({ req, uniqueId }))
const response = await new Promise((resolve) => {
  ssrEnvironment.on('response', (data) => {
    data = deserialize(data)
    if (data.uniqueId === uniqueId) {
      resolve(data.res)
    }
  })
})

// -------------------------------------
// virtual:entrypoint
const { createHandler } = await import('./entrypoint.js')
const handler = createHandler(input)

import.meta.hot.on('request', (data) => {
  const { req, uniqueId } = deserialize(data)
  const res = handler(req)
  import.meta.hot.send('response', serialize({ res: res, uniqueId }))
})

const response = handler(new Request('/'))

// -------------------------------------
// ./entrypoint.js
export function createHandler(input) {
  return function handler(req) {
    return new Response('hello')
  }
}
```

## Окружения во время сборки

В CLI вызовы `vite build` и `vite build --ssr` по-прежнему будут собирать только клиентское и только SSR окружения для обратной совместимости.

Когда `builder` не является `undefined` (или при вызове `vite build --app`), `vite build` будет собирать всё приложение целиком. В будущем это станет значением по умолчанию в мажорной версии. Будет создан экземпляр `ViteBuilder` (эквивалент `ViteDevServer` во время сборки) для сборки всех настроенных окружений для продакшена. По умолчанию сборка окружений выполняется последовательно с учетом порядка записи `environments`. Фреймворк или пользователь может дополнительно настроить способ сборки окружений:

```js
export default {
  builder: {
    buildApp: async (builder) => {
      const environments = Object.values(builder.environments)
      return Promise.all(
        environments.map((environment) => builder.build(environment)),
      )
    },
  },
}
```

## Код, независимый от окружения

В большинстве случаев текущий экземпляр `environment` будет доступен как часть контекста выполняемого кода, поэтому необходимость доступа к ним через `server.environments` должна быть редкой. Например, внутри хуков плагина окружение доступно как часть `PluginContext`, поэтому к нему можно получить доступ с помощью `this.environment`. Смотрите [API окружений для плагинов](./api-environment-plugins.md), чтобы узнать, как создавать плагины, учитывающие окружение.
