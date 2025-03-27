# Возможности

На самом базовом уровне разработка с использованием Vite не сильно отличается от использования статического файлового сервера. Однако Vite предоставляет множество улучшений по сравнению с нативными ESM импортами для поддержки различных функций, которые обычно встречаются в настройках на основе бандлеров.

## Разрешение npm зависимостей и предварительная бандлизация

Нативные ES импорты не поддерживают "голые" импорты модулей, такие как:

```js
import { someMethod } from 'my-dep'
```

Вышеуказанное вызовет ошибку в браузере. Vite обнаружит такие "голые" импорты модулей во всех обслуживаемых исходных файлах и выполнит следующее:

1. [Предварительно соберет](./dep-pre-bundling) их для улучшения скорости загрузки страницы и преобразования CommonJS / UMD модулей в ESM. Предварительная сборка выполняется с помощью [esbuild](http://esbuild.github.io/) и делает время холодного запуска Vite значительно быстрее, чем у любого JavaScript-бандлера.

2. Перепишет импорты в валидные URL, такие как `/node_modules/.vite/deps/my-dep.js?v=f3sf2ebd`, чтобы браузер мог правильно их импортировать.

**Зависимости сильно кэшируются**

Vite кэширует запросы зависимостей через HTTP заголовки, поэтому если вы хотите локально отредактировать/отладить зависимость, следуйте шагам [здесь](./dep-pre-bundling#browser-cache).

## Hot Module Replacement

Vite предоставляет [HMR API](./api-hmr) поверх нативного ESM. Фреймворки с возможностями HMR могут использовать API для предоставления мгновенных, точных обновлений без перезагрузки страницы или потери состояния приложения. Vite предоставляет интеграции HMR первого уровня для [Vue Single File Components](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue) и [React Fast Refresh](https://github.com/vitejs/vite-plugin-react/tree/main/packages/plugin-react). Также есть официальные интеграции для Preact через [@prefresh/vite](https://github.com/JoviDeCroock/prefresh/tree/main/packages/vite).

Обратите внимание, что вам не нужно вручную настраивать это - когда вы [создаете приложение через `create-vite`](./), выбранные шаблоны уже будут иметь это предварительно настроенным.

## TypeScript

Vite поддерживает импорт `.ts` файлов из коробки.

### Только транспиляция

Обратите внимание, что Vite выполняет только транспиляцию `.ts` файлов и **НЕ** выполняет проверку типов. Предполагается, что проверка типов выполняется вашей IDE и процессом сборки.

Причина, по которой Vite не выполняет проверку типов как часть процесса трансформации, заключается в том, что эти две задачи работают принципиально по-разному. Транспиляция может работать на основе каждого файла и идеально соответствует модели компиляции по требованию Vite. Для сравнения, проверка типов требует знания всего графа модулей. Втискивание проверки типов в конвейер трансформации Vite неизбежно поставит под угрозу преимущества скорости Vite.

Задача Vite - как можно быстрее преобразовать ваши исходные модули в форму, которая может работать в браузере. С этой целью мы рекомендуем отделить статический анализ от конвейера трансформации Vite. Этот принцип применим к другим проверкам статического анализа, таким как ESLint.

- Для production сборок вы можете запустить `tsc --noEmit` в дополнение к команде сборки Vite.

- Во время разработки, если вам нужно больше, чем подсказки IDE, мы рекомендуем запускать `tsc --noEmit --watch` в отдельном процессе или использовать [vite-plugin-checker](https://github.com/fi3ework/vite-plugin-checker), если вы предпочитаете видеть ошибки типов непосредственно в браузере.

Vite использует [esbuild](https://github.com/evanw/esbuild) для транспиляции TypeScript в JavaScript, что примерно в 20~30 раз быстрее, чем vanilla `tsc`, и обновления HMR могут отражаться в браузере менее чем за 50мс.

Используйте синтаксис [Type-Only Imports and Export](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3.8.html#type-only-imports-and-export) для избежания потенциальных проблем, таких как неправильная бандлизация импортов только для типов, например:

```ts
import type { T } from 'only/types'
export type { T }
```

### Опции компилятора TypeScript

Некоторые поля под `compilerOptions` в `tsconfig.json` требуют особого внимания.

#### `isolatedModules`

- [Документация TypeScript](https://www.typescriptlang.org/tsconfig#isolatedModules)

Должно быть установлено в `true`.

Это потому, что `esbuild` выполняет только транспиляцию без информации о типах, он не поддерживает определенные функции, такие как const enum и неявные импорты только для типов.

Вы должны установить `"isolatedModules": true` в вашем `tsconfig.json` под `compilerOptions`, чтобы TS предупреждал вас о функциях, которые не работают с изолированной транспиляцией.

Если зависимость не работает хорошо с `"isolatedModules": true`. Вы можете использовать `"skipLibCheck": true` для временного подавления ошибок, пока это не будет исправлено вышестоящими разработчиками.

#### `useDefineForClassFields`

- [Документация TypeScript](https://www.typescriptlang.org/tsconfig#useDefineForClassFields)

Значение по умолчанию будет `true`, если цель TypeScript - `ES2022` или новее, включая `ESNext`. Это согласуется с [поведением TypeScript 4.3.2+](https://github.com/microsoft/TypeScript/pull/42663).
Другие цели TypeScript будут по умолчанию установлены в `false`.

`true` - это стандартное поведение времени выполнения ECMAScript.

Если вы используете библиотеку, которая сильно зависит от полей класса, пожалуйста, будьте осторожны с предполагаемым использованием библиотекой этого.
Хотя большинство библиотек ожидают `"useDefineForClassFields": true`, вы можете явно установить `useDefineForClassFields` в `false`, если ваша библиотека не поддерживает это.

#### `target`

- [Документация TypeScript](https://www.typescriptlang.org/tsconfig#target)

Vite игнорирует значение `target` в `tsconfig.json`, следуя тому же поведению, что и `esbuild`.

Чтобы указать цель в dev, можно использовать опцию [`esbuild.target`](/config/shared-options.html#esbuild), которая по умолчанию установлена в `esnext` для минимальной транспиляции. В сборках опция [`build.target`](/config/build-options.html#build-target) имеет более высокий приоритет над `esbuild.target` и также может быть установлена при необходимости.

::: warning `useDefineForClassFields`

Если `target` в `tsconfig.json` не `ESNext` или `ES2022` или новее, или если файла `tsconfig.json` нет, `useDefineForClassFields` будет по умолчанию установлен в `false`, что может быть проблематично с значением по умолчанию `esbuild.target` равным `esnext`. Это может транспилировать в [блоки статической инициализации](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Static_initialization_blocks#browser_compatibility), которые могут не поддерживаться в вашем браузере.

Поэтому рекомендуется установить `target` в `ESNext` или `ES2022` или новее, или явно установить `useDefineForClassFields` в `true` при настройке `tsconfig.json`.
:::

#### Другие опции компилятора, влияющие на результат сборки

- [`extends`](https://www.typescriptlang.org/tsconfig#extends)
- [`importsNotUsedAsValues`](https://www.typescriptlang.org/tsconfig#importsNotUsedAsValues)
- [`preserveValueImports`](https://www.typescriptlang.org/tsconfig#preserveValueImports)
- [`verbatimModuleSyntax`](https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax)
- [`jsx`](https://www.typescriptlang.org/tsconfig#jsx)
- [`jsxFactory`](https://www.typescriptlang.org/tsconfig#jsxFactory)
- [`jsxFragmentFactory`](https://www.typescriptlang.org/tsconfig#jsxFragmentFactory)
- [`jsxImportSource`](https://www.typescriptlang.org/tsconfig#jsxImportSource)
- [`experimentalDecorators`](https://www.typescriptlang.org/tsconfig#experimentalDecorators)
- [`alwaysStrict`](https://www.typescriptlang.org/tsconfig#alwaysStrict)

::: tip `skipLibCheck`
Шаблоны стартера Vite имеют `"skipLibCheck": "true"` по умолчанию, чтобы избежать проверки типов зависимостей, так как они могут выбрать поддержку только определенных версий и конфигураций TypeScript. Вы можете узнать больше на [vuejs/vue-cli#5688](https://github.com/vuejs/vue-cli/pull/5688).
:::

### Клиентские типы

Типы по умолчанию Vite предназначены для его Node.js API. Чтобы создать окружение для клиентского кода в приложении Vite, добавьте файл объявления `d.ts`:

```typescript
/// <reference types="vite/client" />
```

::: details Использование `compilerOptions.types`

Альтернативно, вы можете добавить `vite/client` в `compilerOptions.types` внутри `tsconfig.json`:

```json [tsconfig.json]
{
  "compilerOptions": {
    "types": ["vite/client", "some-other-global-lib"]
  }
}
```

Обратите внимание, что если указан [`compilerOptions.types`](https://www.typescriptlang.org/tsconfig#types), только эти пакеты будут включены в глобальную область видимости (вместо всех видимых пакетов "@types").

:::

`vite/client` предоставляет следующие типы:

- Импорты ресурсов (например, импорт файла `.svg`)
- Типы для [констант](./env-and-mode#env-variables), внедренных Vite в `import.meta.env`
- Типы для [HMR API](./api-hmr) в `import.meta.hot`

::: tip
Чтобы переопределить типизацию по умолчанию, добавьте файл определения типов, содержащий ваши типы. Затем добавьте ссылку на тип перед `vite/client`.

Например, чтобы сделать импорт по умолчанию `*.svg` компонентом React:

- `vite-env-override.d.ts` (файл, содержащий ваши типы):
  ```ts
  declare module '*.svg' {
    const content: React.FC<React.SVGProps<SVGElement>>
    export default content
  }
  ```
- Файл, содержащий ссылку на `vite/client`:
  ```ts
  /// <reference types="./vite-env-override.d.ts" />
  /// <reference types="vite/client" />
  ```

:::

## HTML

HTML файлы занимают [центральное место](/guide/#index-html-and-project-root) в Vite проекте, служа точками входа для вашего приложения, что делает простым создание одностраничных и [многостраничных приложений](/guide/build.html#multi-page-app).

Любые HTML файлы в корне вашего проекта могут быть напрямую доступны по их соответствующему пути директории:

- `<root>/index.html` -> `http://localhost:5173/`
- `<root>/about.html` -> `http://localhost:5173/about.html`
- `<root>/blog/index.html` -> `http://localhost:5173/blog/index.html`

Ресурсы, на которые ссылаются HTML элементы, такие как `<script type="module" src>` и `<link href>`, обрабатываются и собираются как часть приложения. Полный список поддерживаемых элементов приведен ниже:

- `<audio src>`
- `<embed src>`
- `<img src>` и `<img srcset>`
- `<image src>`
- `<input src>`
- `<link href>` и `<link imagesrcset>`
- `<object data>`
- `<script type="module" src>`
- `<source src>` и `<source srcset>`
- `<track src>`
- `<use href>` и `<use xlink:href>`
- `<video src>` и `<video poster>`
- `<meta content>`
  - Только если атрибут `name` соответствует `msapplication-tileimage`, `msapplication-square70x70logo`, `msapplication-square150x150logo`, `msapplication-wide310x150logo`, `msapplication-square310x310logo`, `msapplication-config` или `twitter:image`
  - Или только если атрибут `property` соответствует `og:image`, `og:image:url`, `og:image:secure_url`, `og:audio`, `og:audio:secure_url`, `og:video` или `og:video:secure_url`

```html {4-5,8-9}
<!doctype html>
<html>
  <head>
    <link rel="icon" href="/favicon.ico" />
    <link rel="stylesheet" href="/src/styles.css" />
  </head>
  <body>
    <img src="/src/images/logo.svg" alt="logo" />
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

Чтобы отказаться от обработки HTML на определенных элементах, вы можете добавить атрибут `vite-ignore` на элемент, что может быть полезно при ссылке на внешние ресурсы или CDN.

## Фреймворки

Все современные фреймворки поддерживают интеграции с Vite. Большинство плагинов фреймворков поддерживаются командами каждого фреймворка, за исключением официальных плагинов Vue и React для Vite, которые поддерживаются в организации vite:

- Поддержка Vue через [@vitejs/plugin-vue](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue)
- Поддержка Vue JSX через [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue-jsx)
- Поддержка React через [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/tree/main/packages/plugin-react)
- Поддержка React с использованием SWC через [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react-swc)

Проверьте [Руководство по плагинам](https://vite.dev/plugins) для получения дополнительной информации.

## JSX

`.jsx` и `.tsx` файлы также поддерживаются из коробки. Транспиляция JSX также обрабатывается через [esbuild](https://esbuild.github.io).

Ваш выбранный фреймворк уже настроит JSX из коробки (например, пользователи Vue должны использовать официальный плагин [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue-jsx), который предоставляет функции, специфичные для Vue 3, включая HMR, глобальное разрешение компонентов, директивы и слоты).

Если вы используете JSX с вашим собственным фреймворком, можно настроить пользовательские `jsxFactory` и `jsxFragment` с помощью опции [`esbuild`](/config/shared-options.md#esbuild). Например, плагин Preact будет использовать:

```js twoslash [vite.config.js]
import { defineConfig } from 'vite'

export default defineConfig({
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment',
  },
})
```

Подробнее в [документации esbuild](https://esbuild.github.io/content-types/#jsx).

Вы можете внедрить помощники JSX с помощью `jsxInject` (что является опцией только для Vite), чтобы избежать ручных импортов:

```js twoslash [vite.config.js]
import { defineConfig } from 'vite'

export default defineConfig({
  esbuild: {
    jsxInject: `import React from 'react'`,
  },
})
```

## CSS

Импорт `.css` файлов внедрит их содержимое на страницу через тег `<style>` с поддержкой HMR.

### Встраивание и перебазирование `@import`

Vite предварительно настроен для поддержки встраивания CSS `@import` через `postcss-import`. Алиасы Vite также учитываются для CSS `@import`. Кроме того, все ссылки CSS `url()`, даже если импортированные файлы находятся в разных директориях, всегда автоматически перебазируются для обеспечения правильности.

Встраивание `@import` и перебазирование URL также поддерживается для файлов Sass и Less (см. [CSS Pre-processors](#css-pre-processors)).

### PostCSS

Если проект содержит валидную конфигурацию PostCSS (любой формат, поддерживаемый [postcss-load-config](https://github.com/postcss/postcss-load-config), например, `postcss.config.js`), она будет автоматически применена ко всем импортированным CSS.

Обратите внимание, что минификация CSS будет выполняться после PostCSS и будет использовать опцию [`build.cssTarget`](/config/build-options.md#build-csstarget).

### CSS Modules

Любой CSS файл, заканчивающийся на `.module.css`, считается [файлом CSS modules](https://github.com/css-modules/css-modules). Импорт такого файла вернет соответствующий объект модуля:

```css [example.module.css]
.red {
  color: red;
}
```

```js twoslash
import 'vite/client'
// ---cut---
import classes from './example.module.css'
document.getElementById('foo').className = classes.red
```

Поведение CSS modules можно настроить через опцию [`css.modules`](/config/shared-options.md#css-modules).

Если `css.modules.localsConvention` установлен для включения camelCase локальных имен (например, `localsConvention: 'camelCaseOnly'`), вы также можете использовать именованные импорты:

```js twoslash
import 'vite/client'
// ---cut---
// .apply-color -> applyColor
import { applyColor } from './example.module.css'
document.getElementById('foo').className = applyColor
```

### CSS Pre-processors

Поскольку Vite нацелен только на современные браузеры, рекомендуется использовать нативные CSS переменные с плагинами PostCSS, которые реализуют черновики CSSWG (например, [postcss-nesting](https://github.com/csstools/postcss-plugins/tree/main/plugins/postcss-nesting)) и писать простой, соответствующий будущим стандартам CSS.

Тем не менее, Vite предоставляет встроенную поддержку для `.scss`, `.sass`, `.less`, `.styl` и `.stylus` файлов. Нет необходимости устанавливать плагины, специфичные для Vite, для них, но сам соответствующий препроцессор должен быть установлен:

```bash
# .scss и .sass
npm add -D sass-embedded # или sass

# .less
npm add -D less

# .styl и .stylus
npm add -D stylus
```

Если вы используете Vue single file components, это также автоматически включает `<style lang="sass">` и т.д.

Vite улучшает разрешение `@import` для Sass и Less, так что алиасы Vite также учитываются. Кроме того, относительные ссылки `url()` внутри импортированных файлов Sass/Less, которые находятся в разных директориях от корневого файла, также автоматически перебазируются для обеспечения правильности.

Встраивание `@import` и перебазирование URL не поддерживается для Stylus из-за ограничений его API.

Вы также можете использовать CSS modules в сочетании с препроцессорами, добавив `.module` к расширению файла, например `style.module.scss`.

### Отключение внедрения CSS на страницу

Автоматическое внедрение содержимого CSS можно отключить с помощью параметра запроса `?inline`. В этом случае обработанная CSS строка возвращается как экспорт по умолчанию модуля как обычно, но стили не внедряются на страницу.

```js twoslash
import 'vite/client'
// ---cut---
import './foo.css' // будет внедрен на страницу
import otherStyles from './bar.css?inline' // не будет внедрен
```

::: tip ПРИМЕЧАНИЕ
Экспорты по умолчанию и именованные экспорты из CSS файлов (например, `import style from './foo.css'`) удалены с Vite 5. Используйте параметр `?inline` вместо этого.
:::

### Lightning CSS

Начиная с Vite 4.4, есть экспериментальная поддержка [Lightning CSS](https://lightningcss.dev/). Вы можете включить это, добавив [`css.transformer: 'lightningcss'`](../config/shared-options.md#css-transformer) в ваш конфигурационный файл и установив опциональную зависимость [`lightningcss`](https://www.npmjs.com/package/lightningcss):

```bash
npm add -D lightningcss
```

Если включено, CSS файлы будут обрабатываться Lightning CSS вместо PostCSS. Для его настройки вы можете передать опции Lightning CSS в опцию конфигурации [`css.lightningcss`](../config/shared-options.md#css-lightningcss).

Для настройки CSS Modules вы будете использовать [`css.lightningcss.cssModules`](https://lightningcss.dev/css-modules.html) вместо [`css.modules`](../config/shared-options.md#css-modules) (который настраивает способ обработки CSS modules PostCSS).

По умолчанию Vite использует esbuild для минификации CSS. Lightning CSS также может использоваться как минификатор CSS с [`build.cssMinify: 'lightningcss'`](../config/build-options.md#build-cssminify).

::: tip ПРИМЕЧАНИЕ
[CSS Pre-processors](#css-pre-processors) не поддерживаются при использовании Lightning CSS.
:::

## Статические ресурсы

Импорт статического ресурса вернет разрешенный публичный URL при его обслуживании:

```js twoslash
import 'vite/client'
// ---cut---
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

Специальные запросы могут изменить способ загрузки ресурсов:

```js twoslash
import 'vite/client'
// ---cut---
// Явно загрузить ресурсы как URL
import assetAsURL from './asset.js?url'
```

```js twoslash
import 'vite/client'
// ---cut---
// Загрузить ресурсы как строки
import assetAsString from './shader.glsl?raw'
```

```js twoslash
import 'vite/client'
// ---cut---
// Загрузить Web Workers
import Worker from './worker.js?worker'
```

```js twoslash
import 'vite/client'
// ---cut---
// Web Workers встроены как base64 строки во время сборки
import InlineWorker from './worker.js?worker&inline'
```

Подробнее в [Обработка статических ресурсов](./assets).

## JSON

JSON файлы могут быть напрямую импортированы - именованные импорты также поддерживаются:

```js twoslash
import 'vite/client'
// ---cut---
// импортировать весь объект
import json from './example.json'
// импортировать корневое поле как именованные экспорты - помогает с tree-shaking!
import { field } from './example.json'
```

## Glob Import

Vite поддерживает импорт нескольких модулей из файловой системы через специальную функцию `import.meta.glob`:

```js twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob('./dir/*.js')
```

Вышеуказанное будет преобразовано в следующее:

```js
// код, произведенный vite
const modules = {
  './dir/bar.js': () => import('./dir/bar.js'),
  './dir/foo.js': () => import('./dir/foo.js'),
}
```

Затем вы можете итерироваться по ключам объекта `modules` для доступа к соответствующим модулям:

```js
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```

Сопоставленные файлы по умолчанию лениво загружаются через динамический импорт и будут разделены на отдельные чанки во время сборки. Если вы предпочитаете импортировать все модули напрямую (например, полагаясь на побочные эффекты в этих модулях, которые должны быть применены первыми), вы можете передать `{ eager: true }` как второй аргумент:

```js twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob('./dir/*.js', { eager: true })
```

Вышеуказанное будет преобразовано в следующее:

```js
// код, произведенный vite
import * as __vite_glob_0_0 from './dir/bar.js'
import * as __vite_glob_0_1 from './dir/foo.js'
const modules = {
  './dir/bar.js': __vite_glob_0_0,
  './dir/foo.js': __vite_glob_0_1,
}
```

### Несколько паттернов

Первый аргумент может быть массивом глобов, например

```js twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob(['./dir/*.js', './another/*.js'])
```

### Отрицательные паттерны

Также поддерживаются отрицательные глоб паттерны (с префиксом `!`). Чтобы игнорировать некоторые файлы из результата, вы можете добавить паттерны исключения в первый аргумент:

```js twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob(['./dir/*.js', '!**/bar.js'])
```

```js
// код, произведенный vite
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
}
```

#### Именованные импорты

Возможно импортировать только части модулей с опцией `import`.

```ts twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob('./dir/*.js', { import: 'setup' })
```

```ts
// код, произведенный vite
const modules = {
  './dir/bar.js': () => import('./dir/bar.js').then((m) => m.setup),
  './dir/foo.js': () => import('./dir/foo.js').then((m) => m.setup),
}
```

При комбинировании с `eager` даже возможно включить tree-shaking для этих модулей.

```ts twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob('./dir/*.js', {
  import: 'setup',
  eager: true,
})
```

```ts
// код, произведенный vite:
import { setup as __vite_glob_0_0 } from './dir/bar.js'
import { setup as __vite_glob_0_1 } from './dir/foo.js'
const modules = {
  './dir/bar.js': __vite_glob_0_0,
  './dir/foo.js': __vite_glob_0_1,
}
```

Установите `import` в `default` для импорта экспорта по умолчанию.

```ts twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob('./dir/*.js', {
  import: 'default',
  eager: true,
})
```

```ts
// код, произведенный vite:
import { default as __vite_glob_0_0 } from './dir/bar.js'
import { default as __vite_glob_0_1 } from './dir/foo.js'
const modules = {
  './dir/bar.js': __vite_glob_0_0,
  './dir/foo.js': __vite_glob_0_1,
}
```

#### Пользовательские запросы

Вы также можете использовать опцию `query` для предоставления запросов к импортам, например, для импорта ресурсов [как строки](https://vite.dev/guide/assets.html#importing-asset-as-string) или [как url](https://vite.dev/guide/assets.html#importing-asset-as-url):

```ts twoslash
import 'vite/client'
// ---cut---
const moduleStrings = import.meta.glob('./dir/*.svg', {
  query: '?raw',
  import: 'default',
})
const moduleUrls = import.meta.glob('./dir/*.svg', {
  query: '?url',
  import: 'default',
})
```

```ts
// код, произведенный vite:
const moduleStrings = {
  './dir/bar.svg': () => import('./dir/bar.svg?raw').then((m) => m['default']),
  './dir/foo.svg': () => import('./dir/foo.svg?raw').then((m) => m['default']),
}
const moduleUrls = {
  './dir/bar.svg': () => import('./dir/bar.svg?url').then((m) => m['default']),
  './dir/foo.svg': () => import('./dir/foo.svg?url').then((m) => m['default']),
}
```

Вы также можете предоставить пользовательские запросы для использования другими плагинами:

```ts twoslash
import 'vite/client'
// ---cut---
const modules = import.meta.glob('./dir/*.js', {
  query: { foo: 'bar', bar: true },
})
```

### Предостережения Glob Import

Обратите внимание, что:

- Это функция только для Vite и не является веб- или ES стандартом.
- Глоб паттерны обрабатываются как спецификаторы импорта: они должны быть либо относительными (начинаться с `./`), либо абсолютными (начинаться с `/`, разрешаются относительно корня проекта) или путем алиаса (см. опцию [`resolve.alias`](/config/shared-options.md#resolve-alias)).
- Глоб сопоставление выполняется через [`tinyglobby`](https://github.com/SuperchupuDev/tinyglobby).
- Вы также должны знать, что все аргументы в `import.meta.glob` должны быть **переданы как литералы**. Вы НЕ можете использовать переменные или выражения в них.

## Динамический импорт

Подобно [glob import](#glob-import), Vite также поддерживает динамический импорт с переменными.

```ts
const module = await import(`./dir/${file}.js`)
```

Обратите внимание, что переменные представляют только имена файлов на одном уровне глубины. Если `file` это `'foo/bar'`, импорт не удастся. Для более продвинутого использования вы можете использовать функцию [glob import](#glob-import).

## WebAssembly

Предварительно скомпилированные `.wasm` файлы могут быть импортированы с `?init`.
Экспорт по умолчанию будет функцией инициализации, которая возвращает Promise [`WebAssembly.Instance`](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Instance):

```js twoslash
import 'vite/client'
// ---cut---
import init from './example.wasm?init'

init().then((instance) => {
  instance.exports.test()
})
```

Функция init также может принимать importObject, который передается как второй аргумент в [`WebAssembly.instantiate`](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/instantiate):

```js twoslash
import 'vite/client'
import init from './example.wasm?init'
// ---cut---
init({
  imports: {
    someFunc: () => {
      /* ... */
    },
  },
}).then(() => {
  /* ... */
})
```

В production сборке `.wasm` файлы меньше `assetInlineLimit` будут встроены как base64 строки. В противном случае они будут обрабатываться как [статические ресурсы](./assets) и загружаться по требованию.

::: tip ПРИМЕЧАНИЕ
[Предложение по интеграции ES Module для WebAssembly](https://github.com/WebAssembly/esm-integration) в настоящее время не поддерживается.
Используйте [`vite-plugin-wasm`](https://github.com/Menci/vite-plugin-wasm) или другие плагины сообщества для обработки этого.
:::

### Доступ к модулю WebAssembly

Если вам нужен доступ к объекту `Module`, например, для его инстанцирования несколько раз, используйте [явный URL импорт](./assets#explicit-url-imports) для разрешения ресурса, а затем создания экземпляра:

```js twoslash
import 'vite/client'
// ---cut---
import wasmUrl from 'foo.wasm?url'

const main = async () => {
  const responsePromise = fetch(wasmUrl)
  const { module, instance } =
    await WebAssembly.instantiateStreaming(responsePromise)
  /* ... */
}

main()
```

### Получение модуля в Node.js

В SSR `fetch()`, происходящий как часть импорта `?init`, может завершиться с ошибкой `TypeError: Invalid URL`.
См. проблему [Support wasm in SSR](https://github.com/vitejs/vite/issues/8882).

Вот альтернатива, предполагая, что база проекта - текущая директория:

```js twoslash
import 'vite/client'
// ---cut---
import wasmUrl from 'foo.wasm?url'
import { readFile } from 'node:fs/promises'

const main = async () => {
  const resolvedUrl = (await import('./test/boot.test.wasm?url')).default
  const buffer = await readFile('.' + resolvedUrl)
  const { instance } = await WebAssembly.instantiate(buffer, {
    /* ... */
  })
  /* ... */
}

main()
```

## Web Workers

### Импорт с конструкторами

Скрипт веб-воркера может быть импортирован с помощью [`new Worker()`](https://developer.mozilla.org/en-US/docs/Web/API/Worker/Worker) и [`new SharedWorker()`](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker/SharedWorker). По сравнению с суффиксами воркера, этот синтаксис ближе к стандартам и является **рекомендуемым** способом создания воркеров.

```ts
const worker = new Worker(new URL('./worker.js', import.meta.url))
```

Конструктор воркера также принимает опции, которые могут быть использованы для создания "модульных" воркеров:

```ts
const worker = new Worker(new URL('./worker.js', import.meta.url), {
  type: 'module',
})
```

Обнаружение воркера будет работать только если конструктор `new URL()` используется напрямую внутри объявления `new Worker()`. Кроме того, все параметры опций должны быть статическими значениями (т.е. строковыми литералами).

### Импорт с суффиксами запросов

Скрипт веб-воркера может быть напрямую импортирован добавлением `?worker` или `?sharedworker` к запросу импорта. Экспорт по умолчанию будет пользовательским конструктором воркера:

```js twoslash
import 'vite/client'
// ---cut---
import MyWorker from './worker?worker'

const worker = new MyWorker()
```

Скрипт воркера также может использовать ESM `import` операторы вместо `importScripts()`. **Примечание**: Во время разработки это зависит от [нативной поддержки браузера](https://caniuse.com/?search=module%20worker), но для production сборки это компилируется.

По умолчанию скрипт воркера будет выдан как отдельный чанк в production сборке. Если вы хотите встроить воркер как base64 строки, добавьте запрос `inline`:

```js twoslash
import 'vite/client'
// ---cut---
import MyWorker from './worker?worker&inline'
```

Если вы хотите получить воркер как URL, добавьте запрос `url`:

```js twoslash
import 'vite/client'
// ---cut---
import MyWorker from './worker?worker&url'
```

См. [Опции воркера](/config/worker-options.md) для подробностей о настройке бандлинга всех воркеров.

## Content Security Policy (CSP)

Для развертывания CSP определенные директивы или конфигурации должны быть установлены из-за внутренней работы Vite.

### [`'nonce-{RANDOM}'`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/Sources#nonce-base64-value)

Когда установлен [`html.cspNonce`](/config/shared-options#html-cspnonce), Vite добавляет атрибут nonce с указанным значением к любым тегам `<script>` и `<style>`, а также тегам `<link>` для таблиц стилей и предзагрузки модулей. Кроме того, когда эта опция установлена, Vite внедрит мета-тег (`<meta property="csp-nonce" nonce="PLACEHOLDER" />`).

Значение nonce мета-тега с `property="csp-nonce"` будет использоваться Vite при необходимости как во время разработки, так и после сборки.

:::warning
Убедитесь, что вы заменяете заполнитель уникальным значением для каждого запроса. Это важно для предотвращения обхода политики ресурса, что в противном случае может быть легко сделано.
:::

### [`data:`](<https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/Sources#scheme-source:~:text=schemes%20(not%20recommended).-,data%3A,-Allows%20data%3A>)

По умолчанию во время сборки Vite встраивает маленькие ресурсы как data URI. Разрешение `data:` для связанных директив (например, [`img-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/img-src), [`font-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/font-src)), или, если необходимо, отключите через установку [`build.assetsInlineLimit: 0`](/config/build-options#build-assetsinlinelimit).

:::warning
Не разрешайте `data:` для [`script-src`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src). Это позволит внедрять произвольные скрипты.
:::

## Оптимизации сборки

> Функции, перечисленные ниже, автоматически применяются как часть процесса сборки, и нет необходимости в явной конфигурации, если вы не хотите их отключить.

### Разделение кода CSS

Vite автоматически извлекает CSS, используемый модулями в асинхронном чанке, и генерирует отдельный файл для него. CSS файл автоматически загружается через тег `<link>` когда загружается связанный асинхронный чанк, и асинхронный чанк гарантированно будет оценен только после загрузки CSS, чтобы избежать [FOUC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content#:~:text=A%20flash%20of%20unstyled%20content,before%20all%20information%20is%20retrieved.).

Если вы предпочитаете хранить весь CSS извлеченным в один файл, вы можете отключить разделение кода CSS, установив [`build.cssCodeSplit`](/config/build-options.md#build-csscodesplit) в `false`.

### Генерация директив предзагрузки

Vite автоматически генерирует директивы `<link rel="modulepreload">` для чанков входа и их прямых импортов в собранном HTML.

### Оптимизация загрузки асинхронных чанков

В реальных приложениях Rollup часто генерирует "общие" чанки - код, который разделяется между двумя или более другими чанками. В сочетании с динамическими импортами довольно часто встречается следующий сценарий:

<script setup>
import graphSvg from '../images/graph.svg?raw'
</script>
<svg-image :svg="graphSvg" />

В неоптимизированных сценариях, когда импортируется асинхронный чанк `A`, браузеру придется запросить и разобрать `A`, прежде чем он сможет понять, что ему также нужен общий чанк `C`. Это приводит к дополнительному сетевому обходу:

```
Entry ---> A ---> C
```

Vite автоматически переписывает вызовы динамического импорта с разделением кода с шагом предзагрузки, так что когда запрашивается `A`, `C` загружается **параллельно**:

```
Entry ---> (A + C)
```

Возможно, что `C` имеет дальнейшие импорты, что приведет к еще большему количеству обходов в неоптимизированном сценарии. Оптимизация Vite проследит все прямые импорты, чтобы полностью устранить обходы независимо от глубины импорта.
