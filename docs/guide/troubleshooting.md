# Решение проблем

Также см. [руководство по решению проблем Rollup](https://rollupjs.org/troubleshooting/) для получения дополнительной информации.

Если предложенные здесь решения не работают, пожалуйста, попробуйте задать вопросы на [GitHub Discussions](https://github.com/vitejs/vite/discussions) или в канале `#help` в [Vite Land Discord](https://chat.vite.dev).

## CJS

### Node API Vite в формате CJS устарел

Сборка Node API Vite в формате CJS устарела и будет удалена в Vite 6. См. [обсуждение на GitHub](https://github.com/vitejs/vite/discussions/13928) для получения дополнительного контекста. Вам следует обновить ваши файлы или фреймворки, чтобы импортировать ESM-сборку Vite.

В базовом проекте Vite убедитесь, что:

1. Содержимое файла `vite.config.js` использует синтаксис ESM.
2. Ближайший файл `package.json` имеет `"type": "module"`, или используйте расширение `.mjs`/`.mts`, например, `vite.config.mjs` или `vite.config.mts`.

Для других проектов есть несколько общих подходов:

- **Настроить ESM по умолчанию, при необходимости включить CJS:** Добавьте `"type": "module"` в `package.json` проекта. Все файлы `*.js` теперь интерпретируются как ESM и должны использовать синтаксис ESM. Вы можете переименовать файл с расширением `.cjs`, чтобы продолжать использовать CJS.
- **Оставить CJS по умолчанию, при необходимости включить ESM:** Если в `package.json` проекта нет `"type": "module"`, все файлы `*.js` интерпретируются как CJS. Вы можете переименовать файл с расширением `.mjs`, чтобы использовать ESM.
- **Динамически импортировать Vite:** Если вам нужно продолжать использовать CJS, вы можете динамически импортировать Vite с помощью `import('vite')`. Это требует, чтобы ваш код был написан в контексте `async`, но все равно должен быть управляемым, так как API Vite в основном асинхронный.

Если вы не уверены, откуда приходит предупреждение, вы можете запустить ваш скрипт с флагом `VITE_CJS_TRACE=true`, чтобы вывести стек вызовов:

```bash
VITE_CJS_TRACE=true vite dev
```

Если вы хотите временно игнорировать предупреждение, вы можете запустить ваш скрипт с флагом `VITE_CJS_IGNORE_WARNING=true`:

```bash
VITE_CJS_IGNORE_WARNING=true vite dev
```

Обратите внимание, что файлы конфигурации postcss пока не поддерживают ESM + TypeScript (`.mts` или `.ts` в `"type": "module"`). Если у вас есть конфигурации postcss с `.ts` и вы добавили `"type": "module"` в package.json, вам также нужно будет переименовать конфигурацию postcss, чтобы использовать `.cts`.

## CLI

### `Error: Cannot find module 'C:\foo\bar&baz\vite\bin\vite.js'`

Путь к вашей папке проекта может содержать `&`, что не работает с `npm` в Windows ([npm/cmd-shim#45](https://github.com/npm/cmd-shim/issues/45)).

Вам нужно будет либо:

- Переключиться на другой менеджер пакетов (например, `pnpm`, `yarn`)
- Удалить `&` из пути к вашему проекту

## Конфигурация

### Этот пакет только для ESM

При импорте пакета только для ESM с помощью `require` возникает следующая ошибка.

> Failed to resolve "foo". This package is ESM only but it was tried to load by `require`.

> Error [ERR_REQUIRE_ESM]: require() of ES Module /path/to/dependency.js from /path/to/vite.config.js not supported.
> Instead change the require of index.js in /path/to/vite.config.js to a dynamic import() which is available in all CommonJS modules.

В Node.js <=22 файлы ESM не могут быть загружены с помощью [`require`](https://nodejs.org/docs/latest-v22.x/api/esm.html#require) по умолчанию.

Хотя это может работать с использованием [`--experimental-require-module`](https://nodejs.org/docs/latest-v22.x/api/modules.html#loading-ecmascript-modules-using-require), или Node.js >22, или в других средах выполнения, мы все равно рекомендуем преобразовать вашу конфигурацию в ESM, либо:

- добавив `"type": "module"` в ближайший `package.json`
- переименовав `vite.config.js`/`vite.config.ts` в `vite.config.mjs`/`vite.config.mts`

## Dev Server

### Запросы зависают навсегда

Если вы используете Linux, ограничения на количество файловых дескрипторов и inotify могут вызывать проблему. Поскольку Vite не бандлит большинство файлов, браузеры могут запрашивать много файлов, что требует много файловых дескрипторов, превышая лимит.

Чтобы решить это:

- Увеличьте лимит файловых дескрипторов с помощью `ulimit`

  ```shell
  # Проверить текущий лимит
  $ ulimit -Sn
  # Изменить лимит (временно)
  $ ulimit -Sn 10000 # Возможно, вам также нужно будет изменить жесткий лимит
  # Перезапустите ваш браузер
  ```

- Увеличьте следующие связанные с inotify лимиты с помощью `sysctl`

  ```shell
  # Проверить текущие лимиты
  $ sysctl fs.inotify
  # Изменить лимиты (временно)
  $ sudo sysctl fs.inotify.max_queued_events=16384
  $ sudo sysctl fs.inotify.max_user_instances=8192
  $ sudo sysctl fs.inotify.max_user_watches=524288
  ```

Если вышеуказанные шаги не работают, вы можете попробовать добавить `DefaultLimitNOFILE=65536` как не закомментированную конфигурацию в следующие файлы:

- /etc/systemd/system.conf
- /etc/systemd/user.conf

Для Ubuntu Linux вам может потребоваться добавить строку `* - nofile 65536` в файл `/etc/security/limits.conf` вместо обновления файлов конфигурации systemd.

Обратите внимание, что эти настройки сохраняются, но **требуется перезагрузка**.

Альтернативно, если сервер запущен внутри devcontainer VS Code, запрос может казаться зависшим. Чтобы исправить эту проблему, см.
[Dev Containers / VS Code Port Forwarding](#dev-containers-vs-code-port-forwarding).

### Сетевые запросы перестают загружаться

При использовании самоподписанного SSL-сертификата Chrome игнорирует все директивы кэширования и перезагружает содержимое. Vite полагается на эти директивы кэширования.

Чтобы решить проблему, используйте доверенный SSL-сертификат.

См.: [Проблемы с кэшем](https://helpx.adobe.com/mt/experience-manager/kb/cache-problems-on-chrome-with-SSL-certificate-errors.html), [Проблема Chrome](https://bugs.chromium.org/p/chromium/issues/detail?id=110649#c8)

#### macOS

Вы можете установить доверенный сертификат через CLI с помощью этой команды:

```
security add-trusted-cert -d -r trustRoot -k ~/Library/Keychains/login.keychain-db your-cert.cer
```

Или, импортировав его в приложение Keychain Access и обновив доверие к вашему сертификату на "Always Trust."

### 431 Request Header Fields Too Large

Когда сервер / WebSocket сервер получает большой HTTP-заголовок, запрос будет отброшен и будет показано следующее предупреждение.

> Server responded with status code 431. See https://vite.dev/guide/troubleshooting.html#_431-request-header-fields-too-large.

Это происходит потому, что Node.js ограничивает размер заголовков запроса для смягчения [CVE-2018-12121](https://www.cve.org/CVERecord?id=CVE-2018-12121).

Чтобы избежать этого, попробуйте уменьшить размер заголовка запроса. Например, если cookie длинный, удалите его. Или вы можете использовать [`--max-http-header-size`](https://nodejs.org/api/cli.html#--max-http-header-sizesize), чтобы изменить максимальный размер заголовка.

### Dev Containers / VS Code Port Forwarding

Если вы используете Dev Container или функцию переадресации портов в VS Code, вам может потребоваться установить опцию [`server.host`](/config/server-options.md#server-host) в `127.0.0.1` в конфигурации, чтобы это работало.

Это происходит потому, что [функция переадресации портов в VS Code не поддерживает IPv6](https://github.com/microsoft/vscode-remote-release/issues/7029).

См. [#16522](https://github.com/vitejs/vite/issues/16522) для получения дополнительных сведений.

## HMR

### Vite обнаруживает изменение файла, но HMR не работает

Возможно, вы импортируете файл с другим регистром. Например, существует `src/foo.js`, а `src/bar.js` содержит:

```js
import './Foo.js' // должно быть './foo.js'
```

Связанная проблема: [#964](https://github.com/vitejs/vite/issues/964)

### Vite не обнаруживает изменение файла

Если вы запускаете Vite с WSL2, Vite не может отслеживать изменения файлов в некоторых условиях. См. [опцию `server.watch`](/config/server-options.md#server-watch).

### Происходит полная перезагрузка вместо HMR

Если HMR не обрабатывается Vite или плагином, произойдет полная перезагрузка, так как это единственный способ обновить состояние.

Если HMR обрабатывается, но находится в циклической зависимости, также произойдет полная перезагрузка для восстановления порядка выполнения. Чтобы решить это, попробуйте разорвать цикл. Вы можете запустить `vite --debug hmr`, чтобы вывести путь циклической зависимости, если изменение файла вызвало его.

## Сборка

### Собранный файл не работает из-за ошибки CORS

Если HTML-файл был открыт с помощью протокола `file`, скрипты не будут выполняться со следующей ошибкой.

> Access to script at 'file:///foo/bar.js' from origin 'null' has been blocked by CORS policy: Cross origin requests are only supported for protocol schemes: http, data, isolated-app, chrome-extension, chrome, https, chrome-untrusted.

> Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at file:///foo/bar.js. (Reason: CORS request not http).

См. [Reason: CORS request not HTTP - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSRequestNotHttp) для получения дополнительной информации о том, почему это происходит.

Вам нужно будет получить доступ к файлу с помощью протокола `http`. Самый простой способ достичь этого - запустить `npx vite preview`.

## Оптимизированные зависимости

### Устаревшие предварительно собранные зависимости при связывании с локальным пакетом

Хеш-ключ, используемый для инвалидации оптимизированных зависимостей, зависит от содержимого package-lock, примененных патчей к зависимостям и опций в файле конфигурации Vite, которые влияют на сборку node modules. Это означает, что Vite обнаружит, когда зависимость переопределена с помощью функции, такой как [npm overrides](https://docs.npmjs.com/cli/v9/configuring-npm/package-json#overrides), и пересоберет ваши зависимости при следующем запуске сервера. Vite не будет инвалидировать зависимости, когда вы используете функцию, такую как [npm link](https://docs.npmjs.com/cli/v9/commands/npm-link). В случае, если вы связываете или развязываете зависимость, вам нужно будет принудительно переоптимизировать при следующем запуске сервера, используя `vite --force`. Мы рекомендуем использовать overrides вместо этого, которые теперь поддерживаются каждым менеджером пакетов (см. также [pnpm overrides](https://pnpm.io/package_json#pnpmoverrides) и [yarn resolutions](https://yarnpkg.com/configuration/manifest/#resolutions)).

## Узкие места производительности

Если вы страдаете от узких мест производительности приложения, приводящих к медленной загрузке, вы можете запустить встроенный инспектор Node.js с вашим dev-сервером Vite или при сборке вашего приложения для создания профиля CPU:

::: code-group

```bash [dev server]
vite --profile --open
```

```bash [build]
vite build --profile
```

:::

::: tip Vite Dev Server
После того, как ваше приложение открыто в браузере, просто дождитесь завершения его загрузки, затем вернитесь в терминал и нажмите клавишу `p` (остановит инспектор Node.js), затем нажмите клавишу `q`, чтобы остановить dev-сервер.
:::

Инспектор Node.js сгенерирует `vite-profile-0.cpuprofile` в корневой папке, перейдите на https://www.speedscope.app/ и загрузите профиль CPU с помощью кнопки `BROWSE`, чтобы проверить результат.

Вы можете установить [vite-plugin-inspect](https://github.com/antfu/vite-plugin-inspect), который позволяет вам проверять промежуточное состояние плагинов Vite и также может помочь вам определить, какие плагины или middleware являются узким местом в ваших приложениях. Плагин можно использовать как в режиме dev, так и в режиме сборки. Проверьте файл readme для получения дополнительных сведений.

## Другие

### Модуль исключен для совместимости с браузером

Когда вы используете модуль Node.js в браузере, Vite выведет следующее предупреждение.

> Module "fs" has been externalized for browser compatibility. Cannot access "fs.readFile" in client code.

Это происходит потому, что Vite не полифиллит модули Node.js автоматически.

Мы рекомендуем избегать модулей Node.js для кода браузера, чтобы уменьшить размер бандла, хотя вы можете добавить полифиллы вручную. Если модуль импортируется из сторонней библиотеки (которая предназначена для использования в браузере), рекомендуется сообщить о проблеме в соответствующую библиотеку.

### Происходит синтаксическая ошибка / ошибка типа

Vite не может обрабатывать и не поддерживает код, который работает только в нестрогом режиме (sloppy mode). Это происходит потому, что Vite использует ESM, и он всегда в [строгом режиме](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) внутри ESM.

Например, вы можете увидеть эти ошибки.

> [ERROR] With statements cannot be used with the "esm" output format due to strict mode

> TypeError: Cannot create property 'foo' on boolean 'false'

Если эти коды используются внутри зависимостей, вы можете использовать [`patch-package`](https://github.com/ds300/patch-package) (или [`yarn patch`](https://yarnpkg.com/cli/patch) или [`pnpm patch`](https://pnpm.io/cli/patch)) для обходного пути.

### Расширения браузера

Некоторые расширения браузера (например, блокировщики рекламы) могут препятствовать отправке запросов клиентом Vite на dev-сервер Vite. В этом случае вы можете увидеть белый экран без зарегистрированных ошибок. Попробуйте отключить расширения, если у вас есть эта проблема.

### Перекрестные ссылки на дисках в Windows

Если в вашем проекте на Windows есть перекрестные ссылки на дисках, Vite может не работать.

Примеры перекрестных ссылок на дисках:

- виртуальный диск, связанный с папкой командой `subst`
- символическая ссылка/соединение на другой диск командой `mklink` (например, глобальный кэш Yarn)

Связанная проблема: [#10802](https://github.com/vitejs/vite/issues/10802)
