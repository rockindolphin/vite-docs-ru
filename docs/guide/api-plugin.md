# API плагинов

Плагины Vite расширяют хорошо спроектированный интерфейс плагинов Rollup несколькими дополнительными опциями, специфичными для Vite. В результате вы можете написать плагин Vite один раз, и он будет работать как для разработки, так и для сборки.

**Рекомендуется сначала ознакомиться с [документацией по плагинам Rollup](https://rollupjs.org/plugin-development/), прежде чем читать следующие разделы.**

## Создание плагина

Vite стремится предлагать установленные паттерны из коробки, поэтому перед созданием нового плагина убедитесь, что вы проверили [Руководство по возможностям](https://vite.dev/guide/features), чтобы увидеть, покрыта ли ваша потребность. Также просмотрите доступные плагины сообщества, как в форме [совместимого плагина Rollup](https://github.com/rollup/awesome), так и [специфичных для Vite плагинов](https://github.com/vitejs/awesome-vite#plugins)

При создании плагина вы можете встроить его в ваш `vite.config.js`. Нет необходимости создавать новый пакет для него. Как только вы увидите, что плагин был полезен в ваших проектах, подумайте о том, чтобы поделиться им, чтобы помочь другим [в экосистеме](https://chat.vite.dev).

::: tip
При изучении, отладке или создании плагинов мы предлагаем включить [vite-plugin-inspect](https://github.com/antfu/vite-plugin-inspect) в ваш проект. Это позволит вам проверять промежуточное состояние плагинов Vite. После установки вы можете посетить `localhost:5173/__inspect/`, чтобы проверить модули и стек трансформации вашего проекта. Проверьте инструкции по установке в [документации vite-plugin-inspect](https://github.com/antfu/vite-plugin-inspect).
![vite-plugin-inspect](/images/vite-plugin-inspect.png)
:::

## Конвенции

Если плагин не использует специфичные для Vite хуки и может быть реализован как [Совместимый плагин Rollup](#rollup-plugin-compatibility), то рекомендуется использовать [Конвенции именования плагинов Rollup](https://rollupjs.org/plugin-development/#conventions).

- Плагины Rollup должны иметь четкое имя с префиксом `rollup-plugin-`.
- Включите ключевые слова `rollup-plugin` и `vite-plugin` в package.json.

Это позволяет использовать плагин также в чистых проектах Rollup или WMR.

Для плагинов только для Vite:

- Плагины Vite должны иметь четкое имя с префиксом `vite-plugin-`.
- Включите ключевое слово `vite-plugin` в package.json.
- Включите раздел в документации плагина, объясняющий, почему это плагин только для Vite (например, он использует специфичные для Vite хуки плагина).

Если ваш плагин будет работать только для определенного фреймворка, его имя должно включаться как часть префикса:

- Префикс `vite-plugin-vue-` для плагинов Vue
- Префикс `vite-plugin-react-` для плагинов React
- Префикс `vite-plugin-svelte-` для плагинов Svelte

См. также [Конвенцию виртуальных модулей](#virtual-modules-convention).

## Конфигурация плагинов

Пользователи будут добавлять плагины в `devDependencies` проекта и настраивать их с помощью опции массива `plugins`.

```js [vite.config.js]
import vitePlugin from 'vite-plugin-feature'
import rollupPlugin from 'rollup-plugin-feature'

export default defineConfig({
  plugins: [vitePlugin(), rollupPlugin()],
})
```

Ложные плагины будут игнорироваться, что можно использовать для легкой активации или деактивации плагинов.

`plugins` также принимает пресеты, включающие несколько плагинов как один элемент. Это полезно для сложных функций (например, интеграции фреймворка), которые реализованы с использованием нескольких плагинов. Массив будет автоматически сглажен внутри.

```js
// framework-plugin
import frameworkRefresh from 'vite-plugin-framework-refresh'
import frameworkDevtools from 'vite-plugin-framework-devtools'

export default function framework(config) {
  return [frameworkRefresh(config), frameworkDevTools(config)]
}
```

```js [vite.config.js]
import { defineConfig } from 'vite'
import framework from 'vite-plugin-framework'

export default defineConfig({
  plugins: [framework()],
})
```

## Простые примеры

:::tip
Общепринято создавать плагин Vite/Rollup как фабричную функцию, которая возвращает фактический объект плагина. Функция может принимать опции, которые позволяют пользователям настраивать поведение плагина.
:::

### Трансформация пользовательских типов файлов

```js
const fileRegex = /\.(my-file-ext)$/

export default function myPlugin() {
  return {
    name: 'transform-file',

    transform(src, id) {
      if (fileRegex.test(id)) {
        return {
          code: compileFileToJS(src),
          map: null, // предоставьте source map, если доступно
        }
      }
    },
  }
}
```

### Импорт виртуального файла

См. пример в [следующем разделе](#virtual-modules-convention).

## Конвенция виртуальных модулей

Виртуальные модули - это полезная схема, которая позволяет вам передавать информацию времени сборки в исходные файлы, используя обычный синтаксис импорта ESM.

```js
export default function myPlugin() {
  const virtualModuleId = 'virtual:my-module'
  const resolvedVirtualModuleId = '\0' + virtualModuleId

  return {
    name: 'my-plugin', // обязательно, будет показано в предупреждениях и ошибках
    resolveId(id) {
      if (id === virtualModuleId) {
        return resolvedVirtualModuleId
      }
    },
    load(id) {
      if (id === resolvedVirtualModuleId) {
        return `export const msg = "from virtual module"`
      }
    },
  }
}
```

Что позволяет импортировать модуль в JavaScript:

```js
import { msg } from 'virtual:my-module'

console.log(msg)
```

Виртуальные модули в Vite (и Rollup) по конвенции имеют префикс `virtual:` для пользовательского пути. Если возможно, имя плагина должно использоваться как пространство имен, чтобы избежать конфликтов с другими плагинами в экосистеме. Например, `vite-plugin-posts` может попросить пользователей импортировать виртуальные модули `virtual:posts` или `virtual:posts/helpers` для получения информации времени сборки. Внутри плагины, использующие виртуальные модули, должны префиксировать ID модуля с `\0` при разрешении id, конвенция из экосистемы rollup. Это предотвращает попытки других плагинов обработать id (например, разрешение node), и основные функции, такие как sourcemaps, могут использовать эту информацию для различения виртуальных модулей и обычных файлов. `\0` не является разрешенным символом в URL импорта, поэтому мы должны заменять их во время анализа импорта. ID `\0{id}` виртуального модуля в итоге кодируется как `/@id/__x00__{id}` во время разработки в браузере. ID будет декодирован обратно перед входом в конвейер плагинов, поэтому это не видно в коде хуков плагинов.

Обратите внимание, что модули, напрямую производные от реального файла, как в случае скриптового модуля в Single File Component (например, .vue или .svelte SFC), не нуждаются в соблюдении этой конвенции. SFC обычно генерируют набор подмодулей при обработке, но код в них может быть сопоставлен с файловой системой. Использование `\0` для этих подмодулей предотвратит правильную работу sourcemaps.

## Универсальные хуки

Во время разработки сервер разработки Vite создает контейнер плагина, который вызывает [Хуки сборки Rollup](https://rollupjs.org/plugin-development/#build-hooks) так же, как это делает Rollup.

Следующие хуки вызываются один раз при запуске сервера:

- [`options`](https://rollupjs.org/plugin-development/#options)
- [`buildStart`](https://rollupjs.org/plugin-development/#buildstart)

Следующие хуки вызываются при каждом входящем запросе модуля:

- [`resolveId`](https://rollupjs.org/plugin-development/#resolveid)
- [`load`](https://rollupjs.org/plugin-development/#load)
- [`transform`](https://rollupjs.org/plugin-development/#transform)

Эти хуки также имеют расширенный параметр `options` с дополнительными свойствами, специфичными для Vite. Вы можете прочитать больше в [документации SSR](/guide/ssr#ssr-specific-plugin-logic).

Некоторые вызовы `resolveId` могут иметь значение `importer` как абсолютный путь для общего `index.html` в корне, так как не всегда возможно вывести фактический импортер из-за паттерна небандлированного сервера разработки Vite. Для импортов, обрабатываемых в конвейере разрешения Vite, импортер может отслеживаться во время фазы анализа импорта, предоставляя правильное значение `importer`.

Следующие хуки вызываются при закрытии сервера:

- [`buildEnd`](https://rollupjs.org/plugin-development/#buildend)
- [`closeBundle`](https://rollupjs.org/plugin-development/#closebundle)

Обратите внимание, что хук [`moduleParsed`](https://rollupjs.org/plugin-development/#moduleparsed) **не** вызывается во время разработки, потому что Vite избегает полного парсинга AST для лучшей производительности.

[Хуки генерации вывода](https://rollupjs.org/plugin-development/#output-generation-hooks) (кроме `closeBundle`) **не** вызываются во время разработки. Вы можете думать о сервере разработки Vite как о вызове только `rollup.rollup()` без вызова `bundle.generate()`.

## Специфичные для Vite хуки

Плагины Vite также могут предоставлять хуки, которые служат специфичным для Vite целям. Эти хуки игнорируются Rollup.

### `config`

- **Тип:** `(config: UserConfig, env: { mode: string, command: string }) => UserConfig | null | void`
- **Вид:** `async`, `sequential`

  Изменить конфигурацию Vite до её разрешения. Хук получает сырую пользовательскую конфигурацию (опции CLI, объединенные с файлом конфигурации) и текущую конфигурацию env, которая раскрывает `mode` и `command`, которые используются. Он может вернуть частичный объект конфигурации, который будет глубоко объединен в существующую конфигурацию, или напрямую мутировать конфигурацию (если стандартное объединение не может достичь желаемого результата).

  **Пример:**

  ```js
  // вернуть частичную конфигурацию (рекомендуется)
  const partialConfigPlugin = () => ({
    name: 'return-partial',
    config: () => ({
      resolve: {
        alias: {
          foo: 'bar',
        },
      },
    }),
  })

  // мутировать конфигурацию напрямую (используйте только когда объединение не работает)
  const mutateConfigPlugin = () => ({
    name: 'mutate-config',
    config(config, { command }) {
      if (command === 'build') {
        config.root = 'foo'
      }
    },
  })
  ```

  ::: warning Примечание
  Пользовательские плагины разрешаются до запуска этого хука, поэтому внедрение других плагинов внутри хука `config` не будет иметь эффекта.
  :::

### `configResolved`

- **Тип:** `(config: ResolvedConfig) => void | Promise<void>`
- **Вид:** `async`, `parallel`

  Вызывается после разрешения конфигурации Vite. Используйте этот хук для чтения и хранения окончательной разрешенной конфигурации. Это также полезно, когда плагину нужно сделать что-то другое в зависимости от выполняемой команды.

  **Пример:**

  ```js
  const examplePlugin = () => {
    let config

    return {
      name: 'read-config',

      configResolved(resolvedConfig) {
        // сохранить разрешенную конфигурацию
        config = resolvedConfig
      },

      // использовать сохраненную конфигурацию в других хуках
      transform(code, id, options) {
        if (config.command === 'serve') {
          // dev: плагин вызывается сервером разработки
        } else {
          // build: плагин вызывается Rollup
        }
      },
    }
  }
  ```

  Обратите внимание, что значение `command` равно `serve` в dev (в CLI `vite`, `vite dev` и `vite serve` являются алиасами).

### `configureServer`

- **Тип:** `(server: ViteDevServer) => (() => void) | void | Promise<(() => void) | void>`
- **Вид:** `async`, `sequential`
- **См. также:** [ViteDevServer](./api-javascript#vitedevserver)

  Хук для настройки сервера разработки. Наиболее распространенный случай использования - добавление пользовательских middleware во внутреннее приложение [connect](https://github.com/senchalabs/connect):

  ```js
  const myPlugin = () => ({
    name: 'configure-server',
    configureServer(server) {
      server.middlewares.use((req, res, next) => {
        // пользовательская обработка запроса...
      })
    },
  })
  ```

  **Внедрение Post Middleware**

  Хук `configureServer` вызывается до установки внутренних middleware, поэтому пользовательские middleware будут выполняться до внутренних middleware по умолчанию. Если вы хотите внедрить middleware **после** внутренних middleware, вы можете вернуть функцию из `configureServer`, которая будет вызвана после установки внутренних middleware:

  ```js
  const myPlugin = () => ({
    name: 'configure-server',
    configureServer(server) {
      // вернуть post-хук, который вызывается после установки
      // внутренних middleware
      return () => {
        server.middlewares.use((req, res, next) => {
          // пользовательская обработка запроса...
        })
      }
    },
  })
  ```

  **Хранение доступа к серверу**

  В некоторых случаях другим хукам плагина может потребоваться доступ к экземпляру сервера разработки (например, доступ к веб-сокет серверу, наблюдателю файловой системы или графу модулей). Этот хук также можно использовать для хранения экземпляра сервера для доступа в других хуках:

  ```js
  const myPlugin = () => {
    let server
    return {
      name: 'configure-server',
      configureServer(_server) {
        server = _server
      },
      transform(code, id) {
        if (server) {
          // использовать сервер...
        }
      },
    }
  }
  ```

  Обратите внимание, что `configureServer` не вызывается при запуске production сборки, поэтому ваши другие хуки должны защищаться от его отсутствия.

### `configurePreviewServer`

- **Тип:** `(server: PreviewServer) => (() => void) | void | Promise<(() => void) | void>`
- **Вид:** `async`, `sequential`
- **См. также:** [PreviewServer](./api-javascript#previewserver)

  То же, что и [`configureServer`](/guide/api-plugin.html#configureserver), но для сервера предпросмотра. Аналогично `configureServer`, хук `configurePreviewServer` вызывается до установки других middleware. Если вы хотите внедрить middleware **после** других middleware, вы можете вернуть функцию из `configurePreviewServer`, которая будет вызвана после установки внутренних middleware:

  ```js
  const myPlugin = () => ({
    name: 'configure-preview-server',
    configurePreviewServer(server) {
      // вернуть post-хук, который вызывается после установки
      // других middleware
      return () => {
        server.middlewares.use((req, res, next) => {
          // пользовательская обработка запроса...
        })
      }
    },
  })
  ```

### `transformIndexHtml`

- **Тип:** `IndexHtmlTransformHook | { order?: 'pre' | 'post', handler: IndexHtmlTransformHook }`
- **Вид:** `async`, `sequential`

  Специальный хук для трансформации HTML-файлов точки входа, таких как `index.html`. Хук получает текущую HTML-строку и контекст трансформации. Контекст раскрывает экземпляр [`ViteDevServer`](./api-javascript#vitedevserver) во время разработки и раскрывает бандл вывода Rollup во время сборки.

  Хук может быть асинхронным и может вернуть одно из следующего:

  - Трансформированную HTML-строку
  - Массив объектов дескрипторов тегов (`{ tag, attrs, children }`) для внедрения в существующий HTML. Каждый тег также может указать, куда он должен быть внедрен (по умолчанию - в начало `<head>`)
  - Объект, содержащий оба как `{ html, tags }`

  По умолчанию `order` равен `undefined`, при этом этот хук применяется после трансформации HTML. Чтобы внедрить скрипт, который должен пройти через конвейер плагинов Vite, `order: 'pre'` применит хук до обработки HTML. `order: 'post'` применяет хук после всех хуков с `order` undefined.

  **Базовый пример:**

  ```js
  const htmlPlugin = () => {
    return {
      name: 'html-transform',
      transformIndexHtml(html) {
        return html.replace(
          /<title>(.*?)<\/title>/,
          `<title>Title replaced!</title>`,
        )
      },
    }
  }
  ```

  **Полная сигнатура хука:**

  ```ts
  type IndexHtmlTransformHook = (
    html: string,
    ctx: {
      path: string
      filename: string
      server?: ViteDevServer
      bundle?: import('rollup').OutputBundle
      chunk?: import('rollup').OutputChunk
    },
  ) =>
    | IndexHtmlTransformResult
    | void
    | Promise<IndexHtmlTransformResult | void>

  type IndexHtmlTransformResult =
    | string
    | HtmlTagDescriptor[]
    | {
        html: string
        tags: HtmlTagDescriptor[]
      }

  interface HtmlTagDescriptor {
    tag: string
    attrs?: Record<string, string | boolean>
    children?: string | HtmlTagDescriptor[]
    /**
     * по умолчанию: 'head-prepend'
     */
    injectTo?: 'head' | 'body' | 'head-prepend' | 'body-prepend'
  }
  ```

  ::: warning Примечание
  Этот хук не будет вызван, если вы используете фреймворк, который имеет пользовательскую обработку файлов входа (например, [SvelteKit](https://github.com/sveltejs/kit/discussions/8269#discussioncomment-4509145)).
  :::

### `handleHotUpdate`

- **Тип:** `(ctx: HmrContext) => Array<ModuleNode> | void | Promise<Array<ModuleNode> | void>`
- **См. также:** [HMR API](./api-hmr)

  Выполнить пользовательскую обработку обновления HMR. Хук получает объект контекста со следующей сигнатурой:

  ```ts
  interface HmrContext {
    file: string
    timestamp: number
    modules: Array<ModuleNode>
    read: () => string | Promise<string>
    server: ViteDevServer
  }
  ```

  - `modules` - это массив модулей, на которые влияет измененный файл. Это массив, потому что один файл может соответствовать нескольким обслуживаемым модулям (например, Vue SFC).

  - `read` - это асинхронная функция чтения, которая возвращает содержимое файла. Это предоставляется, потому что на некоторых системах callback изменения файла может сработать слишком быстро, прежде чем редактор закончит обновление файла, и прямой `fs.readFile` вернет пустое содержимое. Функция чтения, переданная внутрь, нормализует это поведение.

  Хук может выбрать:

  - Отфильтровать и сузить список затронутых модулей, чтобы HMR был более точным.

  - Вернуть пустой массив и выполнить полную перезагрузку:

    ```js
    handleHotUpdate({ server, modules, timestamp }) {
      // Инвалидировать модули вручную
      const invalidatedModules = new Set()
      for (const mod of modules) {
        server.moduleGraph.invalidateModule(
          mod,
          invalidatedModules,
          timestamp,
          true
        )
      }
      server.ws.send({ type: 'full-reload' })
      return []
    }
    ```

  - Вернуть пустой массив и выполнить полностью пользовательскую обработку HMR, отправляя пользовательские события клиенту:

    ```js
    handleHotUpdate({ server }) {
      server.ws.send({
        type: 'custom',
        event: 'special-update',
        data: {}
      })
      return []
    }
    ```

    Клиентский код должен зарегистрировать соответствующий обработчик, используя [HMR API](./api-hmr) (это может быть внедрено тем же плагином через хук `transform`):

    ```js
    if (import.meta.hot) {
      import.meta.hot.on('special-update', (data) => {
        // выполнить пользовательское обновление
      })
    }
    ```

## Порядок плагинов

Плагин Vite может дополнительно указать свойство `enforce` (аналогично webpack loaders) для настройки порядка его применения. Значение `enforce` может быть либо `"pre"`, либо `"post"`. Разрешенные плагины будут в следующем порядке:

- Alias
- Пользовательские плагины с `enforce: 'pre'`
- Основные плагины Vite
- Пользовательские плагины без значения enforce
- Плагины сборки Vite
- Пользовательские плагины с `enforce: 'post'`
- Пост-плагины сборки Vite (минификация, манифест, отчеты)

Обратите внимание, что это отдельно от порядка хуков, те все еще подчиняются своему атрибуту `order` [как обычно для хуков Rollup](https://rollupjs.org/plugin-development/#build-hooks).

## Условное применение

По умолчанию плагины вызываются как для serve, так и для build. В случаях, когда плагин должен быть условно применен только во время serve или build, используйте свойство `apply`, чтобы вызывать их только во время `'build'` или `'serve'`:

```js
function myPlugin() {
  return {
    name: 'build-only',
    apply: 'build', // или 'serve'
  }
}
```

Также можно использовать функцию для более точного контроля:

```js
apply(config, { command }) {
  // применить только при сборке, но не для SSR
  return command === 'build' && !config.build.ssr
}
```

## Совместимость с плагинами Rollup

Довольно большое количество плагинов Rollup будет работать напрямую как плагин Vite (например, `@rollup/plugin-alias` или `@rollup/plugin-json`), но не все из них, так как некоторые хуки плагинов не имеют смысла в контексте небандлированного сервера разработки.

В общем, пока плагин Rollup соответствует следующим критериям, он должен просто работать как плагин Vite:

- Он не использует хук [`moduleParsed`](https://rollupjs.org/plugin-development/#moduleparsed).
- У него нет сильной связи между хуками фазы бандла и хуками фазы вывода.

Если плагин Rollup имеет смысл только для фазы сборки, то его можно указать под `build.rollupOptions.plugins` вместо этого. Он будет работать так же, как плагин Vite с `enforce: 'post'` и `apply: 'build'`.

Вы также можете расширить существующий плагин Rollup свойствами, специфичными для Vite:

```js [vite.config.js]
import example from 'rollup-plugin-example'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [
    {
      ...example(),
      enforce: 'post',
      apply: 'build',
    },
  ],
})
```

## Нормализация путей

Vite нормализует пути при разрешении id, чтобы использовать разделители POSIX ( / ), сохраняя при этом том в Windows. С другой стороны, Rollup по умолчанию оставляет разрешенные пути нетронутыми, поэтому разрешенные id имеют разделители win32 ( \\ ) в Windows. Однако плагины Rollup используют утилиту [`normalizePath`](https://github.com/rollup/plugins/tree/master/packages/pluginutils#normalizepath) из `@rollup/pluginutils` внутри, которая преобразует разделители в POSIX перед выполнением сравнений. Это означает, что когда эти плагины используются в Vite, паттерн `include` и `exclude` конфигурации и другие подобные сравнения путей с разрешенными id работают правильно.

Итак, для плагинов Vite при сравнении путей с разрешенными id важно сначала нормализовать пути, чтобы использовать разделители POSIX. Эквивалентная утилита `normalizePath` экспортируется из модуля `vite`.

```js
import { normalizePath } from 'vite'

normalizePath('foo\\bar') // 'foo/bar'
normalizePath('foo/bar') // 'foo/bar'
```

## Фильтрация, паттерны include/exclude

Vite раскрывает функцию [`createFilter` из `@rollup/pluginutils`](https://github.com/rollup/plugins/tree/master/packages/pluginutils#createfilter), чтобы поощрить специфичные для Vite плагины и интеграции использовать стандартный паттерн фильтрации include/exclude, который также используется в самом ядре Vite.

## Общение клиент-сервер

Начиная с Vite 2.9, мы предоставляем некоторые утилиты для плагинов, чтобы помочь с общением с клиентами.

### Сервер клиенту

На стороне плагина мы можем использовать `server.ws.send` для широковещательной рассылки событий клиенту:

```js [vite.config.js]
export default defineConfig({
  plugins: [
    {
      // ...
      configureServer(server) {
        server.ws.on('connection', () => {
          server.ws.send('my:greetings', { msg: 'hello' })
        })
      },
    },
  ],
})
```

::: tip ПРИМЕЧАНИЕ
Мы рекомендуем **всегда префиксировать** ваши имена событий, чтобы избежать конфликтов с другими плагинами.
:::

На стороне клиента используйте [`hot.on`](/guide/api-hmr.html#hot-on-event-cb) для прослушивания событий:

```ts twoslash
import 'vite/client'
// ---cut---
// сторона клиента
if (import.meta.hot) {
  import.meta.hot.on('my:greetings', (data) => {
    console.log(data.msg) // hello
  })
}
```

### Клиент серверу

Чтобы отправлять события от клиента к серверу, мы можем использовать [`hot.send`](/guide/api-hmr.html#hot-send-event-payload):

```ts
// сторона клиента
if (import.meta.hot) {
  import.meta.hot.send('my:from-client', { msg: 'Hey!' })
}
```

Затем используйте `server.ws.on` и слушайте события на стороне сервера:

```js [vite.config.js]
export default defineConfig({
  plugins: [
    {
      // ...
      configureServer(server) {
        server.ws.on('my:from-client', (data, client) => {
          console.log('Сообщение от клиента:', data.msg) // Hey!
          // ответить только клиенту (если нужно)
          client.send('my:ack', { msg: 'Привет! Я получил ваше сообщение!' })
        })
      },
    },
  ],
})
```

### TypeScript для пользовательских событий

Внутри Vite выводит тип полезной нагрузки из интерфейса `CustomEventMap`, возможно типизировать пользовательские события, расширяя интерфейс:

:::tip Примечание
Убедитесь, что вы включаете расширение `.d.ts` при указании файлов объявлений TypeScript. В противном случае TypeScript может не знать, какой файл пытается расширить модуль.
:::

```ts [events.d.ts]
import 'vite/types/customEvent.d.ts'

declare module 'vite/types/customEvent.d.ts' {
  interface CustomEventMap {
    'custom:foo': { msg: string }
    // 'ключ-события': полезная-нагрузка
  }
}
```

Это расширение интерфейса используется `InferCustomEventPayload<T>` для вывода типа полезной нагрузки для события `T`. Для получения дополнительной информации о том, как используется этот интерфейс, обратитесь к [Документации HMR API](./api-hmr#hmr-api).

```ts twoslash
import 'vite/client'
import type { InferCustomEventPayload } from 'vite/types/customEvent.d.ts'
declare module 'vite/types/customEvent.d.ts' {
  interface CustomEventMap {
    'custom:foo': { msg: string }
  }
}
// ---cut---
type CustomFooPayload = InferCustomEventPayload<'custom:foo'>
import.meta.hot?.on('custom:foo', (payload) => {
  // Тип payload будет { msg: string }
})
import.meta.hot?.on('unknown:event', (payload) => {
  // Тип payload будет any
})
```
