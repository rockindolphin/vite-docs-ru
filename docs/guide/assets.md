# Обработка статических ресурсов

- См. также: [Базовый публичный путь](./build#public-base-path)
- См. также: [Опция конфигурации `assetsInclude`](/config/shared-options.md#assetsinclude)

## Импорт ресурса как URL

Импорт статического ресурса вернет разрешенный публичный URL при его обслуживании:

```js twoslash
import 'vite/client'
// ---cut---
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

Например, `imgUrl` будет `/src/img.png` во время разработки и станет `/assets/img.2d8efhg.png` в production-сборке.

Поведение похоже на `file-loader` webpack. Разница в том, что импорт может использовать абсолютные публичные пути (на основе корня проекта во время разработки) или относительные пути.

- Ссылки `url()` в CSS обрабатываются так же.

- При использовании плагина Vue ссылки на ресурсы в шаблонах Vue SFC автоматически преобразуются в импорты.

- Общие типы файлов изображений, медиа и шрифтов автоматически определяются как ресурсы. Вы можете расширить внутренний список с помощью [опции `assetsInclude`](/config/shared-options.md#assetsinclude).

- Указанные ресурсы включаются как часть графа ресурсов сборки, получают хэшированные имена файлов и могут обрабатываться плагинами для оптимизации.

- Ресурсы, размер которых в байтах меньше [опции `assetsInlineLimit`](/config/build-options.md#build-assetsinlinelimit), будут встроены как base64 data URLs.

- Плейсхолдеры Git LFS автоматически исключаются из встраивания, так как они не содержат содержимого файла, который они представляют. Для получения встраивания убедитесь, что вы загрузили содержимое файла через Git LFS перед сборкой.

- TypeScript по умолчанию не распознает импорты статических ресурсов как действительные модули. Чтобы исправить это, включите [`vite/client`](./features#client-types).

::: tip Встраивание SVG через `url()`
При передаче URL SVG в вручную сконструированный `url()` через JS, переменная должна быть обернута в двойные кавычки.

```js twoslash
import 'vite/client'
// ---cut---
import imgUrl from './img.svg'
document.getElementById('hero-img').style.background = `url("${imgUrl}")`
```

:::

### Явный импорт URL

Ресурсы, которые не включены во внутренний список или в `assetsInclude`, могут быть явно импортированы как URL с использованием суффикса `?url`. Это полезно, например, для импорта [Houdini Paint Worklets](https://developer.mozilla.org/en-US/docs/Web/API/CSS/paintWorklet_static).

```js twoslash
import 'vite/client'
// ---cut---
import workletURL from 'extra-scalloped-border/worklet.js?url'
CSS.paintWorklet.addModule(workletURL)
```

### Явная обработка встраивания

Ресурсы могут быть явно импортированы со встраиванием или без встраивания с использованием суффикса `?inline` или `?no-inline` соответственно.

```js twoslash
import 'vite/client'
// ---cut---
import imgUrl1 from './img.svg?no-inline'
import imgUrl2 from './img.png?inline'
```

### Импорт ресурса как строки

Ресурсы могут быть импортированы как строки с использованием суффикса `?raw`.

```js twoslash
import 'vite/client'
// ---cut---
import shaderString from './shader.glsl?raw'
```

### Импорт скрипта как Worker

Скрипты могут быть импортированы как веб-воркеры с суффиксом `?worker` или `?sharedworker`.

```js twoslash
import 'vite/client'
// ---cut---
// Отдельный чанк в production-сборке
import Worker from './shader.js?worker'
const worker = new Worker()
```

```js twoslash
import 'vite/client'
// ---cut---
// sharedworker
import SharedWorker from './shader.js?sharedworker'
const sharedWorker = new SharedWorker()
```

```js twoslash
import 'vite/client'
// ---cut---
// Встроено как base64 строки
import InlineWorker from './shader.js?worker&inline'
```

Подробнее см. раздел [Web Worker](./features.md#web-workers).

## Директория `public`

Если у вас есть ресурсы, которые:

- Никогда не используются в исходном коде (например, `robots.txt`)
- Должны сохранять точно такое же имя файла (без хэширования)
- ...или вы просто не хотите сначала импортировать ресурс, чтобы получить его URL

Тогда вы можете поместить ресурс в специальную директорию `public` под корнем вашего проекта. Ресурсы в этой директории будут обслуживаться по корневому пути `/` во время разработки и копироваться в корень директории dist как есть.

Директория по умолчанию `<root>/public`, но может быть настроена через [опцию `publicDir`](/config/shared-options.md#publicdir).

Обратите внимание, что вы всегда должны ссылаться на ресурсы `public` используя абсолютный корневой путь - например, `public/icon.png` должен быть указан в исходном коде как `/icon.png`.

## new URL(url, import.meta.url)

[import.meta.url](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta) - это нативная функция ESM, которая раскрывает URL текущего модуля. В сочетании с нативным [конструктором URL](https://developer.mozilla.org/en-US/docs/Web/API/URL), мы можем получить полный, разрешенный URL статического ресурса, используя относительный путь из JavaScript модуля:

```js
const imgUrl = new URL('./img.png', import.meta.url).href

document.getElementById('hero-img').src = imgUrl
```

Это работает нативно в современных браузерах - фактически, Vite не нужно обрабатывать этот код во время разработки!

Этот паттерн также поддерживает динамические URL через шаблонные литералы:

```js
function getImageUrl(name) {
  // обратите внимание, что это не включает файлы в поддиректориях
  return new URL(`./dir/${name}.png`, import.meta.url).href
}
```

Во время production-сборки Vite выполнит необходимые преобразования, чтобы URL все еще указывали на правильное местоположение даже после бандлинга и хэширования ресурсов. Однако строка URL должна быть статической, чтобы ее можно было проанализировать, иначе код останется как есть, что может вызвать ошибки во время выполнения, если `build.target` не поддерживает `import.meta.url`

```js
// Vite не будет преобразовывать это
const imgUrl = new URL(imagePath, import.meta.url).href
```

::: details Как это работает

Vite преобразует функцию `getImageUrl` в:

```js
import __img0png from './dir/img0.png'
import __img1png from './dir/img1.png'

function getImageUrl(name) {
  const modules = {
    './dir/img0.png': __img0png,
    './dir/img1.png': __img1png,
  }
  return new URL(modules[`./dir/${name}.png`], import.meta.url).href
}
```

:::

::: warning Не работает с SSR
Этот паттерн не работает, если вы используете Vite для Server-Side Rendering, потому что `import.meta.url` имеет разную семантику в браузерах и Node.js. Серверный бандл также не может определить URL хоста клиента заранее.
:::
