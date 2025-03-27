# `this.environment` в хуках

::: tip Обратная связь
Оставьте нам обратную связь в [обсуждении обратной связи по API окружений](https://github.com/vitejs/vite/discussions/16358)
:::

До Vite 6 было доступно только два окружения: `client` и опционально `ssr`. Единственный аргумент `options.ssr` в хуках плагина `resolveId`, `load` и `transform` позволял авторам плагинов различать эти два окружения при обработке модулей в хуках плагина. В Vite 6 приложение Vite может определять любое количество именованных окружений по мере необходимости. Мы представляем `this.environment` в контексте плагина для взаимодействия с окружением текущего модуля в хуках.

Затрагивает: `Авторы плагинов Vite`

::: warning Будущее устаревание
`this.environment` был представлен в `v6.0`. Устаревание `options.ssr` запланировано для `v7.0`. В этот момент мы начнем рекомендовать миграцию ваших плагинов на новый API. Чтобы определить ваше использование, установите `future.removePluginHookSsrArgument` в `"warn"` в вашей конфигурации vite.
:::

## Мотивация

`this.environment` не только позволяет реализации хука плагина знать имя текущего окружения, но также дает доступ к опциям конфигурации окружения, информации о графе модулей и пайплайну трансформации (`environment.config`, `environment.moduleGraph`, `environment.transformRequest()`). Наличие экземпляра окружения в контексте позволяет авторам плагинов избежать зависимости от всего dev-сервера (обычно кэшируется при запуске через хук `configureServer`).

## Руководство по миграции

Для существующего плагина для быстрой миграции замените аргумент `options.ssr` на `this.environment.name !== 'client'` в хуках `resolveId`, `load` и `transform`:

```ts
import { Plugin } from 'vite'

export function myPlugin(): Plugin {
  return {
    name: 'my-plugin',
    resolveId(id, importer, options) {
      const isSSR = options.ssr // [!code --]
      const isSSR = this.environment.name !== 'client' // [!code ++]

      if (isSSR) {
        // Логика для SSR
      } else {
        // Логика для клиента
      }
    },
  }
}
```

Для более надежной долгосрочной реализации хук плагина должен обрабатывать [несколько окружений](/guide/api-environment.html#accessing-the-current-environment-in-hooks) используя детальные опции окружения вместо полагания на имя окружения.
