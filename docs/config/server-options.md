# Опции сервера

Если не указано иное, опции в этом разделе применяются только в режиме разработки.

## server.host

- **Тип:** `string | boolean`
- **По умолчанию:** `'localhost'`

Укажите, на каких IP-адресах сервер должен слушать.
Установите это значение в `0.0.0.0` или `true`, чтобы слушать на всех адресах, включая LAN и публичные адреса.

Это можно установить через CLI, используя `--host 0.0.0.0` или `--host`.

::: tip ПРИМЕЧАНИЕ

Существуют случаи, когда другие серверы могут отвечать вместо Vite.

Первый случай — когда используется `localhost`. Node.js версии до 17 по умолчанию изменяет порядок результатов DNS-резолвированных адресов. При доступе к `localhost` браузеры используют DNS для разрешения адреса, и этот адрес может отличаться от адреса, на котором слушает Vite. Vite выводит разрешенный адрес, когда он отличается.

Вы можете установить [`dns.setDefaultResultOrder('verbatim')`](https://nodejs.org/api/dns.html#dns_dns_setdefaultresultorder_order), чтобы отключить поведение изменения порядка. Vite тогда будет выводить адрес как `localhost`.

```js twoslash [vite.config.js]
import { defineConfig } from 'vite'
import dns from 'node:dns'

dns.setDefaultResultOrder('verbatim')

export default defineConfig({
  // опустить
})
```

Второй случай — когда используются универсальные хосты (например, `0.0.0.0`). Это связано с тем, что серверы, слушающие на неуниверсальных хостах, имеют приоритет над теми, которые слушают на универсальных хостах.

:::

::: tip Доступ к серверу на WSL2 из вашей локальной сети

При запуске Vite на WSL2 недостаточно установить `host: true`, чтобы получить доступ к серверу из вашей локальной сети.
Смотрите [документ WSL](https://learn.microsoft.com/en-us/windows/wsl/networking#accessing-a-wsl-2-distribution-from-your-local-area-network-lan) для получения дополнительных сведений.

:::

## server.allowedHosts

- **Тип:** `string[] | true`
- **По умолчанию:** `[]`

Имена хостов, на которые Vite разрешено отвечать.
`localhost` и домены под `.localhost`, а также все IP-адреса разрешены по умолчанию.
При использовании HTTPS эта проверка пропускается.

Если строка начинается с `.`, это позволит этому имени хоста без `.` и всем поддоменам под этим именем. Например, `.example.com` позволит `example.com`, `foo.example.com` и `foo.bar.example.com`. Если установлено в `true`, серверу разрешено отвечать на запросы для любых хостов.

::: details Какие хосты безопасно добавлять?

Хосты, над которыми вы имеете контроль, какие IP-адреса они разрешают, безопасно добавлять в список разрешенных хостов.

Например, если вы владеете доменом `vite.dev`, вы можете добавить `vite.dev` и `.vite.dev` в список. Если вы не владеете этим доменом и не можете доверять владельцу этого домена, вы не должны его добавлять.

Особенно вы никогда не должны добавлять домены верхнего уровня, такие как `.com`, в список. Это связано с тем, что любой может купить домен, например, `example.com`, и контролировать IP-адрес, на который он разрешается.

:::

::: danger

Установка `server.allowedHosts` в `true` позволяет любому веб-сайту отправлять запросы на ваш сервер разработки через атаки переопределения DNS, позволяя им загружать ваш исходный код и контент. Мы рекомендуем всегда использовать явный список разрешенных хостов. Смотрите [GHSA-vg6x-rcgg-rjx6](https://github.com/vitejs/vite/security/advisories/GHSA-vg6x-rcgg-rjx6) для получения дополнительных сведений.

:::

::: details Настройка через переменную окружения
Вы можете установить переменную окружения `__VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS`, чтобы добавить дополнительный разрешенный хост.
:::

## server.port

- **Тип:** `number`
- **По умолчанию:** `5173`

Укажите порт сервера. Обратите внимание, если порт уже используется, Vite автоматически попытается использовать следующий доступный порт, поэтому это может не быть фактическим портом, на котором сервер в конечном итоге будет слушать.

## server.strictPort

- **Тип:** `boolean`

Установите в `true`, чтобы выйти, если порт уже используется, вместо автоматической попытки использовать следующий доступный порт.

## server.https

- **Тип:** `https.ServerOptions`

Включите TLS + HTTP/2. Значение — это [объект опций](https://nodejs.org/api/https.html#https_https_createserver_options_requestlistener), передаваемый в `https.createServer()`.

Обратите внимание, что это понижается до TLS только в том случае, если также используется опция [`server.proxy`](#server-proxy).

Необходим действительный сертификат. Для базовой настройки вы можете добавить [@vitejs/plugin-basic-ssl](https://github.com/vitejs/vite-plugin-basic-ssl) в плагины проекта, который автоматически создаст и кэширует самоподписанный сертификат. Но мы рекомендуем создавать свои собственные сертификаты.

## server.open

- **Тип:** `boolean | string`

Автоматически открывать приложение в браузере при запуске сервера. Когда значение является строкой, оно будет использоваться как путь URL. Если вы хотите открыть сервер в конкретном браузере, который вам нравится, вы можете установить переменную окружения `process.env.BROWSER` (например, `firefox`). Вы также можете установить `process.env.BROWSER_ARGS`, чтобы передать дополнительные аргументы (например, `--incognito`).

`BROWSER` и `BROWSER_ARGS` также являются специальными переменными окружения, которые вы можете установить в файле `.env`, чтобы настроить их. Смотрите [пакет `open`](https://github.com/sindresorhus/open#app) для получения дополнительных сведений.

**Пример:**

```js
export default defineConfig({
  server: {
    open: '/docs/index.html',
  },
})
```

## server.proxy

- **Тип:** `Record<string, string | ProxyOptions>`

Настройте пользовательские правила прокси для сервера разработки. Ожидает объект пар `{ ключ: опции }`. Любые запросы, путь которых начинается с этого ключа, будут проксироваться на указанный целевой адрес. Если ключ начинается с `^`, он будет интерпретироваться как `RegExp`. Опция `configure` может быть использована для доступа к экземпляру прокси. Если запрос соответствует любому из настроенных правил прокси, запрос не будет преобразован Vite.

Обратите внимание, что если вы используете нерелятивный [`base`](/config/shared-options.md#base), вы должны префиксировать каждый ключ этим `base`.

Расширяет [`http-proxy`](https://github.com/http-party/node-http-proxy#options). Дополнительные опции [здесь](https://github.com/vitejs/vite/blob/main/packages/vite/src/node/server/middlewares/proxy.ts#L13).

В некоторых случаях вам также может понадобиться настроить внутренний сервер разработки (например, чтобы добавить пользовательские промежуточные программы в внутреннее приложение [connect](https://github.com/senchalabs/connect)). Для этого вам нужно написать свой собственный [плагин](/guide/using-plugins.html) и использовать функцию [configureServer](/guide/api-plugin.html#configureserver).

**Пример:**

```js
export default defineConfig({
  server: {
    proxy: {
      // строковое сокращение:
      // http://localhost:5173/foo
      //   -> http://localhost:4567/foo
      '/foo': 'http://localhost:4567',
      // с опциями:
      // http://localhost:5173/api/bar
      //   -> http://jsonplaceholder.typicode.com/bar
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
      // с RegExp:
      // http://localhost:5173/fallback/
      //   -> http://jsonplaceholder.typicode.com/
      '^/fallback/.*': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, ''),
      },
      // Используя экземпляр прокси
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        configure: (proxy, options) => {
          // proxy будет экземпляром 'http-proxy'
        },
      },
      // Проксирование веб-сокетов или socket.io:
      // ws://localhost:5173/socket.io
      //   -> ws://localhost:5174/socket.io
      // Будьте осторожны при использовании `rewriteWsOrigin`, так как это может оставить
      // проксирование открытым для атак CSRF.
      '/socket.io': {
        target: 'ws://localhost:5174',
        ws: true,
        rewriteWsOrigin: true,
      },
    },
  },
})
```

## server.cors

- **Тип:** `boolean | CorsOptions`
- **По умолчанию:** `{ origin: /^https?:\/\/(?:(?:[^:]+\.)?localhost|127\.0\.0.1|\[::1\])(?::\d+)?$/ }` (разрешает localhost, `127.0.0.1` и `::1`)

Настройте CORS для сервера разработки. Передайте [объект опций](https://github.com/expressjs/cors#configuration-options), чтобы точно настроить поведение, или `true`, чтобы разрешить любой источник.

::: danger

Установка `server.cors` в `true` позволяет любому веб-сайту отправлять запросы на ваш сервер разработки и загружать ваш исходный код и контент. Мы рекомендуем всегда использовать явный список разрешенных источников.

:::

## server.headers

- **Тип:** `OutgoingHttpHeaders`

Укажите заголовки ответа сервера.

## server.hmr

- **Тип:** `boolean | { protocol?: string, host?: string, port?: number, path?: string, timeout?: number, overlay?: boolean, clientPort?: number, server?: Server }`

Отключите или настройте соединение HMR (в случаях, когда веб-сокет HMR должен использовать другой адрес, чем http-сервер).

Установите `server.hmr.overlay` в `false`, чтобы отключить наложение ошибок сервера.

`protocol` устанавливает протокол WebSocket, используемый для соединения HMR: `ws` (WebSocket) или `wss` (WebSocket Secure).

`clientPort` — это расширенная опция, которая переопределяет порт только на стороне клиента, позволяя вам обслуживать веб-сокет на другом порту, чем тот, который ищет клиентский код.

Когда `server.hmr.server` определен, Vite будет обрабатывать запросы соединения HMR через предоставленный сервер. Если не в режиме промежуточного ПО, Vite попытается обработать запросы соединения HMR через существующий сервер. Это может быть полезно при использовании самоподписанных сертификатов или когда вы хотите открыть Vite через сеть на одном порту.

Посмотрите [`vite-setup-catalogue`](https://github.com/sapphi-red/vite-setup-catalogue) для некоторых примеров.

::: tip ПРИМЕЧАНИЕ

С конфигурацией по умолчанию ожидается, что обратные прокси перед Vite поддерживают проксирование веб-сокетов. Если клиент HMR Vite не может подключиться к веб-сокету, клиент будет пытаться подключиться к веб-сокету напрямую к серверу HMR Vite, обходя обратные прокси:

```
Direct websocket connection fallback. Check out https://vite.dev/config/server-options.html#server-hmr to remove the previous connection error.
```

Ошибку, которая появляется в браузере при возникновении отката, можно игнорировать. Чтобы избежать ошибки, напрямую обойдя обратные прокси, вы можете:

- настроить обратный прокси-сервер для прокси-сервера WebSocket
- установить [`server.strictPort = true`](#server-strictport) и установить `server.hmr.clientPort` на то же значение, что и `server.port`
- установить `server.hmr.port` на другое значение, чем [`server.port`](#server-port)

:::

## server.warmup

- **Тип:** `{ clientFiles?: string[], ssrFiles?: string[] }`
- **Связанный:** [Разогрев часто используемых файлов](/guide/performance.html#warm-up-frequently-used-files)

Разогрев файлов для предварительного преобразования и кэширования результатов. Это улучшает начальную загрузку страницы при запуске сервера и предотвращает каскадные преобразования.

`clientFiles` - это файлы, которые используются только на клиенте, а `ssrFiles` - это файлы, которые используются только в SSR. Они принимают массив путей к файлам или паттерны [`tinyglobby`](https://github.com/SuperchupuDev/tinyglobby) относительно `root`.

Убедитесь, что вы добавляете только часто используемые файлы, чтобы не перегружать сервер разработки Vite при запуске.

```js
export default defineConfig({
  server: {
    warmup: {
      clientFiles: ['./src/components/*.vue', './src/utils/big-utils.js'],
      ssrFiles: ['./src/server/modules/*.js'],
    },
  },
})
```

## server.watch

- **Тип:** `object | null`

Опции наблюдателя файловой системы для передачи в [chokidar](https://github.com/paulmillr/chokidar/tree/3.6.0#api).

Наблюдатель сервера Vite следит за `root` и по умолчанию пропускает директории `.git/`, `node_modules/`, а также `cacheDir` и `build.outDir` Vite. При обновлении отслеживаемого файла Vite применит HMR и обновит страницу только при необходимости.

Если установлено значение `null`, файлы не будут отслеживаться. `server.watcher` предоставит совместимый эмиттер событий, но вызовы `add` или `unwatch` не будут иметь эффекта.

::: warning Отслеживание файлов в `node_modules`

В настоящее время невозможно отслеживать файлы и пакеты в `node_modules`. Для получения дополнительной информации о прогрессе и обходных путях, следите за [issue #8619](https://github.com/vitejs/vite/issues/8619).

:::

::: warning Использование Vite в Windows Subsystem for Linux (WSL) 2

При запуске Vite на WSL2 отслеживание файловой системы не работает, когда файл редактируется приложениями Windows (процесс не-WSL2). Это связано с [ограничением WSL2](https://github.com/microsoft/WSL/issues/4739). Это также применяется при запуске в Docker с бэкендом WSL2.

Чтобы исправить это, вы можете:

- **Рекомендуется**: Использовать приложения WSL2 для редактирования файлов.
  - Также рекомендуется переместить папку проекта за пределы файловой системы Windows. Доступ к файловой системе Windows из WSL2 медленный. Удаление этой нагрузки улучшит производительность.
- Установить `{ usePolling: true }`.
  - Обратите внимание, что [`usePolling` приводит к высокому использованию CPU](https://github.com/paulmillr/chokidar/tree/3.6.0#performance).

:::

## server.middlewareMode

- **Тип:** `boolean`
- **По умолчанию:** `false`

Создание сервера Vite в режиме промежуточного ПО.

- **Связанный:** [appType](./shared-options#apptype), [SSR - Настройка сервера разработки](/guide/ssr#setting-up-the-dev-server)

- **Пример:**

```js twoslash
import express from 'express'
import { createServer as createViteServer } from 'vite'

async function createServer() {
  const app = express()

  // Создание сервера Vite в режиме промежуточного ПО
  const vite = await createViteServer({
    server: { middlewareMode: true },
    // не включать стандартные промежуточные ПО Vite для обработки HTML
    appType: 'custom',
  })
  // Использование экземпляра connect Vite в качестве промежуточного ПО
  app.use(vite.middlewares)

  app.use('*', async (req, res) => {
    // Поскольку `appType` установлен как `'custom'`, здесь нужно обработать ответ.
    // Примечание: если `appType` установлен как `'spa'` или `'mpa'`, Vite включает промежуточные ПО
    // для обработки HTML-запросов и 404, поэтому пользовательские промежуточные ПО должны быть добавлены
    // перед промежуточными ПО Vite, чтобы вступить в силу
  })
}

createServer()
```

## server.fs.strict

- **Тип:** `boolean`
- **По умолчанию:** `true` (включено по умолчанию с Vite 2.7)

Ограничение обслуживания файлов вне корня рабочего пространства.

## server.fs.allow

- **Тип:** `string[]`

Ограничение файлов, которые могут обслуживаться через `/@fs/`. Когда `server.fs.strict` установлен в `true`, доступ к файлам вне этого списка директорий, которые не импортируются из разрешенного файла, приведет к ошибке 403.

Можно указать как директории, так и файлы.

Vite будет искать корень потенциального рабочего пространства и использовать его по умолчанию. Действительное рабочее пространство соответствует следующим условиям, в противном случае будет использоваться [корень проекта](/guide/#index-html-and-project-root).

- содержит поле `workspaces` в `package.json`
- содержит один из следующих файлов
  - `lerna.json`
  - `pnpm-workspace.yaml`

Принимает путь для указания пользовательского корня рабочего пространства. Может быть абсолютным путем или путем относительно [корня проекта](/guide/#index-html-and-project-root). Например:

```js
export default defineConfig({
  server: {
    fs: {
      // Разрешить обслуживание файлов на один уровень выше корня проекта
      allow: ['..'],
    },
  },
})
```

Когда указан `server.fs.allow`, автоматическое определение корня рабочего пространства будет отключено. Для расширения исходного поведения предоставляется утилита `searchForWorkspaceRoot`:

```js
import { defineConfig, searchForWorkspaceRoot } from 'vite'

export default defineConfig({
  server: {
    fs: {
      allow: [
        // поиск вверх для корня рабочего пространства
        searchForWorkspaceRoot(process.cwd()),
        // ваши пользовательские правила
        '/path/to/custom/allow_directory',
        '/path/to/custom/allow_file.demo',
      ],
    },
  },
})
```

## server.fs.deny

- **Тип:** `string[]`
- **По умолчанию:** `['.env', '.env.*', '*.{crt,pem}', '**/.git/**']`

Список блокировки конфиденциальных файлов, которые ограничены от обслуживания сервером разработки Vite. Это имеет более высокий приоритет, чем [`server.fs.allow`](#server-fs-allow). Поддерживаются [паттерны picomatch](https://github.com/micromatch/picomatch#globbing-features).

## server.origin

- **Тип:** `string`

Определяет источник сгенерированных URL-адресов ресурсов во время разработки.

```js
export default defineConfig({
  server: {
    origin: 'http://127.0.0.1:8080',
  },
})
```

## server.sourcemapIgnoreList

- **Тип:** `false | (sourcePath: string, sourcemapPath: string) => boolean`
- **По умолчанию:** `(sourcePath) => sourcePath.includes('node_modules')`

Игнорировать или нет исходные файлы в sourcemap сервера, используется для заполнения расширения sourcemap [`x_google_ignoreList`](https://developer.chrome.com/articles/x-google-ignore-list/).

`server.sourcemapIgnoreList` эквивалентен [`build.rollupOptions.output.sourcemapIgnoreList`](https://rollupjs.org/configuration-options/#output-sourcemapignorelist) для сервера разработки. Разница между двумя опциями конфигурации заключается в том, что функция rollup вызывается с относительным путем для `sourcePath`, в то время как `server.sourcemapIgnoreList` вызывается с абсолютным путем. Во время разработки большинство модулей имеют карту и исходный код в одной папке, поэтому относительный путь для `sourcePath` - это само имя файла. В этих случаях абсолютные пути удобнее использовать.

По умолчанию исключаются все пути, содержащие `node_modules`. Вы можете передать `false`, чтобы отключить это поведение, или, для полного контроля, функцию, которая принимает путь к исходному файлу и путь к sourcemap и возвращает, следует ли игнорировать путь к исходному файлу.

```js
export default defineConfig({
  server: {
    // Это значение по умолчанию, и оно добавит все файлы с node_modules
    // в их путях в список игнорирования.
    sourcemapIgnoreList(sourcePath, sourcemapPath) {
      return sourcePath.includes('node_modules')
    },
  },
})
```

::: tip Примечание
[`server.sourcemapIgnoreList`](#server-sourcemapignorelist) и [`build.rollupOptions.output.sourcemapIgnoreList`](https://rollupjs.org/configuration-options/#output-sourcemapignorelist) нужно устанавливать независимо. `server.sourcemapIgnoreList` - это конфигурация только для сервера и не получает значение по умолчанию из определенных опций rollup.
:::
