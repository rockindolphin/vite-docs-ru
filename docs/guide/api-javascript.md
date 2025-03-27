# JavaScript API

JavaScript API Vite полностью типизированы, и рекомендуется использовать TypeScript или включить проверку типов JS в VS Code для использования автодополнения и валидации.

## `createServer`

**Сигнатура типа:**

```ts
async function createServer(inlineConfig?: InlineConfig): Promise<ViteDevServer>
```

**Пример использования:**

```ts twoslash
import { fileURLToPath } from 'node:url'
import { createServer } from 'vite'

const __dirname = fileURLToPath(new URL('.', import.meta.url))

const server = await createServer({
  // любые допустимые опции пользовательской конфигурации, плюс `mode` и `configFile`
  configFile: false,
  root: __dirname,
  server: {
    port: 1337,
  },
})
await server.listen()

server.printUrls()
server.bindCLIShortcuts({ print: true })
```

::: tip ПРИМЕЧАНИЕ
При использовании `createServer` и `build` в одном процессе Node.js обе функции полагаются на `process.env.NODE_ENV` для правильной работы, что также зависит от опции конфигурации `mode`. Чтобы предотвратить конфликтующее поведение, установите `process.env.NODE_ENV` или `mode` для двух API в `development`. В противном случае вы можете создать дочерний процесс для отдельного запуска API.
:::

