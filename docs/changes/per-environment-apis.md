# Переход к API для каждого окружения

::: tip Обратная связь
Оставьте нам обратную связь в [обсуждении обратной связи по API окружений](https://github.com/vitejs/vite/discussions/16358)
:::

Несколько API из `ViteDevServer`, связанных с графом модулей и трансформацией модулей, были перенесены в экземпляры `DevEnvironment`.

Затрагивает: `Авторы плагинов Vite`

::: warning Будущее устаревание
Экземпляр `Environment` был впервые представлен в `v6.0`. Устаревание `server.moduleGraph` и других методов, которые теперь находятся в окружениях, запланировано для `v7.0`. Мы пока не рекомендуем отказываться от методов сервера. Чтобы определить ваше использование, установите это в вашей конфигурации vite.

```ts
future: {
  removeServerModuleGraph: 'warn',
  removeServerTransformRequest: 'warn',
}
```

:::

## Мотивация

В Vite v5 и ранее, один dev-сервер Vite всегда имел два окружения (`client` и `ssr`). `server.moduleGraph` содержал смешанные модули из обоих этих окружений. Узлы были связаны через списки `clientImportedModules` и `ssrImportedModules` (но единый список `importers` поддерживался для каждого). Трансформированный модуль представлялся через `id` и булево значение `ssr`. Это булево значение нужно было передавать в API, например `server.moduleGraph.getModuleByUrl(url, ssr)` и `server.transformRequest(url, { ssr })`.

В Vite v6 теперь возможно создавать любое количество пользовательских окружений (`client`, `ssr`, `edge` и т.д.). Одного булева значения `ssr` больше недостаточно. Вместо изменения API к форме `server.transformRequest(url, { environment })`, мы перенесли эти методы в экземпляр окружения, позволяя вызывать их без dev-сервера Vite.

## Руководство по миграции

- `server.moduleGraph` -> [`environment.moduleGraph`](/guide/api-environment#separate-module-graphs)
- `server.transformRequest(url, ssr)` -> `environment.transformRequest(url)`
- `server.warmupRequest(url, ssr)` -> `environment.warmupRequest(url)`
