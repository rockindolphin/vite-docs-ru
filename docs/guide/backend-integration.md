# Интеграция с бэкендом

:::tip Примечание
Если вы хотите обслуживать HTML с помощью традиционного бэкенда (например, Rails, Laravel), но использовать Vite для обслуживания ассетов, проверьте существующие интеграции, перечисленные в [Awesome Vite](https://github.com/vitejs/awesome-vite#integrations-with-backends).

Если вам нужна пользовательская интеграция, вы можете следовать шагам в этом руководстве для ручной настройки
:::

1. В вашей конфигурации Vite настройте точку входа и включите манифест сборки:

   ```js twoslash [vite.config.js]
   import { defineConfig } from 'vite'
   // ---cut---
   export default defineConfig({
     server: {
       cors: {
         // происхождение, через которое вы будете получать доступ через браузер
         origin: 'http://my-backend.example.com',
       },
     },
     build: {
       // генерировать .vite/manifest.json в outDir
       manifest: true,
       rollupOptions: {
         // перезаписать точку входа .html по умолчанию
         input: '/path/to/main.js',
       },
     },
   })
   ```

   Если вы не отключили [полифилл предзагрузки модулей](/config/build-options.md#build-polyfillmodulepreload), вам также нужно импортировать полифилл в вашей точке входа

   ```js
   // добавьте в начало точки входа вашего приложения
   import 'vite/modulepreload-polyfill'
   ```

2. Для разработки внедрите следующее в HTML-шаблон вашего сервера (замените `http://localhost:5173` на локальный URL, по которому работает Vite):

   ```html
   <!-- если разработка -->
   <script type="module" src="http://localhost:5173/@vite/client"></script>
   <script type="module" src="http://localhost:5173/main.js"></script>
   ```

   Для правильного обслуживания ассетов у вас есть два варианта:

   - Убедитесь, что сервер настроен на проксирование запросов статических ассетов на сервер Vite
   - Установите [`server.origin`](/config/server-options.md#server-origin), чтобы сгенерированные URL ассетов разрешались с использованием URL бэкенд-сервера вместо относительного пути

   Это необходимо для правильной загрузки ассетов, таких как изображения.

   Примечание: если вы используете React с `@vitejs/plugin-react`, вам также нужно добавить это перед скриптами выше, так как плагин не может модифицировать HTML, который вы обслуживаете (замените `http://localhost:5173` на локальный URL, по которому работает Vite):

   ```html
   <script type="module">
     import RefreshRuntime from 'http://localhost:5173/@react-refresh'
     RefreshRuntime.injectIntoGlobalHook(window)
     window.$RefreshReg$ = () => {}
     window.$RefreshSig$ = () => (type) => type
     window.__vite_plugin_react_preamble_installed__ = true
   </script>
   ```

3. Для продакшена: после запуска `vite build` файл `.vite/manifest.json` будет сгенерирован вместе с другими файлами ассетов. Пример файла манифеста выглядит так:

   ```json [.vite/manifest.json]
   {
     "_shared-B7PI925R.js": {
       "file": "assets/shared-B7PI925R.js",
       "name": "shared",
       "css": ["assets/shared-ChJ_j-JJ.css"]
     },
     "_shared-ChJ_j-JJ.css": {
       "file": "assets/shared-ChJ_j-JJ.css",
       "src": "_shared-ChJ_j-JJ.css"
     },
     "baz.js": {
       "file": "assets/baz-B2H3sXNv.js",
       "name": "baz",
       "src": "baz.js",
       "isDynamicEntry": true
     },
     "views/bar.js": {
       "file": "assets/bar-gkvgaI9m.js",
       "name": "bar",
       "src": "views/bar.js",
       "isEntry": true,
       "imports": ["_shared-B7PI925R.js"],
       "dynamicImports": ["baz.js"]
     },
     "views/foo.js": {
       "file": "assets/foo-BRBmoGS9.js",
       "name": "foo",
       "src": "views/foo.js",
       "isEntry": true,
       "imports": ["_shared-B7PI925R.js"],
       "css": ["assets/foo-5UjPuW-k.css"]
     }
   }
   ```

   - Манифест имеет структуру `Record<name, chunk>`
   - Для чанков точки входа или динамической точки входа ключом является относительный путь src от корня проекта.
   - Для не-точек входа чанков ключом является базовое имя сгенерированного файла с префиксом `_`.
   - Для CSS файла, сгенерированного когда [`build.cssCodeSplit`](/config/build-options.md#build-csscodesplit) равен `false`, ключом является `style.css`.
   - Чанки будут содержать информацию о его статических и динамических импортах (оба являются ключами, которые отображаются на соответствующий чанк в манифесте), а также его соответствующие CSS и файлы ассетов (если есть).

4. Вы можете использовать этот файл для рендеринга ссылок или директив предзагрузки с хэшированными именами файлов.

   Вот пример HTML-шаблона для рендеринга правильных ссылок. Синтаксис здесь только для объяснения, замените на язык шаблонизации вашего сервера. Функция `importedChunks` для иллюстрации и не предоставляется Vite.

   ```html
   <!-- если продакшен -->

   <!-- для cssFile из manifest[name].css -->
   <link rel="stylesheet" href="/{{ cssFile }}" />

   <!-- для chunk из importedChunks(manifest, name) -->
   <!-- для cssFile из chunk.css -->
   <link rel="stylesheet" href="/{{ cssFile }}" />

   <script type="module" src="/{{ manifest[name].file }}"></script>

   <!-- для chunk из importedChunks(manifest, name) -->
   <link rel="modulepreload" href="/{{ chunk.file }}" />
   ```

   В частности, бэкенд, генерирующий HTML, должен включать следующие теги, учитывая файл манифеста и точку входа:

   - Тег `<link rel="stylesheet">` для каждого файла в списке `css` чанка точки входа
   - Рекурсивно следуйте всем чанкам в списке `imports` точки входа и включите тег `<link rel="stylesheet">` для каждого CSS файла каждого импортированного чанка.
   - Тег для ключа `file` чанка точки входа (`<script type="module">` для JavaScript, или `<link rel="stylesheet">` для CSS)
   - Опционально, тег `<link rel="modulepreload">` для `file` каждого импортированного JavaScript чанка, снова рекурсивно следуя импортам, начиная с чанка точки входа.

   Следуя приведенному выше примеру манифеста, для точки входа `views/foo.js` должны быть включены следующие теги в продакшене:

   ```html
   <link rel="stylesheet" href="assets/foo-5UjPuW-k.css" />
   <link rel="stylesheet" href="assets/shared-ChJ_j-JJ.css" />
   <script type="module" src="assets/foo-BRBmoGS9.js"></script>
   <!-- опционально -->
   <link rel="modulepreload" href="assets/shared-B7PI925R.js" />
   ```

   В то время как для точки входа `views/bar.js` должны быть включены:

   ```html
   <link rel="stylesheet" href="assets/shared-ChJ_j-JJ.css" />
   <script type="module" src="assets/bar-gkvgaI9m.js"></script>
   <!-- опционально -->
   <link rel="modulepreload" href="assets/shared-B7PI925R.js" />
   ```

   ::: details Псевдо-реализация `importedChunks`
   Пример псевдо-реализации `importedChunks` на TypeScript (Это нужно будет адаптировать под ваш язык программирования и язык шаблонизации):

   ```ts
   import type { Manifest, ManifestChunk } from 'vite'

   export default function importedChunks(
     manifest: Manifest,
     name: string,
   ): ManifestChunk[] {
     const seen = new Set<string>()

     function getImportedChunks(chunk: ManifestChunk): ManifestChunk[] {
       const chunks: ManifestChunk[] = []
       for (const file of chunk.imports ?? []) {
         const importee = manifest[file]
         if (seen.has(file)) {
           continue
         }
         seen.add(file)

         chunks.push(...getImportedChunks(importee))
         chunks.push(importee)
       }

       return chunks
     }

     return getImportedChunks(manifest[name])
   }
   ```

   :::