::: tip ПРИМЕЧАНИЕ
При использовании [режима middleware](/config/server-options.html#server-middlewaremode) в сочетании с [конфигурацией прокси для WebSocket](/config/server-options.html#server-proxy), родительский http-сервер должен быть предоставлен в `middlewareMode` для правильной привязки прокси.

<details>
<summary>Пример</summary>

```ts twoslash
import http from 'http'
import { createServer } from 'vite'

const parentServer = http.createServer() // или express, koa и т.д.

const vite = await createServer({
  server: {
    // Включить режим middleware
    middlewareMode: {
      // Предоставить родительский http-сервер для прокси WebSocket
      server: parentServer,
    },
    proxy: {
      '/ws': {
        target: 'ws://localhost:3000',
        // Проксирование WebSocket
        ws: true,
      },
    },
  },
})

// @noErrors: 2339
parentServer.use(vite.middlewares)
```

</details>
:::

## `InlineConfig`

Интерфейс `InlineConfig` расширяет `UserConfig` дополнительными свойствами:

- `configFile`: указать файл конфигурации для использования. Если не установлен, Vite попытается автоматически разрешить его из корня проекта. Установите `false`, чтобы отключить автоматическое разрешение.
- `envFile`: Установите `false`, чтобы отключить файлы `.env`.

## `ResolvedConfig`

Интерфейс `ResolvedConfig` имеет все те же свойства, что и `UserConfig`, за исключением того, что большинство свойств разрешены и не равны undefined. Он также содержит утилиты, такие как:

- `config.assetsInclude`: Функция для проверки, считается ли `id` ресурсом.
- `config.logger`: Внутренний объект логгера Vite.

## `ViteDevServer`

```ts
interface ViteDevServer {
  /**
   * Разрешенный объект конфигурации Vite.
   */
  config: ResolvedConfig
  /**
   * Экземпляр приложения connect
   * - Может использоваться для подключения пользовательских middleware к серверу разработки.
   * - Также может использоваться как функция-обработчик пользовательского http-сервера
   *   или как middleware в любых фреймворках Node.js в стиле connect.
   *
   * https://github.com/senchalabs/connect#use-middleware
   */
  middlewares: Connect.Server
  /**
   * Нативный экземпляр http-сервера Node.
   * Будет null в режиме middleware.
   */
  httpServer: http.Server | null
  /**
   * Экземпляр наблюдателя Chokidar. Если `config.server.watch` установлен в `null`,
   * он не будет следить за файлами, и вызовы `add` или `unwatch` не будут иметь эффекта.
   * https://github.com/paulmillr/chokidar/tree/3.6.0#api
   */
  watcher: FSWatcher
  /**
   * WebSocket сервер с методом `send(payload)`.
   */
  ws: WebSocketServer
  /**
   * Контейнер плагинов Rollup, который может запускать хуки плагинов для заданного файла.
   */
  pluginContainer: PluginContainer
  /**
   * Граф модулей, который отслеживает отношения импорта, сопоставление URL с файлами
   * и состояние HMR.
   */
  moduleGraph: ModuleGraph
  /**
   * Разрешенные URL, которые Vite выводит в CLI (URL-кодированные). Возвращает `null`
   * в режиме middleware или если сервер не слушает никакой порт.
   */
  resolvedUrls: ResolvedServerUrls | null
  /**
   * Программно разрешить, загрузить и преобразовать URL и получить результат
   * без прохождения через конвейер http-запросов.
   */
  transformRequest(
    url: string,
    options?: TransformOptions,
  ): Promise<TransformResult | null>
  /**
   * Применить встроенные HTML-трансформации Vite и любые HTML-трансформации плагинов.
   */
  transformIndexHtml(
    url: string,
    html: string,
    originalUrl?: string,
  ): Promise<string>
  /**
   * Загрузить заданный URL как инстанцированный модуль для SSR.
   */
  ssrLoadModule(
    url: string,
    options?: { fixStacktrace?: boolean },
  ): Promise<Record<string, any>>
  /**
   * Исправить стек ошибки SSR.
   */
  ssrFixStacktrace(e: Error): void
  /**
   * Вызывает HMR для модуля в графе модулей. Вы можете использовать API `server.moduleGraph`
   * для получения модуля для перезагрузки. Если `hmr` равен false, это пустая операция.
   */
  reloadModule(module: ModuleNode): Promise<void>
  /**
   * Запустить сервер.
   */
  listen(port?: number, isRestart?: boolean): Promise<ViteDevServer>
  /**
   * Перезапустить сервер.
   *
   * @param forceOptimize - принудительно пересобрать оптимизатор, то же самое, что и флаг --force в CLI
   */
  restart(forceOptimize?: boolean): Promise<void>
  /**
   * Остановить сервер.
   */
  close(): Promise<void>
  /**
   * Привязать сочетания клавиш CLI
   */
  bindCLIShortcuts(options?: BindCLIShortcutsOptions<ViteDevServer>): void
  /**
   * Вызов `await server.waitForRequestsIdle(id)` будет ждать, пока все статические импорты
   * не будут обработаны. Если вызывается из хука загрузки или трансформации плагина, id нужно
   * передавать как параметр, чтобы избежать взаимной блокировки. Вызов этой функции после
   * обработки первого раздела статических импортов графа модулей разрешится немедленно.
   * @experimental
   */
  waitForRequestsIdle: (ignoredId?: string) => Promise<void>
}
```

:::info
`waitForRequestsIdle` предназначен для использования как аварийный выход для улучшения DX для функций, которые нельзя реализовать, следуя принципу "по требованию" сервера разработки Vite. Он может использоваться во время запуска такими инструментами, как Tailwind, чтобы отложить генерацию CSS-классов приложения до тех пор, пока не будет просмотрен код приложения, избегая мерцания изменений стилей. Когда эта функция используется в хуке загрузки или трансформации, и используется сервер HTTP1 по умолчанию, один из шести http-каналов будет заблокирован до тех пор, пока сервер не обработает все статические импорты. Оптимизатор зависимостей Vite в настоящее время использует эту функцию, чтобы избежать полной перезагрузки страницы при отсутствующих зависимостях, откладывая загрузку предварительно собранных зависимостей до тех пор, пока все импортированные зависимости не будут собраны из статических импортированных источников. Vite может переключиться на другую стратегию в будущем мажорном релизе, установив `optimizeDeps.crawlUntilStaticImports: false` по умолчанию, чтобы избежать падения производительности в больших приложениях во время холодного запуска.
:::

## `build`

**Сигнатура типа:**

```ts
async function build(
  inlineConfig?: InlineConfig,
): Promise<RollupOutput | RollupOutput[]>
```

**Пример использования:**

```ts twoslash [vite.config.js]
import path from 'node:path'
import { fileURLToPath } from 'node:url'
import { build } from 'vite'

const __dirname = fileURLToPath(new URL('.', import.meta.url))

await build({
  root: path.resolve(__dirname, './project'),
  base: '/foo/',
  build: {
    rollupOptions: {
      // ...
    },
  },
})
```

## `preview`

**Сигнатура типа:**

```ts
async function preview(inlineConfig?: InlineConfig): Promise<PreviewServer>
```

**Пример использования:**

```ts twoslash
import { preview } from 'vite'

const previewServer = await preview({
  // любые допустимые опции пользовательской конфигурации, плюс `mode` и `configFile`
  preview: {
    port: 8080,
    open: true,
  },
})

previewServer.printUrls()
previewServer.bindCLIShortcuts({ print: true })
```

## `PreviewServer`

```ts
interface PreviewServer {
  /**
   * Разрешенный объект конфигурации Vite
   */
  config: ResolvedConfig
  /**
   * Экземпляр приложения connect
   * - Может использоваться для подключения пользовательских middleware к серверу предварительного просмотра
   * - Также может использоваться как функция-обработчик пользовательского http-сервера
   *   или как middleware в любых фреймворках Node.js в стиле connect
   *
   * https://github.com/senchalabs/connect#use-middleware
   */
  middlewares: Connect.Server
  /**
   * Нативный экземпляр http-сервера Node
   */
  httpServer: http.Server
  /**
   * Разрешенные URL, которые Vite выводит в CLI (URL-кодированные). Возвращает `null`,
   * если сервер не слушает никакой порт.
   */
  resolvedUrls: ResolvedServerUrls | null
  /**
   * Вывести URL сервера
   */
  printUrls(): void
  /**
   * Привязать сочетания клавиш CLI
   */
  bindCLIShortcuts(options?: BindCLIShortcutsOptions<PreviewServer>): void
}
```

## `resolveConfig`

**Сигнатура типа:**

```ts
async function resolveConfig(
  inlineConfig: InlineConfig,
  command: 'build' | 'serve',
  defaultMode = 'development',
  defaultNodeEnv = 'development',
  isPreview = false,
): Promise<ResolvedConfig>
```

Значение `command` равно `serve` в режимах dev и preview, и `build` в режиме сборки.

## `mergeConfig`

**Сигнатура типа:**

```ts
function mergeConfig(
  defaults: Record<string, any>,
  overrides: Record<string, any>,
  isRoot = true,
): Record<string, any>
```

Глубоко объединяет две конфигурации Vite. `isRoot` представляет уровень внутри конфигурации Vite, который объединяется. Например, установите `false`, если вы объединяете две опции `build`.

::: tip ПРИМЕЧАНИЕ
`mergeConfig` принимает только конфигурацию в форме объекта. Если у вас есть конфигурация в форме обратного вызова, вы должны вызвать его перед передачей в `mergeConfig`.

Вы можете использовать помощник `defineConfig` для объединения конфигурации в форме обратного вызова с другой конфигурацией:

```ts twoslash
import {
  defineConfig,
  mergeConfig,
  type UserConfigFnObject,
  type UserConfig,
} from 'vite'
declare const configAsCallback: UserConfigFnObject
declare const configAsObject: UserConfig

// ---cut---
export default defineConfig((configEnv) =>
  mergeConfig(configAsCallback(configEnv), configAsObject),
)
```

:::

## `searchForWorkspaceRoot`

**Сигнатура типа:**

```ts
function searchForWorkspaceRoot(
  current: string,
  root = searchForPackageRoot(current),
): string
```

**Связано:** [server.fs.allow](/config/server-options.md#server-fs-allow)

Поиск корня потенциального рабочего пространства, если оно соответствует следующим условиям, в противном случае вернется к `root`:

- содержит поле `workspaces` в `package.json`
- содержит один из следующих файлов
  - `lerna.json`
  - `pnpm-workspace.yaml`

## `loadEnv`

**Сигнатура типа:**

```ts
function loadEnv(
  mode: string,
  envDir: string,
  prefixes: string | string[] = 'VITE_',
): Record<string, string>
```

**Связано:** [`.env` Файлы](./env-and-mode.md#env-files)

Загружает файлы `.env` в пределах `envDir`. По умолчанию загружаются только переменные окружения с префиксом `VITE_`, если `prefixes` не изменен.

## `normalizePath`

**Сигнатура типа:**

```ts
function normalizePath(id: string): string
```

**Связано:** [Нормализация путей](./api-plugin.md#path-normalization)

Нормализует путь для взаимодействия между плагинами Vite.

## `transformWithEsbuild`

**Сигнатура типа:**

```ts
async function transformWithEsbuild(
  code: string,
  filename: string,
  options?: EsbuildTransformOptions,
  inMap?: object,
): Promise<ESBuildTransformResult>
```

Трансформирует JavaScript или TypeScript с помощью esbuild. Полезно для плагинов, которые предпочитают соответствовать внутренней трансформации esbuild Vite.

## `loadConfigFromFile`

**Сигнатура типа:**

```ts
async function loadConfigFromFile(
  configEnv: ConfigEnv,
  configFile?: string,
  configRoot: string = process.cwd(),
  logLevel?: LogLevel,
  customLogger?: Logger,
): Promise<{
  path: string
  config: UserConfig
  dependencies: string[]
} | null>
```

Загружает файл конфигурации Vite вручную с помощью esbuild.

## `preprocessCSS`

- **Экспериментально:** [Оставить отзыв](https://github.com/vitejs/vite/discussions/13815)

**Сигнатура типа:**

```ts
async function preprocessCSS(
  code: string,
  filename: string,
  config: ResolvedConfig,
): Promise<PreprocessCSSResult>

interface PreprocessCSSResult {
  code: string
  map?: SourceMapInput
  modules?: Record<string, string>
  deps?: Set<string>
}
```

Предварительно обрабатывает файлы `.css`, `.scss`, `.sass`, `.less`, `.styl` и `.stylus` в обычный CSS, чтобы их можно было использовать в браузерах или обрабатывать другими инструментами. Аналогично [встроенной поддержке предварительной обработки CSS](/guide/features#css-pre-processors), соответствующий препроцессор должен быть установлен, если используется.

Используемый препроцессор определяется по расширению `filename`. Если `filename` заканчивается на `.module.{ext}`, он определяется как [CSS модуль](https://github.com/css-modules/css-modules), и возвращаемый результат будет включать объект `modules`, сопоставляющий исходные имена классов с преобразованными.

Обратите внимание, что предварительная обработка не будет разрешать URL в `url()` или `image-set()`.
