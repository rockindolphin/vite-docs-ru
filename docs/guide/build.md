# Сборка для продакшена

Когда приходит время развернуть ваше приложение для продакшена, просто запустите команду `vite build`. По умолчанию она использует `<root>/index.html` как точку входа для сборки и создает бандл приложения, подходящий для размещения на статическом хостинге. Ознакомьтесь с [Развертыванием статического сайта](./static-deploy) для руководств по популярным сервисам.

## Совместимость с браузерами

По умолчанию production-бандл предполагает поддержку современного JavaScript, такого как [нативные ES модули](https://caniuse.com/es6-module), [нативный динамический импорт ESM](https://caniuse.com/es6-module-dynamic-import), [`import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta), [оператор нулевого слияния](https://caniuse.com/mdn-javascript_operators_nullish_coalescing) и [BigInt](https://caniuse.com/bigint). Диапазон поддержки браузеров по умолчанию:

<!-- Search for the `ESBUILD_MODULES_TARGET` constant for more information -->

- Chrome >=87
- Firefox >=78
- Safari >=14
- Edge >=88

Вы можете указать пользовательские цели через опцию конфигурации [`build.target`](/config/build-options.md#build-target), где самая низкая цель - `es2015`. Если установлена более низкая цель, Vite все равно будет требовать эти минимальные диапазоны поддержки браузеров, так как он полагается на [нативный динамический импорт ESM](https://caniuse.com/es6-module-dynamic-import) и [`import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta):

<!-- Search for the `defaultEsbuildSupported` constant for more information -->

- Chrome >=64
- Firefox >=67
- Safari >=11.1
- Edge >=79

Обратите внимание, что по умолчанию Vite обрабатывает только синтаксические преобразования и **не включает полифилы**. Вы можете проверить https://cdnjs.cloudflare.com/polyfill/, который автоматически генерирует бандлы полифилов на основе строки UserAgent браузера пользователя.

Устаревшие браузеры могут поддерживаться через [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy), который автоматически генерирует устаревшие чанки и соответствующие полифилы для функций языка ES. Устаревшие чанки загружаются условно только в браузерах, не поддерживающих нативный ESM.

## Базовый публичный путь

- См. также: [Обработка ресурсов](./assets)

Если вы развертываете ваш проект по вложенному публичному пути, просто укажите опцию конфигурации [`base`](/config/shared-options.md#base), и все пути к ресурсам будут переписаны соответственно. Эта опция также может быть указана как флаг командной строки, например `vite build --base=/my/public/path/`.

URL-адреса ресурсов, импортированных через JS, ссылки `url()` в CSS и ссылки на ресурсы в ваших `.html` файлах автоматически корректируются с учетом этой опции во время сборки.

Исключение составляет случай, когда вам нужно динамически объединять URL-адреса на лету. В этом случае вы можете использовать глобально внедренную переменную `import.meta.env.BASE_URL`, которая будет содержать базовый публичный путь. Обратите внимание, что эта переменная статически заменяется во время сборки, поэтому она должна появляться точно как есть (т.е. `import.meta.env['BASE_URL']` не будет работать).

Для расширенного контроля базового пути см. [Расширенные базовые опции](#advanced-base-options).

### Относительный базовый путь

Если вы не знаете базовый путь заранее, вы можете установить относительный базовый путь с помощью `"base": "./"` или `"base": ""`. Это сделает все сгенерированные URL-адреса относительными к каждому файлу.

:::warning Поддержка старых браузеров при использовании относительных базовых путей

Требуется поддержка `import.meta` для относительных базовых путей. Если вам нужно поддерживать [браузеры, не поддерживающие `import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta), вы можете использовать [плагин `legacy`](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy).

:::

## Настройка сборки

Сборку можно настроить с помощью различных [опций конфигурации сборки](/config/build-options.md). В частности, вы можете напрямую настроить базовые [опции Rollup](https://rollupjs.org/configuration-options/) через `build.rollupOptions`:

```js [vite.config.js]
export default defineConfig({
  build: {
    rollupOptions: {
      // https://rollupjs.org/configuration-options/
    },
  },
})
```

Например, вы можете указать несколько выходных данных Rollup с плагинами, которые применяются только во время сборки.

## Стратегия разделения на чанки

Вы можете настроить, как разделяются чанки, используя `build.rollupOptions.output.manualChunks` (см. [документацию Rollup](https://rollupjs.org/configuration-options/#output-manualchunks)). Если вы используете фреймворк, обратитесь к его документации для настройки разделения чанков.

## Обработка ошибок загрузки

Vite генерирует событие `vite:preloadError`, когда не удается загрузить динамические импорты. `event.payload` содержит оригинальную ошибку импорта. Если вы вызовете `event.preventDefault()`, ошибка не будет выброшена.

```js twoslash
window.addEventListener('vite:preloadError', (event) => {
  window.location.reload() // например, обновить страницу
})
```

Когда происходит новое развертывание, хостинг-сервис может удалить ресурсы из предыдущих развертываний. В результате пользователь, посетивший ваш сайт до нового развертывания, может столкнуться с ошибкой импорта. Эта ошибка возникает потому, что ресурсы, работающие на устройстве этого пользователя, устарели и пытаются импортировать соответствующий старый чанк, который был удален. Это событие полезно для решения такой ситуации.

## Пересборка при изменении файлов

Вы можете включить наблюдатель Rollup с помощью `vite build --watch`. Или вы можете напрямую настроить базовые [`WatcherOptions`](https://rollupjs.org/configuration-options/#watch) через `build.watch`:

```js [vite.config.js]
export default defineConfig({
  build: {
    watch: {
      // https://rollupjs.org/configuration-options/#watch
    },
  },
})
```

С включенным флагом `--watch` изменения в `vite.config.js`, а также любых файлах, которые нужно собрать, вызовут пересборку.

## Многостраничное приложение

Предположим, у вас есть следующая структура исходного кода:

```
├── package.json
├── vite.config.js
├── index.html
├── main.js
└── nested
    ├── index.html
    └── nested.js
```

Во время разработки просто перейдите или перейдите по ссылке на `/nested/` - это работает как ожидается, как и для обычного статического файлового сервера.

Во время сборки все, что вам нужно сделать, - это указать несколько `.html` файлов как точки входа:

```js twoslash [vite.config.js]
import { dirname, resolve } from 'node:path'
import { fileURLToPath } from 'node:url'
import { defineConfig } from 'vite'

const __dirname = dirname(fileURLToPath(import.meta.url))

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        nested: resolve(__dirname, 'nested/index.html'),
      },
    },
  },
})
```

Если вы указываете другой корневой каталог, помните, что `__dirname` все равно будет папкой вашего файла vite.config.js при разрешении путей ввода. Поэтому вам нужно будет добавить ваш `root` в аргументы для `resolve`.

Обратите внимание, что для HTML-файлов Vite игнорирует имя, данное входу в объекте `rollupOptions.input`, и вместо этого учитывает разрешенный id файла при генерации HTML-ресурса в папке dist. Это обеспечивает согласованную структуру с тем, как работает сервер разработки.

## Режим библиотеки

Когда вы разрабатываете библиотеку, ориентированную на браузер, вы, вероятно, проводите большую часть времени на тестовой/демонстрационной странице, которая импортирует вашу фактическую библиотеку. С Vite вы можете использовать ваш `index.html` для этой цели, чтобы получить плавный опыт разработки.

Когда приходит время собрать вашу библиотеку для распространения, используйте опцию конфигурации [`build.lib`](/config/build-options.md#build-lib). Убедитесь, что вы также исключили любые зависимости, которые не хотите включать в вашу библиотеку, например, `vue` или `react`:

::: code-group

```js twoslash [vite.config.js (одинарная точка входа)]
import { dirname, resolve } from 'node:path'
import { fileURLToPath } from 'node:url'
import { defineConfig } from 'vite'

