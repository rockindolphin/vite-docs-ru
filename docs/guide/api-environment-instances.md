# Использование экземпляров `Environment`

:::warning Экспериментально
API окружений является экспериментальным. Мы будем поддерживать API стабильным во время Vite 6, чтобы позволить экосистеме экспериментировать и строить на его основе. Мы планируем стабилизировать эти новые API с потенциальными breaking changes в Vite 7.

Ресурсы:

- [Обсуждение обратной связи](https://github.com/vitejs/vite/discussions/16358), где мы собираем отзывы о новых API.
- [PR API окружений](https://github.com/vitejs/vite/pull/16471), где новые API были реализованы и проверены.

Пожалуйста, поделитесь с нами вашими отзывами.
:::

## Доступ к окружениям

Во время разработки доступные окружения в dev-сервере можно получить с помощью `server.environments`:

```js
// создаем сервер или получаем его из хука configureServer
const server = await createServer(/* options */)

const environment = server.environments.client
environment.transformRequest(url)
console.log(server.environments.ssr.moduleGraph)
```

Вы также можете получить доступ к текущему окружению из плагинов. Подробнее см. [API окружений для плагинов](./api-environment-plugins.md#accessing-the-current-environment-in-hooks).

## Класс `DevEnvironment`

Во время разработки каждое окружение является экземпляром класса `DevEnvironment`:

```ts
class DevEnvironment {
  /**
   * Уникальный идентификатор окружения в сервере Vite.
   * По умолчанию Vite предоставляет окружения 'client' и 'ssr'.
   */
  name: string
  /**
   * Канал связи для отправки и получения сообщений от
   * связанного module runner в целевой среде выполнения.
   */
  hot: NormalizedHotChannel
  /**
   * Граф узлов модулей с отношениями импорта между
   * обработанными модулями и кэшированным результатом обработанного кода.
   */
  moduleGraph: EnvironmentModuleGraph
  /**
   * Разрешенные плагины для этого окружения, включая те,
   * которые созданы с использованием хука `create` для каждого окружения
   */
  plugins: Plugin[]
  /**
   * Позволяет разрешать, загружать и преобразовывать код через
   * конвейер плагинов окружения
   */
  pluginContainer: EnvironmentPluginContainer
  /**
   * Разрешенные опции конфигурации для этого окружения. Опции на уровне
   * глобального сервера берутся как значения по умолчанию для всех окружений и могут
   * быть переопределены (условия разрешения, внешние зависимости, optimizedDeps)
   */
  config: ResolvedConfig & ResolvedDevEnvironmentOptions

  constructor(
    name: string,
    config: ResolvedConfig,
    context: DevEnvironmentContext,
  )

  /**
   * Разрешает URL до id, загружает его и обрабатывает код с помощью
   * конвейера плагинов. Граф модулей также обновляется.
   */
  async transformRequest(url: string): Promise<TransformResult | null>

  /**
   * Регистрирует запрос для обработки с низким приоритетом. Это полезно
   * для избежания водопадов. Сервер Vite имеет информацию о
   * импортированных модулях другими запросами, поэтому он может прогреть граф модулей,
   * чтобы модули уже были обработаны, когда они запрашиваются.
   */
  async warmupRequest(url: string): Promise<void>
}
```

Где `DevEnvironmentContext`:

```ts
interface DevEnvironmentContext {
  hot: boolean
  transport?: HotChannel | WebSocketServer
  options?: EnvironmentOptions
  remoteRunner?: {
    inlineSourceMap?: boolean
  }
  depsOptimizer?: DepsOptimizer
}
```

И где `TransformResult`:

```ts
interface TransformResult {
  code: string
  map: SourceMap | { mappings: '' } | null
  etag?: string
  deps?: string[]
  dynamicDeps?: string[]
}
```

Экземпляр окружения в сервере Vite позволяет обрабатывать URL с помощью метода `environment.transformRequest(url)`. Эта функция будет использовать конвейер плагинов для разрешения `url` до `id` модуля, загрузки его (чтения файла из файловой системы или через плагин, реализующий виртуальный модуль), а затем преобразования кода. При обработке модуля импорты и другие метаданные записываются в граф модулей окружения путем создания или обновления соответствующего узла модуля. Когда обработка завершена, результат преобразования также сохраняется в модуле.

:::info Название transformRequest
Мы используем `transformRequest(url)` и `warmupRequest(url)` в текущей версии этого предложения, чтобы это было легче обсуждать и понимать для пользователей, привыкших к текущему API Vite. Перед выпуском мы можем воспользоваться возможностью пересмотреть эти названия. Например, они могут быть названы `environment.processModule(url)` или `environment.loadModule(url)`, беря пример из `context.load(id)` Rollup в хуках плагинов. На данный момент мы считаем, что лучше сохранить текущие названия и отложить это обсуждение.
:::

## Отдельные графы модулей

Каждое окружение имеет изолированный граф модулей. Все графы модулей имеют одинаковую сигнатуру, поэтому можно реализовать общие алгоритмы для обхода или запроса графа без зависимости от окружения. `hotUpdate` - хороший пример. Когда файл изменяется, граф модулей каждого окружения будет использоваться для обнаружения затронутых модулей и выполнения HMR для каждого окружения независимо.

::: info
Vite v5 имел смешанный граф модулей Client и SSR. Учитывая необработанный или недействительный узел, невозможно знать, соответствует ли он окружениям Client, SSR или обоим. Узлы модулей имеют некоторые свойства с префиксами, такие как `clientImportedModules` и `ssrImportedModules` (и `importedModules`, который возвращает объединение обоих). `importers` содержит все импортеры из окружений Client и SSR для каждого узла модуля. Узел модуля также имеет `transformResult` и `ssrTransformResult`. Слой обратной совместимости позволяет экосистеме мигрировать с устаревшего `server.moduleGraph`.
:::

Каждый модуль представлен экземпляром `EnvironmentModuleNode`. Модули могут быть зарегистрированы в графе без обработки (`transformResult` будет `null` в этом случае). `importers` и `importedModules` также обновляются после обработки модуля.

```ts
class EnvironmentModuleNode {
  environment: string

  url: string
  id: string | null = null
  file: string | null = null

  type: 'js' | 'css'

  importers = new Set<EnvironmentModuleNode>()
  importedModules = new Set<EnvironmentModuleNode>()
  importedBindings: Map<string, Set<string>> | null = null

  info?: ModuleInfo
  meta?: Record<string, any>
  transformResult: TransformResult | null = null

  acceptedHmrDeps = new Set<EnvironmentModuleNode>()
  acceptedHmrExports: Set<string> | null = null
  isSelfAccepting?: boolean
  lastHMRTimestamp = 0
  lastInvalidationTimestamp = 0
}
```

`environment.moduleGraph` является экземпляром `EnvironmentModuleGraph`:

```ts
export class EnvironmentModuleGraph {
  environment: string

  urlToModuleMap = new Map<string, EnvironmentModuleNode>()
  idToModuleMap = new Map<string, EnvironmentModuleNode>()
  etagToModuleMap = new Map<string, EnvironmentModuleNode>()
  fileToModulesMap = new Map<string, Set<EnvironmentModuleNode>>()

  constructor(
    environment: string,
    resolveId: (url: string) => Promise<PartialResolvedId | null>,
  )

  async getModuleByUrl(
    rawUrl: string,
  ): Promise<EnvironmentModuleNode | undefined>

  getModuleById(id: string): EnvironmentModuleNode | undefined

  getModulesByFile(file: string): Set<EnvironmentModuleNode> | undefined

  onFileChange(file: string): void

  onFileDelete(file: string): void

  invalidateModule(
    mod: EnvironmentModuleNode,
    seen: Set<EnvironmentModuleNode> = new Set(),
    timestamp: number = Date.now(),
    isHmr: boolean = false,
  ): void

  invalidateAll(): void

  async ensureEntryFromUrl(
    rawUrl: string,
    setIsSelfAccepting = true,
  ): Promise<EnvironmentModuleNode>

  createFileOnlyEntry(file: string): EnvironmentModuleNode

  async resolveUrl(url: string): Promise<ResolvedUrl>

  updateModuleTransformResult(
    mod: EnvironmentModuleNode,
    result: TransformResult | null,
  ): void

  getModuleByEtag(etag: string): EnvironmentModuleNode | undefined
}
```
