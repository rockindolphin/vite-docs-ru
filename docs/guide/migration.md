# Миграция с v5

## API окружений

В рамках нового экспериментального [API окружений](/guide/api-environment.md), потребовалась большая внутренняя переработка. Vite 6 стремится избежать разрушающих изменений, чтобы большинство проектов могли быстро обновиться до новой мажорной версии. Мы будем ждать, пока большая часть экосистемы перейдет на новую версию, чтобы стабилизировать и начать рекомендовать использование новых API. Могут быть некоторые крайние случаи, но они должны затрагивать только низкоуровневое использование фреймворками и инструментами. Мы работали с поддерживающими экосистему, чтобы смягчить эти различия перед выпуском. Пожалуйста, [откройте проблему](https://github.com/vitejs/vite/issues/new?assignees=&labels=pending+triage&projects=&template=bug_report.yml), если вы заметите регрессию.

Некоторые внутренние API были удалены из-за изменений в реализации Vite. Если вы полагались на один из них, пожалуйста, создайте [запрос на функцию](https://github.com/vitejs/vite/issues/new?assignees=&labels=enhancement%3A+pending+triage&projects=&template=feature_request.yml).

## API времени выполнения Vite

Экспериментальный API времени выполнения Vite эволюционировал в API модуля Runner, выпущенный в Vite 6 как часть нового экспериментального [API окружений](/guide/api-environment). Поскольку функция была экспериментальной, удаление предыдущего API, введенного в Vite 5.1, не является разрушающим изменением, но пользователям потребуется обновить свое использование на эквивалент модуля Runner в процессе миграции на Vite 6.

## Общие изменения

### Значение по умолчанию для `resolve.conditions`

Это изменение не затрагивает пользователей, которые не настраивали [`resolve.conditions`](/config/shared-options#resolve-conditions) / [`ssr.resolve.conditions`](/config/ssr-options#ssr-resolve-conditions) / [`ssr.resolve.externalConditions`](/config/ssr-options#ssr-resolve-externalconditions).

В Vite 5 значение по умолчанию для `resolve.conditions` было `[]`, и некоторые условия добавлялись внутренне. Значение по умолчанию для `ssr.resolve.conditions` было значением `resolve.conditions`.

С Vite 6 некоторые условия больше не добавляются внутренне и должны быть включены в значения конфигурации.
Условия, которые больше не добавляются внутренне для

- `resolve.conditions` - это `['module', 'browser', 'development|production']`
- `ssr.resolve.conditions` - это `['module', 'node', 'development|production']`

Значения по умолчанию для этих опций обновлены до соответствующих значений, и `ssr.resolve.conditions` больше не использует `resolve.conditions` в качестве значения по умолчанию. Обратите внимание, что `development|production` является специальной переменной, которая заменяется на `production` или `development` в зависимости от значения `process.env.NODE_ENV`. Эти значения по умолчанию экспортируются из `vite` как `defaultClientConditions` и `defaultServerConditions`.

Если вы указали пользовательское значение для `resolve.conditions` или `ssr.resolve.conditions`, вам нужно обновить его, чтобы включить новые условия.
Например, если вы ранее указали `['custom']` для `resolve.conditions`, вам нужно указать `['custom', ...defaultClientConditions]` вместо этого.

### JSON stringify

В Vite 5, когда [`json.stringify: true`](/config/shared-options#json-stringify) установлен, [`json.namedExports`](/config/shared-options#json-namedexports) был отключен.

С Vite 6, даже когда `json.stringify: true` установлен, `json.namedExports` не отключается, и значение учитывается. Если вы хотите достичь предыдущего поведения, вы можете установить `json.namedExports: false`.

Vite 6 также вводит новое значение по умолчанию для `json.stringify`, которое равно `'auto'`, что будет сериализовать только большие JSON-файлы. Чтобы отключить это поведение, установите `json.stringify: false`.

### Расширенная поддержка ссылок на ресурсы в HTML-элементах

В Vite 5 только несколько поддерживаемых HTML-элементов могли ссылаться на ресурсы, которые будут обрабатываться и упаковываться Vite, такие как `<link href>`, `<img src>`, и т.д.

Vite 6 расширяет поддержку даже для большего количества HTML-элементов. Полный список можно найти в документации [HTML-функций](/guide/features.html#html).

Чтобы отказаться от обработки HTML для определенных элементов, вы можете добавить атрибут `vite-ignore` на элемент.

### postcss-load-config

[`postcss-load-config`](https://npmjs.com/package/postcss-load-config) был обновлен до v6 с v4. [`tsx`](https://www.npmjs.com/package/tsx) или [`jiti`](https://www.npmjs.com/package/jiti) теперь требуется для загрузки файлов конфигурации postcss на TypeScript вместо [`ts-node`](https://www.npmjs.com/package/ts-node). Также [`yaml`](https://www.npmjs.com/package/yaml) теперь требуется для загрузки файлов конфигурации postcss на YAML.

### Sass теперь использует современный API по умолчанию

В Vite 5 по умолчанию использовался устаревший API для Sass. Vite 5.4 добавил поддержку современного API.

С Vite 6 современный API используется по умолчанию для Sass. Если вы хотите по-прежнему использовать устаревший API, вы можете установить [`css.preprocessorOptions.sass.api: 'legacy' / css.preprocessorOptions.scss.api: 'legacy'`](/config/shared-options#css-preprocessoroptions). Но обратите внимание, что поддержка устаревшего API будет удалена в Vite 7.

Чтобы перейти на современный API, смотрите [документацию по Sass](https://sass-lang.com/documentation/breaking-changes/legacy-js-api/).

### Настройка имени выходного файла CSS в режиме библиотеки

В Vite 5 имя выходного файла CSS в режиме библиотеки всегда было `style.css` и не могло быть легко изменено через конфигурацию Vite.

С Vite 6 имя по умолчанию теперь использует `"name"` в `package.json`, аналогично JS выходным файлам. Если [`build.lib.fileName`](/config/build-options.md#build-lib) установлено со строкой, это значение также будет использоваться для имени выходного файла CSS. Чтобы явно установить другое имя файла CSS, вы можете использовать новый [`build.lib.cssFileName`](/config/build-options.md#build-lib) для его настройки.

Чтобы перейти, если вы полагались на имя файла `style.css`, вам следует обновить ссылки на него на новое имя на основе имени вашего пакета. Например:

```json [package.json]
{
  "name": "my-lib",
  "exports": {
    "./style.css": "./dist/style.css" // [!code --]
    "./style.css": "./dist/my-lib.css" // [!code ++]
  }
}
```

Если вы предпочитаете остаться с `style.css`, как в Vite 5, вы можете установить `build.lib.cssFileName: 'style'` вместо этого.

## Расширенные

Существуют и другие разрушающие изменения, которые затрагивают лишь немногих пользователей.

- [[#17922] fix(css)!: удалить импорт по умолчанию в ssr dev](https://github.com/vitejs/vite/pull/17922)
  - Поддержка импорта по умолчанию файлов CSS была [устаревшей в Vite 4](https://v4.vite.dev/guide/migration.html#importing-css-as-a-string) и удалена в Vite 5, но она все еще непреднамеренно поддерживалась в режиме разработки SSR. Эта поддержка теперь удалена.
- [[#15637] fix!: по умолчанию `build.cssMinify` установить в `'esbuild'` для SSR](https://github.com/vitejs/vite/pull/15637)
  - [`build.cssMinify`](/config/build-options#build-cssminify) теперь включен по умолчанию даже для сборок SSR.
- [[#18070] feat!: обход прокси с помощью WebSocket](https://github.com/vitejs/vite/pull/18070)
  - `server.proxy[path].bypass` теперь вызывается для запросов обновления WebSocket, и в этом случае параметр `res` будет `undefined`.
- [[#18209] refactor!: увеличить минимальную версию terser до 5.16.0](https://github.com/vitejs/vite/pull/18209)
  - Минимально поддерживаемая версия terser для [`build.minify: 'terser'`](/config/build-options#build-minify) была увеличена до 5.16.0 с 5.4.0.
- [[#18231] chore(deps): обновить зависимость @rollup/plugin-commonjs до v28](https://github.com/vitejs/vite/pull/18231)
  - [`commonjsOptions.strictRequires`](https://github.com/rollup/plugins/blob/master/packages/commonjs/README.md#strictrequires) теперь по умолчанию `true` (было `'auto'` ранее).
    - Это может привести к увеличению размера бандла, но обеспечит более детерминированные сборки.
    - Если вы указываете файл CommonJS в качестве точки входа, вам могут потребоваться дополнительные шаги. Читайте документацию плагина commonjs для получения дополнительных сведений.
- [[#18243] chore(deps)!: миграция `fast-glob` на `tinyglobby`](https://github.com/vitejs/vite/pull/18243)
  - Диафрагмы диапазонов (`{01..03}` ⇒ `['01', '02', '03']`) и инкрементальные диафрагмы (`{2..8..2}` ⇒ `['2', '4', '6', '8']`) больше не поддерживаются в globах.
- [[#18395] feat(resolve)!: разрешить удаление условий](https://github.com/vitejs/vite/pull/18395)
  - Этот PR не только вводит разрушающее изменение, упомянутое выше как "Значение по умолчанию для `resolve.conditions`", но также делает так, что `resolve.mainFields` не используется для неэкстернализованных зависимостей в SSR. Если вы использовали `resolve.mainFields` и хотите применить это к неэкстернализованным зависимостям в SSR, вы можете использовать [`ssr.resolve.mainFields`](/config/ssr-options#ssr-resolve-mainfields).
- [[#18493] refactor!: удалить опцию fs.cachedChecks](https://github.com/vitejs/vite/pull/18493)
  - Эта опция оптимизации по желанию была удалена из-за крайних случаев, когда файл записывается в кэшированную папку и немедленно импортируется.
- ~~[[#18697] fix(deps)!: обновить зависимость dotenv-expand до v12](https://github.com/vitejs/vite/pull/18697)~~
  - ~~Переменные, используемые в интерполяции, должны быть объявлены перед интерполяцией. Для получения дополнительных сведений смотрите [журнал изменений `dotenv-expand`](https://github.com/motdotla/dotenv-expand/blob/v12.0.1/CHANGELOG.md#1200-2024-11-16).~~ Это разрушающее изменение было отменено в v6.1.0.
- [[#16471] feat: v6 - API окружений](https://github.com/vitejs/vite/pull/16471)

  - Обновления к модулю, предназначенному только для SSR, больше не вызывают полную перезагрузку страницы на клиенте. Чтобы вернуть предыдущее поведение, можно использовать пользовательский плагин Vite:
    <details>
    <summary>Нажмите, чтобы развернуть пример</summary>

    ```ts twoslash
    import type { Plugin, EnvironmentModuleNode } from 'vite'

    function hmrReload(): Plugin {
      return {
        name: 'hmr-reload',
        enforce: 'post',
        hotUpdate: {
          order: 'post',
          handler({ modules, server, timestamp }) {
            if (this.environment.name !== 'ssr') return

            let hasSsrOnlyModules = false

            const invalidatedModules = new Set<EnvironmentModuleNode>()
            for (const mod of modules) {
              if (mod.id == null) continue
              const clientModule =
                server.environments.client.moduleGraph.getModuleById(mod.id)
              if (clientModule != null) continue

              this.environment.moduleGraph.invalidateModule(
                mod,
                invalidatedModules,
                timestamp,
                true,
              )
              hasSsrOnlyModules = true
            }

            if (hasSsrOnlyModules) {
              server.ws.send({ type: 'full-reload' })
              return []
            }
          },
        },
      }
    }
    ```

    </details>

## Миграция с v4

Сначала проверьте [Руководство по миграции с v4](https://v5.vite.dev/guide/migration.html) в документации Vite v5, чтобы увидеть необходимые изменения для переноса вашего приложения на Vite 5, а затем перейдите к изменениям на этой странице.