const __dirname = dirname(fileURLToPath(import.meta.url))

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'lib/main.js'),
      name: 'MyLib',
      // будут добавлены правильные расширения
      fileName: 'my-lib',
    },
    rollupOptions: {
      // убедитесь, что вы исключили зависимости, которые не должны быть включены
      // в вашу библиотеку
      external: ['vue'],
      output: {
        // Предоставьте глобальные переменные для использования в UMD-сборке
        // для исключенных зависимостей
        globals: {
          vue: 'Vue',
        },
      },
    },
  },
})
```

```js twoslash [vite.config.js (множественные точки входа)]
import { dirname, resolve } from 'node:path'
import { fileURLToPath } from 'node:url'
import { defineConfig } from 'vite'

const __dirname = dirname(fileURLToPath(import.meta.url))

export default defineConfig({
  build: {
    lib: {
      entry: {
        'my-lib': resolve(__dirname, 'lib/main.js'),
        secondary: resolve(__dirname, 'lib/secondary.js'),
      },
      name: 'MyLib',
    },
    rollupOptions: {
      // убедитесь, что вы исключили зависимости, которые не должны быть включены
      // в вашу библиотеку
      external: ['vue'],
      output: {
        // Предоставьте глобальные переменные для использования в UMD-сборке
        // для исключенных зависимостей
        globals: {
          vue: 'Vue',
        },
      },
    },
  },
})
```

:::

Файл точки входа будет содержать экспорты, которые могут быть импортированы пользователями вашего пакета:

```js [lib/main.js]
import Foo from './Foo.vue'
import Bar from './Bar.vue'
export { Foo, Bar }
```

Запуск `vite build` с этой конфигурацией использует пресет Rollup, ориентированный на распространение библиотек, и создает два формата бандла:

- `es` и `umd` (для одинарной точки входа)
- `es` и `cjs` (для множественных точек входа)

Форматы можно настроить с помощью опции [`build.lib.formats`](/config/build-options.md#build-lib).

```
$ vite build
building for production...
dist/my-lib.js      0.08 kB / gzip: 0.07 kB
dist/my-lib.umd.cjs 0.30 kB / gzip: 0.16 kB
```

Рекомендуемый `package.json` для вашей библиотеки:

::: code-group

```json [package.json (одинарная точка входа)]
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.umd.cjs"
    }
  }
}
```

```json [package.json (множественные точки входа)]
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.cjs"
    },
    "./secondary": {
      "import": "./dist/secondary.js",
      "require": "./dist/secondary.cjs"
    }
  }
}
```

:::

### Поддержка CSS

Если ваша библиотека импортирует CSS, он будет собран в отдельный CSS файл вместе с JS файлами, например `dist/my-lib.css`. Имя по умолчанию берется из `build.lib.fileName`, но его можно изменить с помощью [`build.lib.cssFileName`](/config/build-options.md#build-lib).

Вы можете экспортировать CSS файл в вашем `package.json`, чтобы пользователи могли его импортировать:

```json {12}
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.umd.cjs"
    },
    "./style.css": "./dist/my-lib.css"
  }
}
```

::: tip Расширения файлов
Если в `package.json` не указано `"type": "module"`, Vite сгенерирует другие расширения файлов для совместимости с Node.js. `.js` станет `.mjs`, а `.cjs` станет `.js`.
:::

::: tip Переменные окружения
В режиме библиотеки все использования [`import.meta.env.*`](./env-and-mode.md) статически заменяются при сборке для продакшена. Однако использования `process.env.*` не заменяются, чтобы потребители вашей библиотеки могли динамически изменять их. Если это нежелательно, вы можете использовать `define: { 'process.env.NODE_ENV': '"production"' }` для статической замены или [`esm-env`](https://github.com/benmccann/esm-env) для лучшей совместимости с бандлерами и рантаймами.
:::

::: warning Расширенное использование
Режим библиотеки включает простую и предустановленную конфигурацию для библиотек, ориентированных на браузер и JS фреймворки. Если вы создаете библиотеки не для браузера или требуете расширенных процессов сборки, вы можете использовать [Rollup](https://rollupjs.org) или [esbuild](https://esbuild.github.io) напрямую.
:::

## Расширенные базовые опции

::: warning
Эта функция экспериментальная. [Оставить отзыв](https://github.com/vitejs/vite/discussions/13834).
:::

Для расширенных случаев использования развернутые ассеты и публичные файлы могут находиться в разных путях, например, для использования разных стратегий кэширования.
Пользователь может выбрать развертывание в трех разных путях:

- Сгенерированные входные HTML файлы (которые могут обрабатываться во время SSR)
- Сгенерированные хешированные ассеты (JS, CSS и другие типы файлов, такие как изображения)
- Скопированные [публичные файлы](assets.md#the-public-directory)

Одного статического [базового пути](#public-base-path) недостаточно в этих сценариях. Vite предоставляет экспериментальную поддержку расширенных базовых опций во время сборки с помощью `experimental.renderBuiltUrl`.

```ts twoslash
import type { UserConfig } from 'vite'
// prettier-ignore
const config: UserConfig = {
// ---cut-before---
experimental: {
  renderBuiltUrl(filename, { hostType }) {
    if (hostType === 'js') {
      return { runtime: `window.__toCdnUrl(${JSON.stringify(filename)})` }
    } else {
      return { relative: true }
    }
  },
},
// ---cut-after---
}
```

Если хешированные ассеты и публичные файлы не развертываются вместе, опции для каждой группы можно определить независимо, используя `type` ассета, включенный во второй параметр `context`, передаваемый в функцию.

```ts twoslash
import type { UserConfig } from 'vite'
import path from 'node:path'
// prettier-ignore
const config: UserConfig = {
// ---cut-before---
experimental: {
  renderBuiltUrl(filename, { hostId, hostType, type }) {
    if (type === 'public') {
      return 'https://www.domain.com/' + filename
    } else if (path.extname(hostId) === '.js') {
      return {
        runtime: `window.__assetsPath(${JSON.stringify(filename)})`
      }
    } else {
      return 'https://cdn.domain.com/assets/' + filename
    }
  },
},
// ---cut-after---
}
```

Обратите внимание, что переданный `filename` является декодированным URL, и если функция возвращает строку URL, она также должна быть декодирована. Vite автоматически обработает кодирование при рендеринге URL. Если возвращается объект с `runtime`, кодирование должно обрабатываться самостоятельно там, где это необходимо, так как код рантайма будет отрендерен как есть.
