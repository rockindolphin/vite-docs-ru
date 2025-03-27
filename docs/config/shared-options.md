# Общие Опции

Если не указано иное, опции в этом разделе применяются ко всем режимам разработки, сборки и предпросмотра.

## root

- **Тип:** `string`
- **По умолчанию:** `process.cwd()`

Корневая директория проекта (где находится `index.html`). Может быть абсолютным путем или путем относительно текущей рабочей директории.

Подробнее см. [Корневая директория проекта](/guide/#index-html-and-project-root).

## base

- **Тип:** `string`
- **По умолчанию:** `/`
- **Связано:** [`server.origin`](/config/server-options.md#server-origin)

Базовый публичный путь при запуске в режиме разработки или продакшена. Допустимые значения включают:

- Абсолютный URL-путь, например `/foo/`
- Полный URL, например `https://bar.com/foo/` (Часть origin не будет использоваться в режиме разработки, поэтому значение такое же, как `/foo/`)
- Пустая строка или `./` (для встроенного развертывания)

Подробнее см. [Публичный базовый путь](/guide/build#public-base-path).

## mode

- **Тип:** `string`
- **По умолчанию:** `'development'` для serve, `'production'` для build

Указание этого в конфигурации переопределит режим по умолчанию **как для serve, так и для build**. Это значение также может быть переопределено через опцию командной строки `--mode`.

Подробнее см. [Переменные окружения и режимы](/guide/env-and-mode).

## define

- **Тип:** `Record<string, any>`

Определение глобальных констант для замены. Записи будут определены как глобальные переменные во время разработки и статически заменены во время сборки.

Vite использует [определения esbuild](https://esbuild.github.io/api/#define) для выполнения замен, поэтому выражения значений должны быть строками, содержащими JSON-сериализуемое значение (null, boolean, number, string, array или object) или одним идентификатором. Для нестроковых значений Vite автоматически преобразует их в строку с помощью `JSON.stringify`.

**Пример:**

```js
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify('v1.0.0'),
    __API_URL__: 'window.__backend_api_url',
  },
})
```

::: tip ПРИМЕЧАНИЕ
Для пользователей TypeScript убедитесь, что вы добавили объявления типов в файл `env.d.ts` или `vite-env.d.ts`, чтобы получить проверку типов и поддержку IntelliSense.

Пример:

```ts
// vite-env.d.ts
declare const __APP_VERSION__: string
```

:::

## plugins

- **Тип:** `(Plugin | Plugin[] | Promise<Plugin | Plugin[]>)[]`

Массив плагинов для использования. Ложные плагины игнорируются, а массивы плагинов сглаживаются. Если возвращается промис, он будет разрешен перед запуском. Подробнее о плагинах Vite см. [Plugin API](/guide/api-plugin).

## publicDir

- **Тип:** `string | false`
- **По умолчанию:** `"public"`

Директория для обслуживания статических ресурсов. Файлы в этой директории обслуживаются по пути `/` во время разработки и копируются в корень `outDir` во время сборки, и всегда обслуживаются или копируются как есть без трансформации. Значение может быть либо абсолютным путем файловой системы, либо путем относительно корня проекта.

Установка `publicDir` как `false` отключает эту функцию.

Подробнее см. [Директория `public`](/guide/assets#the-public-directory).

## cacheDir

- **Тип:** `string`
- **По умолчанию:** `"node_modules/.vite"`

Директория для сохранения кэш-файлов. Файлы в этой директории - это предварительно собранные зависимости или другие кэш-файлы, сгенерированные vite, которые могут улучшить производительность. Вы можете использовать флаг `--force` или вручную удалить директорию для регенерации кэш-файлов. Значение может быть либо абсолютным путем файловой системы, либо путем относительно корня проекта. По умолчанию `.vite`, если package.json не обнаружен.

## resolve.alias

- **Тип:**
  `Record<string, string> | Array<{ find: string | RegExp, replacement: string, customResolver?: ResolverFunction | ResolverObject }>`

Будет передано в `@rollup/plugin-alias` как его [опция entries](https://github.com/rollup/plugins/tree/master/packages/alias#entries). Может быть либо объектом, либо массивом пар `{ find, replacement, customResolver }`.

При создании алиасов для путей файловой системы всегда используйте абсолютные пути. Относительные значения алиасов будут использоваться как есть и не будут преобразованы в пути файловой системы.

Более продвинутое пользовательское разрешение может быть достигнуто через [плагины](/guide/api-plugin).

::: warning Использование с SSR
Если вы настроили алиасы для [SSR внешних зависимостей](/guide/ssr.md#ssr-externals), возможно, вы захотите создать алиасы для реальных пакетов `node_modules`. И [Yarn](https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-alias), и [pnpm](https://pnpm.io/aliases/) поддерживают алиасы через префикс `npm:`.
:::

## resolve.dedupe

- **Тип:** `string[]`

Если у вас есть дублирующиеся копии одной и той же зависимости в вашем приложении (вероятно, из-за подъема или связанных пакетов в монорепозиториях), используйте эту опцию, чтобы заставить Vite всегда разрешать перечисленные зависимости к одной и той же копии (из корня проекта).

:::warning SSR + ESM
Для SSR сборок дедупликация не работает для ESM выходных данных сборки, настроенных из `build.rollupOptions.output`. Обходной путь - использовать CJS выходные данные до тех пор, пока ESM не получит лучшую поддержку плагинов для загрузки модулей.
:::

## resolve.conditions

- **Тип:** `string[]`
- **По умолчанию:** `['module', 'browser', 'development|production']` (`defaultClientConditions`)

Дополнительные разрешенные условия при разрешении [Условных экспортов](https://nodejs.org/api/packages.html#packages_conditional_exports) из пакета.

Пакет с условными экспортами может иметь следующее поле `exports` в своем `package.json`:

```json
{
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.js"
    }
  }
}
```

Здесь `import` и `require` - это "условия". Условия могут быть вложенными и должны быть указаны от наиболее специфичных к наименее специфичным.

`development|production` - это специальное значение, которое заменяется на `production` или `development` в зависимости от значения `process.env.NODE_ENV`. Оно заменяется на `production`, когда `process.env.NODE_ENV === 'production'`, и на `development` в противном случае.

Обратите внимание, что условия `import`, `require`, `default` всегда применяются, если требования выполнены.

:::warning Разрешение подпутей экспорта
Ключи экспорта, заканчивающиеся на "/", устарели в Node и могут работать некорректно. Пожалуйста, свяжитесь с автором пакета, чтобы использовать [шаблоны подпутей `*`](https://nodejs.org/api/packages.html#package-entry-points) вместо этого.
:::

## resolve.mainFields

- **Тип:** `string[]`
- **По умолчанию:** `['browser', 'module', 'jsnext:main', 'jsnext']` (`defaultClientMainFields`)

Список полей в `package.json`, которые нужно проверять при разрешении точки входа пакета. Обратите внимание, что это имеет более низкий приоритет, чем условные экспорты, разрешенные из поля `exports`: если точка входа успешно разрешена из `exports`, поле main будет проигнорировано.

## resolve.extensions

- **Тип:** `string[]`
- **По умолчанию:** `['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json']`

Список расширений файлов для проверки при импортах без расширений. Обратите внимание, что **НЕ** рекомендуется опускать расширения для пользовательских типов импорта (например, `.vue`), так как это может мешать поддержке IDE и типов.

## resolve.preserveSymlinks

- **Тип:** `boolean`
- **По умолчанию:** `false`

Включение этой настройки заставляет vite определять идентичность файла по оригинальному пути файла (т.е. пути без следования по символическим ссылкам) вместо реального пути файла (т.е. пути после следования по символическим ссылкам).

- **Связано:** [esbuild#preserve-symlinks](https://esbuild.github.io/api/#preserve-symlinks), [webpack#resolve.symlinks](https://webpack.js.org/configuration/resolve/#resolvesymlinks)

## html.cspNonce

- **Тип:** `string`
- **Связано:** [Content Security Policy (CSP)](/guide/features#content-security-policy-csp)

Заполнитель значения nonce, который будет использоваться при генерации тегов script / style. Установка этого значения также сгенерирует мета-тег со значением nonce.

## css.modules

- **Тип:**
  ```ts
  interface CSSModulesOptions {
    getJSON?: (
      cssFileName: string,
      json: Record<string, string>,
      outputFileName: string,
    ) => void
    scopeBehaviour?: 'global' | 'local'
    globalModulePaths?: RegExp[]
    exportGlobals?: boolean
    generateScopedName?:
      | string
      | ((name: string, filename: string, css: string) => string)
    hashPrefix?: string
    /**
     * по умолчанию: undefined
     */
    localsConvention?:
      | 'camelCase'
      | 'camelCaseOnly'
      | 'dashes'
      | 'dashesOnly'
      | ((
          originalClassName: string,
          generatedClassName: string,
          inputFile: string,
        ) => string)
  }
  ```

Настройка поведения CSS модулей. Опции передаются в [postcss-modules](https://github.com/css-modules/postcss-modules).

Эта опция не имеет никакого эффекта при использовании [Lightning CSS](../guide/features.md#lightning-css). Если включена, вместо этого следует использовать [`css.lightningcss.cssModules`](https://lightningcss.dev/css-modules.html).

## css.postcss

- **Тип:** `string | (postcss.ProcessOptions & { plugins?: postcss.AcceptedPlugin[] })`

Встроенная конфигурация PostCSS или пользовательская директория для поиска конфигурации PostCSS (по умолчанию - корень проекта).

Для встроенной конфигурации PostCSS ожидается тот же формат, что и в `postcss.config.js`. Но для свойства `plugins` можно использовать только [формат массива](https://github.com/postcss/postcss-load-config/blob/main/README.md#array).

Поиск выполняется с помощью [postcss-load-config](https://github.com/postcss/postcss-load-config) и загружаются только поддерживаемые имена конфигурационных файлов. Конфигурационные файлы вне корня рабочего пространства (или [корня проекта](/guide/#index-html-and-project-root), если рабочее пространство не найдено) по умолчанию не ищутся. При необходимости вы можете указать пользовательский путь вне корня для загрузки конкретного конфигурационного файла.

Обратите внимание, что если предоставлена встроенная конфигурация, Vite не будет искать другие источники конфигурации PostCSS.

## css.preprocessorOptions

- **Тип:** `Record<string, object>`

Укажите опции для передачи в CSS препроцессоры. Расширения файлов используются как ключи для опций. Поддерживаемые опции для каждого препроцессора можно найти в их соответствующей документации:

- `sass`/`scss`:
  - Выберите API sass для использования с `api: "modern-compiler" | "modern" | "legacy"` (по умолчанию `"modern-compiler"`, если установлен `sass-embedded`, иначе `"modern"`). Для лучшей производительности рекомендуется использовать `api: "modern-compiler"` с пакетом `sass-embedded`. API `"legacy"` устарел и будет удален в Vite 7.
  - [Опции (modern)](https://sass-lang.com/documentation/js-api/interfaces/stringoptions/)
  - [Опции (legacy)](https://sass-lang.com/documentation/js-api/interfaces/LegacyStringOptions).
- `less`: [Опции](https://lesscss.org/usage/#less-options).
- `styl`/`stylus`: Поддерживается только [`define`](https://stylus-lang.com/docs/js.html#define-name-node), который может быть передан как объект.

**Пример:**

```js
export default defineConfig({
  css: {
    preprocessorOptions: {
      less: {
        math: 'parens-division',
      },
      styl: {
        define: {
          $specialColor: new stylus.nodes.RGBA(51, 197, 255, 1),
        },
      },
      scss: {
        api: 'modern-compiler', // или "modern", "legacy"
        importers: [
          // ...
        ],
      },
    },
  },
})
```

### css.preprocessorOptions[extension].additionalData

- **Тип:** `string | ((source: string, filename: string) => (string | { content: string; map?: SourceMap }))`

Эта опция может использоваться для внедрения дополнительного кода для каждого содержимого стиля. Обратите внимание, что если вы включаете реальные стили, а не только переменные, эти стили будут дублироваться в финальном бандле.

**Пример:**

```js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `$injectedColor: orange;`,
      },
    },
  },
})
```

## css.preprocessorMaxWorkers

- **Экспериментально:** [Дать обратную связь](https://github.com/vitejs/vite/discussions/15835)
- **Тип:** `number | true`
- **По умолчанию:** `0` (не создает никаких воркеров и запускается в основном потоке)

Если эта опция установлена, CSS препроцессоры будут запускаться в воркерах, когда это возможно. `true` означает количество CPU минус 1.

## css.devSourcemap

- **Экспериментально:** [Дать обратную связь](https://github.com/vitejs/vite/discussions/13845)
- **Тип:** `boolean`
- **По умолчанию:** `false`

Включать ли sourcemaps во время разработки.

## css.transformer

- **Экспериментально:** [Дать обратную связь](https://github.com/vitejs/vite/discussions/13835)
- **Тип:** `'postcss' | 'lightningcss'`
- **По умолчанию:** `'postcss'`

Выбирает движок, используемый для обработки CSS. Подробнее см. [Lightning CSS](../guide/features.md#lightning-css).

::: info Дублирование `@import`
Обратите внимание, что postcss (postcss-import) имеет другое поведение с дублированными `@import` по сравнению с браузерами. См. [postcss/postcss-import#462](https://github.com/postcss/postcss-import/issues/462).
:::

## css.lightningcss

- **Экспериментально:** [Дать обратную связь](https://github.com/vitejs/vite/discussions/13835)
- **Тип:**

```js
import type {
  CSSModulesConfig,
  Drafts,
  Features,
  NonStandard,
  PseudoClasses,
  Targets,
} from 'lightningcss'
```

```js
{
  targets?: Targets
  include?: Features
  exclude?: Features
  drafts?: Drafts
  nonStandard?: NonStandard
  pseudoClasses?: PseudoClasses
  unusedSymbols?: string[]
  cssModules?: CSSModulesConfig,
  // ...
}
```

Настраивает Lightning CSS. Полные опции трансформации можно найти в [репозитории Lightning CSS](https://github.com/parcel-bundler/lightningcss/blob/master/node/index.d.ts).

## json.namedExports

- **Тип:** `boolean`
- **По умолчанию:** `true`

Поддерживать ли именованные импорты из `.json` файлов.

## json.stringify

- **Тип:** `boolean | 'auto'`
- **По умолчанию:** `'auto'`

Если установлено в `true`, импортированный JSON будет преобразован в `export default JSON.parse("...")`, что значительно более производительно, чем объектные литералы, особенно когда JSON файл большой.

Если установлено в `'auto'`, данные будут преобразованы в строку только если [данные больше 10kB](https://v8.dev/blog/cost-of-javascript-2019#json:~:text=A%20good%20rule%20of%20thumb%20is%20to%20apply%20this%20technique%20for%20objects%20of%2010%20kB%20or%20larger).

## esbuild

- **Тип:** `ESBuildOptions | false`

`ESBuildOptions` расширяет [собственные опции трансформации esbuild](https://esbuild.github.io/api/#transform). Наиболее распространенный случай использования - это настройка JSX:

```js
export default defineConfig({
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment',
  },
})
```

По умолчанию esbuild применяется к файлам `ts`, `jsx` и `tsx`. Вы можете настроить это с помощью `esbuild.include` и `esbuild.exclude`, которые могут быть регулярным выражением, [шаблоном picomatch](https://github.com/micromatch/picomatch#globbing-features) или массивом любого из них.

Кроме того, вы также можете использовать `esbuild.jsxInject` для автоматического внедрения импортов вспомогательных функций JSX для каждого файла, трансформированного esbuild:

```js
export default defineConfig({
  esbuild: {
    jsxInject: `import React from 'react'`,
  },
})
```

Когда [`build.minify`](./build-options.md#build-minify) установлен в `true`, все оптимизации минификации применяются по умолчанию. Чтобы отключить [определенные аспекты](https://esbuild.github.io/api/#minify) этого, установите любую из опций `esbuild.minifyIdentifiers`, `esbuild.minifySyntax` или `esbuild.minifyWhitespace` в `false`. Обратите внимание, что опция `esbuild.minify` не может использоваться для переопределения `build.minify`.

Установите в `false`, чтобы отключить трансформации esbuild.

## assetsInclude

- **Тип:** `string | RegExp | (string | RegExp)[]`
- **Связано:** [Обработка статических ресурсов](/guide/assets)

Укажите дополнительные [шаблоны picomatch](https://github.com/micromatch/picomatch#globbing-features) для обработки как статических ресурсов, чтобы:

- Они исключались из конвейера трансформации плагинов при ссылке из HTML или прямом запросе через `fetch` или XHR.

- Импорт их из JS вернет их разрешенный URL строку (это может быть переопределено, если у вас есть плагин с `enforce: 'pre'` для обработки типа ресурса по-другому).

Встроенный список типов ресурсов можно найти [здесь](https://github.com/vitejs/vite/blob/main/packages/vite/src/node/constants.ts).

**Пример:**

```js
export default defineConfig({
  assetsInclude: ['**/*.gltf'],
})
```

## logLevel

- **Тип:** `'info' | 'warn' | 'error' | 'silent'`

Настройка уровня подробности вывода в консоль. По умолчанию `'info'`.

## customLogger

- **Тип:**
  ```ts
  interface Logger {
    info(msg: string, options?: LogOptions): void
    warn(msg: string, options?: LogOptions): void
    warnOnce(msg: string, options?: LogOptions): void
    error(msg: string, options?: LogErrorOptions): void
    clearScreen(type: LogType): void
    hasErrorLogged(error: Error | RollupError): boolean
    hasWarned: boolean
  }
  ```

Использовать пользовательский логгер для логирования сообщений. Вы можете использовать API `createLogger` Vite для получения логгера по умолчанию и его настройки, например, для изменения сообщения или фильтрации определенных предупреждений.

```ts twoslash
import { createLogger, defineConfig } from 'vite'

const logger = createLogger()
const loggerWarn = logger.warn

logger.warn = (msg, options) => {
  // Игнорировать предупреждение о пустых CSS файлах
  if (msg.includes('vite:css') && msg.includes(' is empty')) return
  loggerWarn(msg, options)
}

export default defineConfig({
  customLogger: logger,
})
```

## clearScreen

- **Тип:** `boolean`
- **По умолчанию:** `true`

Установите в `false`, чтобы предотвратить очистку экрана терминала Vite при логировании определенных сообщений. Через командную строку используйте `--clearScreen false`.

## envDir

- **Тип:** `string`
- **По умолчанию:** `root`

Директория, из которой загружаются файлы `.env`. Может быть абсолютным путем или путем относительно корня проекта.

Подробнее о файлах окружения см. [здесь](/guide/env-and-mode#env-files).

## envPrefix

- **Тип:** `string | string[]`
- **По умолчанию:** `VITE_`

Переменные окружения, начинающиеся с `envPrefix`, будут доступны в вашем клиентском исходном коде через import.meta.env.

:::warning ЗАМЕТКИ ПО БЕЗОПАСНОСТИ
`envPrefix` не должен быть установлен как `''`, что подвергнет все ваши переменные окружения и вызовет неожиданную утечку конфиденциальной информации. Vite выдаст ошибку при обнаружении `''`.

Если вы хотите раскрыть переменную без префикса, вы можете использовать [define](#define) для её раскрытия:

```js
define: {
  'import.meta.env.ENV_VARIABLE': JSON.stringify(process.env.ENV_VARIABLE)
}
```

:::

## appType

- **Тип:** `'spa' | 'mpa' | 'custom'`
- **По умолчанию:** `'spa'`

Является ли ваше приложение Single Page Application (SPA), [Multi Page Application (MPA)](../guide/build#multi-page-app) или Custom Application (SSR и фреймворки с пользовательской обработкой HTML):

- `'spa'`: включает HTML middleware и использует fallback SPA. Настройте [sirv](https://github.com/lukeed/sirv) с `single: true` в preview
- `'mpa'`: включает HTML middleware
- `'custom'`: не включает HTML middleware

Подробнее в [руководстве по SSR Vite](/guide/ssr#vite-cli). Связано: [`server.middlewareMode`](./server-options#server-middlewaremode).

## future

- **Тип:** `Record<string, 'warn' | undefined>`
- **Связано:** [Критические изменения](/changes/)

Включить будущие критические изменения для подготовки к плавной миграции на следующую основную версию Vite. Список может быть обновлен, добавлен или удален в любое время по мере разработки новых функций.

Подробности о возможных опциях см. на странице [Критические изменения](/changes/).
