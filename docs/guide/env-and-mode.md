# Переменные окружения и режимы

Vite предоставляет определенные константы через специальный объект `import.meta.env`. Эти константы определяются как глобальные переменные во время разработки и статически заменяются во время сборки для эффективного tree-shaking.

## Встроенные константы

Некоторые встроенные константы доступны во всех случаях:

- **`import.meta.env.MODE`**: {string} [режим](#modes), в котором запущено приложение.

- **`import.meta.env.BASE_URL`**: {string} базовый URL, с которого обслуживается приложение. Это определяется опцией конфигурации [`base`](/config/shared-options.md#base).

- **`import.meta.env.PROD`**: {boolean} запущено ли приложение в production (запуск dev-сервера с `NODE_ENV='production'` или запуск приложения, собранного с `NODE_ENV='production'`).

- **`import.meta.env.DEV`**: {boolean} запущено ли приложение в development (всегда противоположно `import.meta.env.PROD`)

- **`import.meta.env.SSR`**: {boolean} запущено ли приложение на [сервере](./ssr.md#conditional-logic).

## Переменные окружения

Vite автоматически предоставляет переменные окружения через объект `import.meta.env` как строки.

Чтобы предотвратить случайную утечку переменных окружения в клиентский код, только переменные с префиксом `VITE_` доступны в коде, обработанном Vite. Например, для следующих переменных окружения:

```[.env]
VITE_SOME_KEY=123
DB_PASSWORD=foobar
```

Только `VITE_SOME_KEY` будет доступен как `import.meta.env.VITE_SOME_KEY` в вашем клиентском исходном коде, но `DB_PASSWORD` не будет.

```js
console.log(import.meta.env.VITE_SOME_KEY) // "123"
console.log(import.meta.env.DB_PASSWORD) // undefined
```

Если вы хотите настроить префикс переменных окружения, см. опцию [envPrefix](/config/shared-options.html#envprefix).

:::tip Парсинг переменных окружения
Как показано выше, `VITE_SOME_KEY` является числом, но возвращает строку при парсинге. То же самое происходит и с булевыми переменными окружения. Убедитесь, что вы преобразуете их в нужный тип при использовании в коде.
:::

### Файлы `.env`

Vite использует [dotenv](https://github.com/motdotla/dotenv) для загрузки дополнительных переменных окружения из следующих файлов в вашей [директории окружения](/config/shared-options.md#envdir):

```
.env                # загружается во всех случаях
.env.local          # загружается во всех случаях, игнорируется git
.env.[mode]         # загружается только в указанном режиме
.env.[mode].local   # загружается только в указанном режиме, игнорируется git
```

:::tip Приоритеты загрузки переменных окружения

Файл окружения для конкретного режима (например, `.env.production`) будет иметь более высокий приоритет, чем общий файл (например, `.env`).

Vite всегда загружает `.env` и `.env.local` в дополнение к файлу режима `.env.[mode]`. Переменные, объявленные в файлах режима, будут иметь приоритет над переменными в общих файлах, но переменные, определенные только в `.env` или `.env.local`, все равно будут доступны в окружении.

Кроме того, переменные окружения, которые уже существуют при запуске Vite, имеют наивысший приоритет и не будут перезаписаны файлами `.env`. Например, при запуске `VITE_SOME_KEY=123 vite build`.

Файлы `.env` загружаются при запуске Vite. Перезапустите сервер после внесения изменений.

:::

Также Vite использует [dotenv-expand](https://github.com/motdotla/dotenv-expand) для расширения переменных, записанных в файлах окружения. Чтобы узнать больше о синтаксисе, ознакомьтесь с [их документацией](https://github.com/motdotla/dotenv-expand#what-rules-does-the-expansion-engine-follow).

Обратите внимание, что если вы хотите использовать `$` внутри значения переменной окружения, вам нужно экранировать его с помощью `\`.

```[.env]
KEY=123
NEW_KEY1=test$foo   # test
NEW_KEY2=test\$foo  # test$foo
NEW_KEY3=test$KEY   # test123
```

:::warning ЗАМЕТКИ ПО БЕЗОПАСНОСТИ

- Файлы `.env.*.local` являются локальными и могут содержать конфиденциальные переменные. Вы должны добавить `*.local` в ваш `.gitignore`, чтобы избежать их добавления в git.

- Поскольку любые переменные, доступные в исходном коде Vite, попадут в ваш клиентский бандл, переменные `VITE_*` _не должны_ содержать никакой конфиденциальной информации.

:::

::: details Расширение переменных в обратном порядке

Vite поддерживает расширение переменных в обратном порядке.
Например, `.env` ниже будет оценен как `VITE_FOO=foobar`, `VITE_BAR=bar`.

```[.env]
VITE_FOO=foo${VITE_BAR}
VITE_BAR=bar
```

Это не работает в shell-скриптах и других инструментах, таких как `docker-compose`.
Тем не менее, Vite поддерживает это поведение, так как оно поддерживается `dotenv-expand` в течение длительного времени, и другие инструменты в экосистеме JavaScript используют более старые версии, которые поддерживают это поведение.

Чтобы избежать проблем с совместимостью, рекомендуется не полагаться на это поведение. Vite может начать выдавать предупреждения для этого поведения в будущем.

:::

## IntelliSense для TypeScript

По умолчанию Vite предоставляет определения типов для `import.meta.env` в [`vite/client.d.ts`](https://github.com/vitejs/vite/blob/main/packages/vite/client.d.ts). Хотя вы можете определить больше пользовательских переменных окружения в файлах `.env.[mode]`, вы можете захотеть получить TypeScript IntelliSense для пользовательских переменных окружения с префиксом `VITE_`.

Для этого вы можете создать `vite-env.d.ts` в директории `src`, а затем расширить `ImportMetaEnv` следующим образом:

```typescript [vite-env.d.ts]
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string
  // больше переменных окружения...
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

Если ваш код полагается на типы из окружений браузера, таких как [DOM](https://github.com/microsoft/TypeScript/blob/main/src/lib/dom.generated.d.ts) и [WebWorker](https://github.com/microsoft/TypeScript/blob/main/src/lib/webworker.generated.d.ts), вы можете обновить поле [lib](https://www.typescriptlang.org/tsconfig#lib) в `tsconfig.json`.

```json [tsconfig.json]
{
  "lib": ["WebWorker"]
}
```

:::warning Импорты нарушают расширение типов

Если расширение `ImportMetaEnv` не работает, убедитесь, что у вас нет никаких операторов `import` в `vite-env.d.ts`. См. [документацию TypeScript](https://www.typescriptlang.org/docs/handbook/2/modules.html#how-javascript-modules-are-defined) для получения дополнительной информации.

:::

## Замена констант в HTML

Vite также поддерживает замену констант в HTML-файлах. Любые свойства в `import.meta.env` могут использоваться в HTML-файлах с помощью специального синтаксиса `%CONST_NAME%`:

```html
<h1>Vite запущен в режиме %MODE%</h1>
<p>Используются данные из %VITE_API_URL%</p>
```

Если переменная окружения не существует в `import.meta.env`, например, `%NON_EXISTENT%`, она будет проигнорирована и не заменена, в отличие от `import.meta.env.NON_EXISTENT` в JS, где она заменяется как `undefined`.

Учитывая, что Vite используется многими фреймворками, он намеренно не навязывает мнения о сложных заменах, таких как условные операторы. Vite может быть расширен с помощью [существующего пользовательского плагина](https://github.com/vitejs/awesome-vite#transformers) или пользовательского плагина, который реализует хук [`transformIndexHtml`](./api-plugin#transformindexhtml).

## Режимы

По умолчанию dev-сервер (команда `dev`) запускается в режиме `development`, а команда `build` запускается в режиме `production`.

Это означает, что при запуске `vite build` он загрузит переменные окружения из `.env.production`, если он существует:

```[.env.production]
VITE_APP_TITLE=My App
```

В вашем приложении вы можете отобразить заголовок, используя `import.meta.env.VITE_APP_TITLE`.

В некоторых случаях вы можете захотеть запустить `vite build` с другим режимом, чтобы отобразить другой заголовок. Вы можете переопределить режим по умолчанию, используемый для команды, передав флаг опции `--mode`. Например, если вы хотите собрать ваше приложение для staging-режима:

```bash
vite build --mode staging
```

И создать файл `.env.staging`:

```[.env.staging]
VITE_APP_TITLE=My App (staging)
```

Поскольку `vite build` по умолчанию запускает production-сборку, вы также можете изменить это и запустить development-сборку, используя другой режим и конфигурацию `.env` файла:

```[.env.testing]
NODE_ENV=development
```

### NODE_ENV и режимы

Важно отметить, что `NODE_ENV` (`process.env.NODE_ENV`) и режимы - это два разных понятия. Вот как разные команды влияют на `NODE_ENV` и режим:

| Команда                                              | NODE_ENV        | Режим            |
| ---------------------------------------------------- | --------------- | --------------- |
| `vite build`                                         | `"production"`  | `"production"`  |
| `vite build --mode development`                      | `"production"`  | `"development"` |
| `NODE_ENV=development vite build`                    | `"development"` | `"production"`  |
| `NODE_ENV=development vite build --mode development` | `"development"` | `"development"` |

Разные значения `NODE_ENV` и режима также отражаются на соответствующих свойствах `import.meta.env`:

| Команда                | `import.meta.env.PROD` | `import.meta.env.DEV` |
| ---------------------- | ---------------------- | --------------------- |
| `NODE_ENV=production`  | `true`                 | `false`               |
| `NODE_ENV=development` | `false`                | `true`                |
| `NODE_ENV=other`       | `false`                | `true`                |

| Команда              | `import.meta.env.MODE` |
| -------------------- | ---------------------- |
| `--mode production`  | `"production"`         |
| `--mode development` | `"development"`        |
| `--mode staging`     | `"staging"`            |

:::tip `NODE_ENV` в файлах `.env`

`NODE_ENV=...` может быть установлен в команде, а также в вашем файле `.env`. Если `NODE_ENV` указан в файле `.env.[mode]`, режим может использоваться для управления его значением. Однако и `NODE_ENV`, и режимы остаются двумя разными понятиями.

Основное преимущество `NODE_ENV=...` в команде заключается в том, что он позволяет Vite обнаружить значение рано. Это также позволяет вам читать `process.env.NODE_ENV` в вашей конфигурации Vite, так как Vite может загружать файлы окружения только после оценки конфигурации.
:::
