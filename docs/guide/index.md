# Начало работы

<audio id="vite-audio">
  <source src="/vite.mp3" type="audio/mpeg">
</audio>

## Обзор

Vite (французское слово, означающее "быстрый", произносится как `/vit/`<button style="border:none;padding:3px;border-radius:4px;vertical-align:bottom" id="play-vite-audio" onclick="document.getElementById('vite-audio').play();"><svg style="height:2em;width:2em"><use href="/voice.svg#voice" /></svg></button>, как "вит") - это инструмент сборки, который стремится предоставить более быстрый и легкий опыт разработки для современных веб-проектов. Он состоит из двух основных частей:

- Dev-сервер, который предоставляет [богатые улучшения функций](./features) поверх [нативных ES модулей](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules), например, чрезвычайно быстрое [Hot Module Replacement (HMR)](./features#hot-module-replacement).

- Команда сборки, которая собирает ваш код с помощью [Rollup](https://rollupjs.org), предварительно настроенная для вывода высокооптимизированных статических ресурсов для продакшена.

Vite поставляется с разумными настройками по умолчанию "из коробки". Узнайте о возможностях в [Руководстве по функциям](./features). Поддержка фреймворков или интеграция с другими инструментами возможна через [Плагины](./using-plugins). Раздел [Конфигурация](../config/) объясняет, как адаптировать Vite к вашему проекту при необходимости.

Vite также хорошо расширяем через свой [API плагинов](./api-plugin) и [JavaScript API](./api-javascript) с полной поддержкой типов.

Вы можете узнать больше о причинах создания проекта в разделе [Почему Vite](./why).

## Поддержка браузеров

Во время разработки Vite устанавливает [`esnext` как цель трансформации](https://esbuild.github.io/api/#target), потому что мы предполагаем использование современного браузера, который поддерживает все последние функции JavaScript и CSS. Это предотвращает понижение синтаксиса, позволяя Vite обслуживать модули как можно ближе к исходному коду.

Для продакшен-сборки по умолчанию Vite нацелен на браузеры, поддерживающие современный JavaScript, такие как [нативные ES модули](https://caniuse.com/es6-module), [нативный динамический импорт ESM](https://caniuse.com/es6-module-dynamic-import), [`import.meta`](https://caniuse.com/mdn-javascript_operators_import_meta), [nullish coalescing](https://caniuse.com/mdn-javascript_operators_nullish_coalescing) и [BigInt](https://caniuse.com/bigint). Устаревшие браузеры могут поддерживаться через официальный [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy). См. раздел [Сборка для продакшена](./build) для более подробной информации.

## Попробовать Vite онлайн

Вы можете попробовать Vite онлайн на [StackBlitz](https://vite.new/). Он запускает настройку сборки на основе Vite прямо в браузере, поэтому она почти идентична локальной настройке, но не требует установки чего-либо на вашем компьютере. Вы можете перейти к `vite.new/{template}`, чтобы выбрать, какой фреймворк использовать.

Поддерживаемые предустановки шаблонов:

|             JavaScript              |                TypeScript                 |
| :---------------------------------: | :---------------------------------------: |
| [vanilla](https://vite.new/vanilla) | [vanilla-ts](https://vite.new/vanilla-ts) |
|     [vue](https://vite.new/vue)     |     [vue-ts](https://vite.new/vue-ts)     |
|   [react](https://vite.new/react)   |   [react-ts](https://vite.new/react-ts)   |
|  [preact](https://vite.new/preact)  |  [preact-ts](https://vite.new/preact-ts)  |
|     [lit](https://vite.new/lit)     |     [lit-ts](https://vite.new/lit-ts)     |
|  [svelte](https://vite.new/svelte)  |  [svelte-ts](https://vite.new/svelte-ts)  |
|   [solid](https://vite.new/solid)   |   [solid-ts](https://vite.new/solid-ts)   |
|    [qwik](https://vite.new/qwik)    |    [qwik-ts](https://vite.new/qwik-ts)    |

## Создание вашего первого проекта Vite

::: tip Примечание о совместимости
Vite требует версию [Node.js](https://nodejs.org/en/) 18+ или 20+. Однако некоторые шаблоны требуют более высокой версии Node.js для работы, пожалуйста, обновите, если ваш менеджер пакетов предупреждает об этом.
:::

::: code-group

```bash [npm]
$ npm create vite@latest
```

```bash [Yarn]
$ yarn create vite
```

```bash [pnpm]
$ pnpm create vite
```

```bash [Bun]
$ bun create vite
```

:::

Затем следуйте инструкциям!

Вы также можете напрямую указать имя проекта и шаблон, который хотите использовать, через дополнительные параметры командной строки. Например, чтобы создать проект Vite + Vue, выполните:

::: code-group

```bash [npm]
# npm 7+, нужен дополнительный двойной дефис:
$ npm create vite@latest my-vue-app -- --template vue
```

```bash [Yarn]
$ yarn create vite my-vue-app --template vue
```

```bash [pnpm]
$ pnpm create vite my-vue-app --template vue
```

```bash [Bun]
$ bun create vite my-vue-app --template vue
```

:::

См. [create-vite](https://github.com/vitejs/vite/tree/main/packages/create-vite) для более подробной информации о каждом поддерживаемом шаблоне: `vanilla`, `vanilla-ts`, `vue`, `vue-ts`, `react`, `react-ts`, `react-swc`, `react-swc-ts`, `preact`, `preact-ts`, `lit`, `lit-ts`, `svelte`, `svelte-ts`, `solid`, `solid-ts`, `qwik`, `qwik-ts`.

Вы можете использовать `.` для имени проекта, чтобы создать проект в текущей директории.

## Шаблоны сообщества

create-vite - это инструмент для быстрого запуска проекта из базового шаблона для популярных фреймворков. Проверьте Awesome Vite для [шаблонов, поддерживаемых сообществом](https://github.com/vitejs/awesome-vite#templates), которые включают другие инструменты или нацелены на разные фреймворки.

Для шаблона на `https://github.com/user/project` вы можете попробовать его онлайн, используя `https://github.stackblitz.com/user/project` (добавляя `.stackblitz` после `github` к URL проекта).

Вы также можете использовать такой инструмент, как [degit](https://github.com/Rich-Harris/degit), чтобы создать проект с одним из шаблонов. Предполагая, что проект находится на GitHub и использует `main` как ветку по умолчанию, вы можете создать локальную копию, используя:

```bash
npx degit user/project#main my-project
cd my-project

npm install
npm run dev
```

## Ручная установка

В вашем проекте вы можете установить CLI `vite`, используя:

::: code-group

```bash [npm]
$ npm install -D vite
```

```bash [Yarn]
$ yarn add -D vite
```

```bash [pnpm]
$ pnpm add -D vite
```

```bash [Bun]
$ bun add -D vite
```

:::

И создать файл `index.html` следующим образом:

```html
<p>Hello Vite!</p>
```

Затем запустите соответствующую команду CLI в вашем терминале:

::: code-group

```bash [npm]
$ npx vite
```

```bash [Yarn]
$ yarn vite
```

```bash [pnpm]
$ pnpm vite
```

```bash [Bun]
$ bunx vite
```

:::

`index.html` будет доступен по адресу `http://localhost:5173`.

## `index.html` и корень проекта

Одна вещь, которую вы могли заметить, это то, что в проекте Vite `index.html` находится в корне проекта, вместо того чтобы быть спрятанным внутри `public`. Это намеренно: во время разработки Vite является сервером, и `index.html` - это точка входа в ваше приложение.

Vite обрабатывает `index.html` как исходный код и часть графа модулей. Он разрешает `<script type="module" src="...">`, который ссылается на ваш JavaScript исходный код. Даже встроенные `<script type="module">` и CSS, на которые ссылаются через `<link href>`, также пользуются специальными функциями Vite. Кроме того, URL-адреса внутри `index.html` преобразуются автоматически, поэтому нет необходимости в специальных заполнителях `%PUBLIC_URL%`.

Подобно статическим http-серверам, Vite имеет концепцию "корневой директории", из которой обслуживаются ваши файлы. Вы увидите, что она упоминается как `<root>` в остальной части документации. Абсолютные URL-адреса в вашем исходном коде будут разрешаться с использованием корня проекта как базы, поэтому вы можете писать код так, как если бы вы работали с обычным статическим файловым сервером (только намного более мощным!). Vite также способен обрабатывать зависимости, которые разрешаются в файловые системы вне корня, что делает его пригодным даже в настройке на основе монорепозитория.

Vite также поддерживает [многостраничные приложения](./build#multi-page-app) с несколькими точками входа `.html`.

#### Указание альтернативного корня

Запуск `vite` запускает dev-сервер, используя текущую рабочую директорию как корень. Вы можете указать альтернативный корень с помощью `vite serve some/sub/dir`.
Обратите внимание, что Vite также будет разрешать [свой конфигурационный файл (т.е. `vite.config.js`)](/config/#configuring-vite) внутри корня проекта, поэтому вам нужно будет переместить его, если корень изменен.

## Интерфейс командной строки

В проекте, где установлен Vite, вы можете использовать бинарный файл `vite` в ваших npm-скриптах или запускать его напрямую с помощью `npx vite`. Вот стандартные npm-скрипты в созданном проекте Vite:

<!-- prettier-ignore -->
```json [package.json]
{
  "scripts": {
    "dev": "vite", // запуск dev-сервера, псевдонимы: `vite dev`, `vite serve`
    "build": "vite build", // сборка для продакшена
    "preview": "vite preview" // локальный предпросмотр продакшен-сборки
  }
}
```

Вы можете указать дополнительные параметры CLI, такие как `--port` или `--open`. Для полного списка параметров CLI выполните `npx vite --help` в вашем проекте.

Узнайте больше об [Интерфейсе командной строки](./cli.md)

## Использование невыпущенных коммитов

Если вы не можете дождаться нового релиза, чтобы протестировать последние функции, вы можете установить конкретный коммит Vite с помощью https://pkg.pr.new:

::: code-group

```bash [npm]
$ npm install -D https://pkg.pr.new/vite@SHA
```

```bash [Yarn]
$ yarn add -D https://pkg.pr.new/vite@SHA
```

```bash [pnpm]
$ pnpm add -D https://pkg.pr.new/vite@SHA
```

```bash [Bun]
$ bun add -D https://pkg.pr.new/vite@SHA
```

:::

Замените `SHA` на любой из [коммитов Vite](https://github.com/vitejs/vite/commits/main/). Обратите внимание, что будут работать только коммиты за последний месяц, так как более старые релизы коммитов удаляются.

Альтернативно, вы также можете клонировать [репозиторий vite](https://github.com/vitejs/vite) на ваш локальный компьютер, а затем собрать и связать его самостоятельно (требуется [pnpm](https://pnpm.io/)):

```bash
git clone https://github.com/vitejs/vite.git
cd vite
pnpm install
cd packages/vite
pnpm run build
pnpm link --global # используйте ваш предпочтительный менеджер пакетов для этого шага
```

Затем перейдите в ваш проект на основе Vite и выполните `pnpm link --global vite` (или менеджер пакетов, который вы использовали для глобальной связи `vite`). Теперь перезапустите dev-сервер, чтобы использовать самые последние изменения!

::: tip Зависимости, использующие Vite
Чтобы заменить версию Vite, используемую зависимостями транзитивно, вы должны использовать [переопределения npm](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#overrides) или [переопределения pnpm](https://pnpm.io/package_json#pnpmoverrides).
:::

## Сообщество

Если у вас есть вопросы или нужна помощь, обратитесь к сообществу на [Discord](https://chat.vite.dev) и [GitHub Discussions](https://github.com/vitejs/vite/discussions).
