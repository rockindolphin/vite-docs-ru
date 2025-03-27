# Руководство по участию в разработке Vite

Привет! Мы очень рады, что вы хотите внести свой вклад в Vite! Прежде чем отправить свой вклад, пожалуйста, прочитайте следующее руководство. Мы также рекомендуем вам прочитать [философию проекта](https://vite.dev/guide/philosophy) в нашей документации.

## Настройка репозитория

Репозиторий Vite - это монорепозиторий, использующий pnpm workspaces. Менеджер пакетов, используемый для установки и связывания зависимостей, должен быть [pnpm](https://pnpm.io/).

Для разработки и тестирования основного пакета `vite`:

1. Выполните `pnpm i` в корневой директории Vite.

2. Выполните `pnpm run build` в корневой директории Vite.

3. Если вы разрабатываете сам Vite, вы можете перейти в `packages/vite` и выполнить `pnpm run dev`, чтобы Vite автоматически пересобирался при изменении его кода.

Альтернативно, вы можете использовать [Vite.js Docker Dev](https://github.com/nystudio107/vitejs-docker-dev) для получения контейнеризированной Docker-среды для разработки Vite.js.

> Vite использует pnpm v7. Если вы работаете над несколькими проектами с разными версиями pnpm, рекомендуется активировать [Corepack](https://github.com/nodejs/corepack), выполнив `corepack enable`.

## Отладка

Если вы хотите использовать точки останова и исследовать выполнение кода, вы можете использовать функцию ["Запуск и отладка"](https://code.visualstudio.com/docs/editor/debugging) VS Code.

1. Вставьте оператор `debugger` там, где вы хотите остановить выполнение кода.

2. Нажмите на значок "Запуск и отладка" на панели действий редактора.

3. Нажмите на кнопку "JavaScript Debug Terminal".

4. Откроется терминал. Затем перейдите в `playground/xxx` и выполните `pnpm run dev`.

5. Выполнение остановится, и вы можете использовать [панель отладки](https://code.visualstudio.com/docs/editor/debugging#_debug-actions) для продолжения, пропуска, перезапуска процесса и т.д.

### Отладка ошибок в тестах Vitest с Playwright (Chromium)

Некоторые ошибки маскируются и скрываются из-за уровней абстракции и песочничной природы Vitest, Playwright и Chromium. Чтобы выяснить, что на самом деле идет не так и увидеть содержимое консоли DevTools в таких случаях, следуйте этому руководству:

1. Добавьте оператор `debugger` в хук `afterAll` в `playground/vitestSetup.ts`. Это остановит выполнение перед завершением тестов и закрытием экземпляра браузера Playwright.

2. Запустите тесты с помощью команды `debug-serve`, которая включает удаленную отладку: `pnpm run debug-serve resolve`.

3. Дождитесь, пока откроются DevTools инспектора в вашем браузере и подключится отладчик.

4. Нажмите кнопку воспроизведения в окне исходного кода в правой панели, чтобы продолжить выполнение и запустить тесты, что откроет экземпляр Chromium.

5. Сфокусируйтесь на экземпляре Chromium, откройте DevTools браузера и проверьте консоль там, чтобы найти основные проблемы.

6. Для закрытия просто завершите процесс тестирования в вашем терминале.

## Тестирование Vite с внешними пакетами

Возможно, вы захотите протестировать вашу локально измененную копию Vite с другим пакетом, созданным с помощью Vite. В pnpm после создания Vite вы можете использовать [`pnpm.overrides`](https://pnpm.io/package_json#pnpmoverrides). Обратите внимание, что `pnpm.overrides` должен быть указан в корневом `package.json`, и вы должны сначала перечислить пакет как зависимость в корневом `package.json`:

```json
{
  "dependencies": {
    "vite": "^2.0.0"
  },
  "pnpm": {
    "overrides": {
      "vite": "link:../path/to/vite/packages/vite"
    }
  }
}
```

И выполните `pnpm install` снова, чтобы связать пакет.

## Запуск тестов

### Интеграционные тесты

Каждый пакет под `playground/` содержит директорию `__tests__`. Тесты выполняются с помощью [Vitest](https://vitest.dev/) + [Playwright](https://playwright.dev/) с использованием пользовательских интеграций для упрощения написания тестов. Подробная настройка находится в файлах `vitest.config.e2e.js` и `playground/vitest*`.

Перед запуском тестов убедитесь, что [Vite собран](#repo-setup). В Windows может потребоваться [включить режим разработчика](https://docs.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development), чтобы решить [проблемы с созданием символических ссылок для непривилегированных пользователей](https://github.com/vitejs/vite/issues/7390). Также может потребоваться [установить git `core.symlinks` в `true` для решения проблем с символическими ссылками в git](https://github.com/vitejs/vite/issues/5242).

Каждый интеграционный тест может быть выполнен в режиме dev-сервера или в режиме сборки.

- `pnpm test` по умолчанию запускает каждый интеграционный тест как в режиме сервера, так и в режиме сборки, а также модульные тесты.

- `pnpm run test-serve` запускает тесты только в режиме сервера.

- `pnpm run test-build` запускает тесты только в режиме сборки.

- Вы также можете использовать `pnpm run test-serve [match]` или `pnpm run test-build [match]` для запуска тестов в определенном пакете playground, например, `pnpm run test-serve asset` запускает тесты для `playground/asset` и `vite/src/node/__tests__/asset` в режиме сервера, а `vite/src/node/__tests__/**/*` запускает только в режиме сервера.

Обратите внимание, что сопоставление пакетов недоступно для скрипта `pnpm test`, который всегда запускает все тесты.

### Модульные тесты

Помимо тестов под `playground/` для интеграционных тестов, пакеты могут содержать модульные тесты в своей директории `__tests__`. Модульные тесты поддерживаются [Vitest](https://vitest.dev/). Подробная конфигурация находится в файлах `vitest.config.ts`.

- `pnpm run test-unit` запускает модульные тесты под каждым пакетом.

- Вы также можете использовать `pnpm run test-unit [match]` для запуска соответствующих тестов.

### Тестовая среда и утилиты

В тестах playground вы можете импортировать объект `page` из `~utils`, который является экземпляром Playwright [`Page`](https://playwright.dev/docs/api/class-page), уже перешедшим на страницу текущего playground. Написание теста так же просто, как:

```js
import { page } from '~utils'

test('should work', async () => {
  expect(await page.textContent('.foo')).toMatch('foo')
})
```

Некоторые общие тестовые помощники, такие как `testDir`, `isBuild` или `editFile`, также доступны в утилитах. Исходный код находится в `playground/test-utils.ts`.

Примечание: Тестовая среда сборки использует [другую стандартную конфигурацию Vite](https://github.com/vitejs/vite/blob/main/playground/vitestSetup.ts#L102-L122), чтобы пропустить транспиляцию во время тестов и ускорить сборку. Это может привести к разным результатам по сравнению со стандартной продакшн-сборкой.

### Расширение набора тестов

Чтобы добавить новые тесты, вы должны найти соответствующий playground для исправления или функции (или создать новый). Примером являются тесты для загрузки статических ресурсов в [Asset Playground](https://github.com/vitejs/vite/tree/main/playground/assets). В этом Vite-приложении есть тест для импортов `?raw`, для которого [определен раздел в `index.html`](https://github.com/vitejs/vite/blob/main/playground/assets/index.html#L121):

```html
<h2>?raw import</h2>
<code class="raw"></code>
```

Это [обновляется с результатом импорта файла](https://github.com/vitejs/vite/blob/main/playground/assets/index.html#L151):

```js
import rawSvg from './nested/fragment.svg?raw'
text('.raw', rawSvg)
```

Где утилита `text` определена как:

```js
function text(el, text) {
  document.querySelector(el).textContent = text
}
```

В [спецификационных тестах](https://github.com/vitejs/vite/blob/main/playground/assets/__tests__/assets.spec.ts#L180) используются изменения DOM, перечисленные выше, для тестирования этой функции:

```js
test('?raw import', async () => {
  expect(await page.textContent('.raw')).toMatch('SVG')
})
```

## Примечание о тестовых зависимостях

Во многих тестовых случаях нам нужно мокать зависимости с протоколами `link:` и `file:`. `pnpm` обрабатывает `link:` как символические ссылки и `file:` как ссылки на диске. Для тестирования зависимостей, как если бы они были скопированы в `node_modules`, используйте протокол `file:`, в других случаях должен использоваться протокол `link:`.

## Размышления перед добавлением зависимости

Большинство зависимостей по умолчанию должны быть добавлены в `devDependencies`, даже если они нужны во время выполнения. Некоторые исключения:

- Пакеты типов. Пример: `@types/*`.
- Зависимости, которые не могут быть правильно собраны из-за бинарных файлов. Пример: `esbuild`.
- Зависимости, которые предоставляют свои собственные типы и чей тип используется в публичных типах Vite. Пример: `rollup`.

Избегайте зависимостей, которые имеют большие транзитивные зависимости и раздуты по сравнению с предоставляемой функциональностью. Например, сам `http-proxy` плюс `@types/http-proxy` составляет чуть более 1 МБ, но `http-proxy-middleware` тянет много зависимостей, так что он составляет 7 МБ (!), в то время как минимальное пользовательское middleware поверх `http-proxy` требует всего несколько строк кода.

### Обеспечение поддержки типов

Vite должен быть полностью пригоден для использования как зависимость в TypeScript-проекте (например, он должен предоставлять правильные типы для VitePress) и также в `vite.config.ts`. Технически это означает, что зависимость, чьи типы предоставляются, должна быть в `dependencies` вместо `devDependencies`. Проблема в том, что мы не можем их собрать.

Чтобы обойти это, мы встраиваем некоторые типы этих зависимостей в `packages/vite/src/dep-types`. Таким образом, мы можем продолжать предоставлять типизацию, но исходный код зависимости будет собран.

Используйте `pnpm run check-dist-types`, чтобы проверить, что собранные типы не зависят от типов в `devDependencies`. Если вы добавляете `dependencies`, убедитесь, что вы настроили `tsconfig.check.json`.

### Размышления перед добавлением еще одной опции

У нас уже есть много конфигурационных опций, и мы не должны решать проблемы, добавляя еще одну опцию. Перед добавлением опции подумайте:

- Действительно ли проблема должна быть решена.
- Может ли проблема быть решена с помощью более умного значения по умолчанию.
- Может ли проблема быть решена с помощью существующих опций.
- Может ли проблема быть решена с помощью плагина вместо этого.

## Руководящие принципы для Pull Requests

- Создайте ветку темы от базовой ветки, например `main`, и объедините её с этой веткой.

- При добавлении новой функции:

  - Добавьте сопровождающий тестовый случай.
  - Предоставьте убедительную причину для добавления этой функции. В идеале вы должны сначала открыть предложение проблемы и получить его одобрение, прежде чем работать над ним.

- При исправлении ошибки:

  - Если вы решаете конкретную проблему, добавьте `(fix #xxxx[,#xxxx])` (#xxxx - это ID проблемы) в заголовок вашего PR для лучшего журнала изменений, например `fix: Update entities encoding/decoding (fix #3899)`.
  - Предоставьте подробное описание ошибки в PR. Предпочтительна живая демонстрация.
  - При необходимости добавьте соответствующее тестовое покрытие.

- Нормально иметь несколько маленьких коммитов во время работы над PR - GitHub может автоматически объединить их перед слиянием.

- Убедитесь, что тесты проходят!

- Сообщения коммитов должны следовать [конвенции сообщений коммитов](./.github/commit-convention.md), чтобы журналы изменений могли генерироваться автоматически. Сообщения коммитов автоматически проверяются перед коммитом (через вызов [Git Hooks](https://git-scm.com/docs/githooks) через [yorkie](https://github.com/yyx990803/yorkie)).

- Вам не нужно беспокоиться о стиле кода, пока у вас установлены зависимости разработки - измененные файлы автоматически форматируются с помощью Prettier при коммите (через вызов [Git Hooks](https://git-scm.com/docs/githooks) через [yorkie](https://github.com/yyx990803/yorkie)).

## Руководящие принципы обслуживания

> Следующий раздел в основном предназначен для maintainer'ов, у которых есть доступ к репозиторию, но полезен для прочтения, если вы хотите внести нетривиальные вклады в код.

### Рабочий процесс обработки проблем

![Рабочий процесс обработки проблем](./.github/issue-workflow.png)

### Рабочий процесс проверки Pull Request

![Рабочий процесс проверки Pull Request](./.github/pr-workflow.png)

## Примечания о зависимостях

Vite должен быть легким, включая количество npm-зависимостей и их размер.

Мы используем Rollup для предварительной обработки большинства зависимостей перед публикацией! Поэтому большинство зависимостей, даже если они используются в исходном коде, по умолчанию должны быть добавлены в `devDependencies`. Это также приводит к некоторым ограничениям, которые мы должны учитывать в коде:

### Использование `require()`

В некоторых случаях мы намеренно используем ленивое требование для некоторых зависимостей, чтобы улучшить производительность запуска. Однако обратите внимание, что мы не можем использовать простые вызовы `require('somedep')`, так как они игнорируются в ESM-файлах и, следовательно, зависимость не будет включена в бандл, и фактическая зависимость не будет присутствовать при публикации, так как она находится в `devDependencies`.

Вместо этого используйте `(await import('somedep')).default`.

### Размышления перед добавлением зависимости

Большинство зависимостей, даже если они нужны во время выполнения, по умолчанию должны быть добавлены в `devDependencies`. Некоторые исключения:

- Пакеты типов. Пример: `@types/*`.
- Зависимости, которые не могут быть правильно собраны из-за бинарных файлов. Пример: `esbuild`.
- Зависимости, которые предоставляют свои собственные типы и чей тип используется в публичных типах Vite. Пример: `rollup`.

Избегайте зависимостей, которые имеют большие транзитивные зависимости и раздуты по сравнению с предоставляемой функциональностью. Например, сам `http-proxy` плюс `@types/http-proxy` составляет чуть более 1 МБ, но `http-proxy-middleware` тянет много зависимостей, так что он составляет 7 МБ (!), в то время как минимальное пользовательское middleware поверх `http-proxy` требует всего несколько строк кода.

### Обеспечение поддержки типов

Vite должен быть полностью пригоден для использования как зависимость в TypeScript-проекте (например, он должен предоставлять правильные типы для VitePress) и также в `vite.config.ts`. Технически это означает, что зависимость, чьи типы предоставляются, должна быть в `dependencies` вместо `devDependencies`. Проблема в том, что мы не можем их собрать.

Чтобы обойти это, мы встраиваем некоторые типы этих зависимостей в `packages/vite/src/dep-types`. Таким образом, мы можем продолжать предоставлять типизацию, но исходный код зависимости будет собран.

Используйте `pnpm run check-dist-types`, чтобы проверить, что собранные типы не зависят от типов в `devDependencies`. Если вы добавляете `dependencies`, убедитесь, что вы настроили `tsconfig.check.json`.

### Размышления перед добавлением еще одной опции

У нас уже есть много конфигурационных опций, и мы не должны решать проблемы, добавляя еще одну опцию. Перед добавлением опции подумайте:

- Действительно ли проблема должна быть решена.
- Может ли проблема быть решена с помощью более умного значения по умолчанию.
- Может ли проблема быть решена с помощью существующих опций.
- Может ли проблема быть решена с помощью плагина вместо этого.

### Richtlinien für Versionsnummern und Änderungsprotokolle

Wir folgen [Semver](https://semver.org/) (Semantic Versioning), um die Versionsnummer von Vite zu verwalten. Hier sind einige Leitlinien, die wir befolgen:

- **Major (1.0.0, 2.0.0, usw.)**: Änderungen, die bestehende APIs brechen oder große funktionale Änderungen einführen. Dies sollten wir nur sehr vorsichtig tun und nur für wirklich signifikante Verbesserungen oder Änderungen. In der Praxis sollten Breaking Changes nur sehr selten vorkommen.

- **Minor (1.1.0, 1.2.0, usw.)**: Neue Features oder Funktionen, die hinzugefügt werden, ohne bestehende APIs zu brechen.

- **Patch (1.0.1, 1.0.2, usw.)**: Fehlerbehebungen, die keine API-Änderungen darstellen.

Um sicherzustellen, dass Versionsnummern korrekt vergeben werden, verwenden wir die folgenden Werkzeuge:

- Wir verwenden [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) und [Conventional Changelog](https://github.com/conventional-changelog/conventional-changelog) für unsere Commit-Nachrichten.

- Wir verwenden [semantic-release](https://semantic-release.gitbook.io/semantic-release/) und [GitHub Actions](https://docs.github.com/en/actions) für automatische Veröffentlichungen. Dies automatisiert den gesamten Prozess der Versionsverwaltung und der Änderungsprotokolle.

- Jeder Pull Request sollte eine Liste von Änderungen und möglichen Auswirkungen auf bestehende Nutzer enthalten.

- Für wichtige Änderungen oder Erweiterungen, die erfordern, dass bestehender Code oder Konfigurationen angepasst werden, sollte dies in der Dokumentation und in Veröffentlichungshinweisen dokumentiert werden.

- Die Versionsnummern sollten niemals manuell geändert werden. Dies wird von semantic-release verwaltet.

### Versionsverwaltung mit semantic-release

Wir verwenden [semantic-release](https://semantic-release.gitbook.io/semantic-release/) für die automatische Versionsverwaltung und das Erstellen von Änderungsprotokollen. Hier ist, wie es funktioniert:

- Jeder Pull Request sollte eine [konventionelle Commit-Nachricht](https://www.conventionalcommits.org/en/v1.0.0/) haben. Dies wird verwendet, um festzustellen, ob es sich um eine Änderung handelt, die eine neue Version von Vite erfordert.

- Wenn ein Pull Request gemerged wird, löst er einen GitHub-Workflow aus, der semantic-release verwendet, um die neue Version zu ermitteln und die Veröffentlichung vorzubereiten.

- Semantic-release verwendet die Commit-Nachrichten und den bisherigen Versionsverlauf, um die geeignete Versionsnummer zu bestimmen (z. B. Patch, Minor, Major).

- Es erstellt automatisch ein Änderungsprotokoll basierend auf den Commit-Nachrichten.

- Es veröffentlicht die neue Version von Vite auf npm und aktualisiert die Versionsnummern in den `package.json`-Dateien.

- Es erstellt auch ein GitHub-Release mit den Änderungsprotokollen und verlinkt auf die veröffentlichte Version auf npm.

Durch die Verwendung von semantic-release stellen wir sicher, dass die Versionsverwaltung konsistent und automatisiert ist und dass Änderungsprotokolle immer aktuell und präzise sind. Dies erleichtert es den Benutzern von Vite, die Versionshistorie nachzuverfolgen und sicherzustellen, dass sie immer die neueste Version verwenden können.
