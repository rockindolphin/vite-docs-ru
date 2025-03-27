# API окружений для сред выполнения

:::warning Экспериментально
API окружений является экспериментальным. Мы будем поддерживать API стабильным во время Vite 6, чтобы позволить экосистеме экспериментировать и строить на его основе. Мы планируем стабилизировать эти новые API с потенциальными breaking changes в Vite 7.

Ресурсы:

- [Обсуждение обратной связи](https://github.com/vitejs/vite/discussions/16358), где мы собираем отзывы о новых API.
- [PR API окружений](https://github.com/vitejs/vite/pull/16471), где новые API были реализованы и проверены.

Пожалуйста, поделитесь с нами вашими отзывами.
:::

## Фабрики окружений

Фабрики окружений предназначены для реализации поставщиками окружений, такими как Cloudflare, а не конечными пользователями. Фабрики окружений возвращают `EnvironmentOptions` для наиболее распространенного случая использования целевой среды выполнения как для dev, так и для build окружений. Значения по умолчанию для опций окружения также могут быть установлены, чтобы пользователю не нужно было это делать.

```ts
function createWorkerdEnvironment(
  userConfig: EnvironmentOptions,
): EnvironmentOptions {
  return mergeConfig(
    {
      resolve: {
        conditions: [
          /*...*/
        ],
      },
      dev: {
        createEnvironment(name, config) {
          return createWorkerdDevEnvironment(name, config, {
            hot: true,
            transport: customHotChannel(),
          })
        },
      },
      build: {
        createEnvironment(name, config) {
          return createWorkerdBuildEnvironment(name, config)
        },
      },
    },
    userConfig,
  )
}
```

Затем файл конфигурации может быть записан как:

```js
import { createWorkerdEnvironment } from 'vite-environment-workerd'

export default {
  environments: {
    ssr: createWorkerdEnvironment({
      build: {
        outDir: '/dist/ssr',
      },
    }),
    rsc: createWorkerdEnvironment({
      build: {
        outDir: '/dist/rsc',
      },
    }),
  },
}
```

И фреймворки могут использовать окружение со средой выполнения workerd для выполнения SSR с помощью:

```js
const ssrEnvironment = server.environments.ssr
```

## Создание новой фабрики окружений

Сервер разработки Vite по умолчанию раскрывает два окружения: окружение `client` и окружение `ssr`. Окружение client по умолчанию является окружением браузера, а module runner реализован путем импорта виртуального модуля `/@vite/client` в клиентские приложения. Окружение SSR по умолчанию работает в той же среде выполнения Node, что и сервер Vite, и позволяет использовать серверы приложений для рендеринга запросов во время разработки с полной поддержкой HMR.

Преобразованный исходный код называется модулем, а отношения между модулями, обработанными в каждом окружении, хранятся в графе модулей. Преобразованный код для этих модулей отправляется в среды выполнения, связанные с каждым окружением, для выполнения. Когда модуль оценивается в среде выполнения, его импортированные модули будут запрашиваться, что вызывает обработку части графа модулей.

Module Runner Vite позволяет запускать любой код, предварительно обработав его с помощью плагинов Vite. Он отличается от `server.ssrLoadModule`, потому что реализация runner отделена от сервера. Это позволяет авторам библиотек и фреймворков реализовать свой слой коммуникации между сервером Vite и runner. Браузер общается со своим соответствующим окружением с помощью Web Socket сервера и через HTTP-запросы. Node Module runner может напрямую выполнять вызовы функций для обработки модулей, так как он работает в том же процессе. Другие окружения могут запускать модули, подключаясь к среде выполнения JavaScript, такой как workerd, или к Worker Thread, как это делает Vitest.

Одна из целей этой функции - предоставить настраиваемый API для обработки и запуска кода. Пользователи могут создавать новые фабрики окружений, используя раскрытые примитивы.

```ts
import { DevEnvironment, HotChannel } from 'vite'

function createWorkerdDevEnvironment(
  name: string,
  config: ResolvedConfig,
  context: DevEnvironmentContext
) {
  const connection = /* ... */
  const transport: HotChannel = {
    on: (listener) => { connection.on('message', listener) },
    send: (data) => connection.send(data),
  }

  const workerdDevEnvironment = new DevEnvironment(name, config, {
    options: {
      resolve: { conditions: ['custom'] },
      ...context.options,
    },
    hot: true,
    transport,
  })
  return workerdDevEnvironment
}
```

## `ModuleRunner`

Module runner создается в целевой среде выполнения. Все API в следующем разделе импортируются из `vite/module-runner`, если не указано иное. Эта точка входа экспорта поддерживается как можно более легкой, экспортируя только минимально необходимое для создания module runner.

**Сигнатура типа:**

```ts
export class ModuleRunner {
  constructor(
    public options: ModuleRunnerOptions,
    public evaluator: ModuleEvaluator = new ESModulesEvaluator(),
    private debug?: ModuleRunnerDebugger,
  ) {}
  /**
   * URL для выполнения.
   * Принимает путь к файлу, путь сервера или id относительно корня.
   */
  public async import<T = any>(url: string): Promise<T>
  /**
   * Очищает все кэши, включая слушатели HMR.
   */
  public clearCache(): void
  /**
   * Очищает все кэши, удаляет все слушатели HMR, сбрасывает поддержку sourcemap.
   * Этот метод не останавливает соединение HMR.
   */
  public async close(): Promise<void>
  /**
   * Возвращает `true`, если runner был закрыт путем вызова `close()`.
   */
  public isClosed(): boolean
}
```

Оценщик модулей в `ModuleRunner` отвечает за выполнение кода. Vite экспортирует `ESModulesEvaluator` из коробки, он использует `new AsyncFunction` для оценки кода. Вы можете предоставить свою собственную реализацию, если ваша среда выполнения JavaScript не поддерживает небезопасную оценку.

Module runner раскрывает метод `import`. Когда сервер Vite вызывает событие HMR `full-reload`, все затронутые модули будут перевыполнены. Обратите внимание, что Module Runner не обновляет объект `exports`, когда это происходит (он перезаписывает его), вам нужно будет запустить `import` или получить модуль из `evaluatedModules` снова, если вы полагаетесь на наличие последнего объекта `exports`.

**Пример использования:**

```js
import { ModuleRunner, ESModulesEvaluator } from 'vite/module-runner'
import { transport } from './rpc-implementation.js'

const moduleRunner = new ModuleRunner(
  {
    transport,
  },
  new ESModulesEvaluator(),
)

await moduleRunner.import('/src/entry-point.js')
```

## `ModuleRunnerOptions`

```ts twoslash
import type {
  InterceptorOptions as InterceptorOptionsRaw,
  ModuleRunnerHmr as ModuleRunnerHmrRaw,
  EvaluatedModules,
} from 'vite/module-runner'
import type { Debug } from '@type-challenges/utils'

type InterceptorOptions = Debug<InterceptorOptionsRaw>
type ModuleRunnerHmr = Debug<ModuleRunnerHmrRaw>
/** see below */
type ModuleRunnerTransport = unknown

// ---cut---
interface ModuleRunnerOptions {
  /**
   * Набор методов для связи с сервером.
   */
  transport: ModuleRunnerTransport
  /**
   * Настройка разрешения source maps.
   * Предпочитает `node`, если доступен `process.setSourceMapsEnabled`.
   * В противном случае по умолчанию будет использовать `prepareStackTrace`, который переопределяет
   * метод `Error.prepareStackTrace`.
   * Вы можете предоставить объект для настройки того, как содержимое файлов и
   * source maps разрешаются для файлов, которые не были обработаны Vite.
   */
  sourcemapInterceptor?:
    | false
    | 'node'
    | 'prepareStackTrace'
    | InterceptorOptions
  /**
   * Отключить HMR или настроить опции HMR.
   *
   * @default true
   */
  hmr?: boolean | ModuleRunnerHmr
  /**
   * Пользовательский кэш модулей. Если не предоставлен, создается отдельный кэш модулей
   * для каждого экземпляра module runner.
   */
  evaluatedModules?: EvaluatedModules
}
```

## `ModuleEvaluator`

**Сигнатура типа:**

```ts twoslash
import type { ModuleRunnerContext as ModuleRunnerContextRaw } from 'vite/module-runner'
import type { Debug } from '@type-challenges/utils'

type ModuleRunnerContext = Debug<ModuleRunnerContextRaw>

// ---cut---
export interface ModuleEvaluator {
  /**
   * Количество префиксных строк в преобразованном коде.
   */
  startOffset?: number
  /**
   * Оценивает код, который был преобразован Vite.
   * @param context Контекст функции
   * @param code Преобразованный код
   * @param id ID, который использовался для получения модуля
   */
  runInlinedModule(
    context: ModuleRunnerContext,
    code: string,
    id: string,
  ): Promise<any>
  /**
   * Оценивает внешний модуль.
   * @param file URL файла внешнего модуля
   */
  runExternalModule(file: string): Promise<any>
}
```

Vite экспортирует `ESModulesEvaluator`, который по умолчанию реализует этот интерфейс. Он использует `new AsyncFunction` для оценки кода, поэтому, если код имеет встроенный source map, он должен содержать [смещение в 2 строки](https://tc39.es/ecma262/#sec-createdynamicfunction), чтобы приспособиться к новым добавленным строкам. Это делается автоматически `ESModulesEvaluator`. Пользовательские оценщики не будут добавлять дополнительные строки.

## `ModuleRunnerTransport`

Объект транспорта, который общается со средой через RPC или путем прямого вызова функции. Когда метод `invoke` не реализован, необходимо реализовать методы `send` и `connect`. Vite построит `invoke` внутренне.

Вам нужно связать его с экземпляром `HotChannel` на сервере, как в этом примере, где module runner создается в worker thread:

::: code-group

```js [worker.js]
import { parentPort } from 'node:worker_threads'
import { fileURLToPath } from 'node:url'
import { ESModulesEvaluator, ModuleRunner } from 'vite/module-runner'

/** @type {import('vite/module-runner').ModuleRunnerTransport} */
const transport = {
  connect({ onMessage, onDisconnection }) {
    parentPort.on('message', onMessage)
    parentPort.on('close', onDisconnection)
  },
  send(data) {
    parentPort.postMessage(data)
  },
}

const runner = new ModuleRunner(
  {
    transport,
  },
  new ESModulesEvaluator(),
)
```

```js [server.js]
import { BroadcastChannel } from 'node:worker_threads'
import { createServer, RemoteEnvironmentTransport, DevEnvironment } from 'vite'

function createWorkerEnvironment(name, config, context) {
  const worker = new Worker('./worker.js')
  const handlerToWorkerListener = new WeakMap()

  const workerHotChannel = {
    send: (data) => worker.postMessage(data),
    on: (event, handler) => {
      if (event === 'connection') return

      const listener = (value) => {
        if (value.type === 'custom' && value.event === event) {
          const client = {
            send(payload) {
              worker.postMessage(payload)
            },
          }
          handler(value.data, client)
        }
      }
      handlerToWorkerListener.set(handler, listener)
      worker.on('message', listener)
    },
    off: (event, handler) => {
      if (event === 'connection') return
      const listener = handlerToWorkerListener.get(handler)
      if (listener) {
        worker.off('message', listener)
        handlerToWorkerListener.delete(handler)
      }
    },
  }

  return new DevEnvironment(name, config, {
    transport: workerHotChannel,
  })
}

await createServer({
  environments: {
    worker: {
      dev: {
        createEnvironment: createWorkerEnvironment,
      },
    },
  },
})
```

:::

Другой пример использования HTTP-запроса для связи между runner и сервером:

```ts
import { ESModulesEvaluator, ModuleRunner } from 'vite/module-runner'

export const runner = new ModuleRunner(
  {
    transport: {
      async invoke(data) {
        const response = await fetch(`http://my-vite-server/invoke`, {
          method: 'POST',
          body: JSON.stringify(data),
        })
        return response.json()
      },
    },
    hmr: false, // отключаем HMR, так как HMR требует transport.connect
  },
  new ESModulesEvaluator(),
)

await runner.import('/entry.js')
```

В этом случае можно использовать метод `handleInvoke` в `NormalizedHotChannel`:

```ts
const customEnvironment = new DevEnvironment(name, config, context)

server.onRequest((request: Request) => {
  const url = new URL(request.url)
  if (url.pathname === '/invoke') {
    const payload = (await request.json()) as HotPayload
    const result = customEnvironment.hot.handleInvoke(payload)
    return new Response(JSON.stringify(result))
  }
  return Response.error()
})
```

Обратите внимание, что для поддержки HMR требуются методы `send` и `connect`. Метод `send` обычно вызывается, когда срабатывает пользовательское событие (например, `import.meta.hot.send("my-event")`).

Vite экспортирует `createServerHotChannel` из основной точки входа для поддержки HMR во время Vite SSR.
