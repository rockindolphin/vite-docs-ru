# Хук плагина HMR `hotUpdate`

::: tip Обратная связь
Оставьте нам отзыв в [обсуждении обратной связи по API окружений](https://github.com/vitejs/vite/discussions/16358)
:::

Мы планируем объявить устаревшим хук плагина `handleHotUpdate` в пользу хуков [`hotUpdate`](/guide/api-environment#the-hotupdate-hook), чтобы быть осведомленным о [API окружений](/guide/api-environment.md) и обрабатывать дополнительные события наблюдения с `create` и `delete`.

Затронутый круг: `Авторы плагинов Vite`

::: warning Будущее устаревание
`hotUpdate` был впервые представлен в `v6.0`. Устаревание `handleHotUpdate` запланировано на `v7.0`. Мы пока не рекомендуем отказываться от `handleHotUpdate`. Если вы хотите поэкспериментировать и оставить нам отзыв, вы можете использовать `future.removePluginHookHandleHotUpdate` для установки в `"warn"` в вашей конфигурации vite.
:::

## Мотивация

Хук [`handleHotUpdate`](/guide/api-plugin.md#handlehotupdate) позволяет выполнять пользовательскую обработку обновлений HMR. Список модулей, которые необходимо обновить, передается в `HmrContext`

```ts
interface HmrContext {
  file: string
<CURRENT_CURSOR_POSITION>
  timestamp: number
  modules: Array<ModuleNode>
  read: () => string | Promise<string>
  server: ViteDevServer
}
```

Этот хук вызывается один раз для всех окружений, и переданные модули содержат смешанную информацию из клиентских и SSR окружений. Как только фреймворки перейдут на пользовательские окружения, потребуется новый хук, который будет вызываться для каждого из них.

Новый хук `hotUpdate` работает так же, как `handleHotUpdate`, но вызывается для каждого окружения и получает новый экземпляр `HotUpdateOptions`:

```ts
interface HotUpdateOptions {
  type: 'create' | 'update' | 'delete'
  file: string
  timestamp: number
  modules: Array<EnvironmentModuleNode>
  read: () => string | Promise<string>
  server: ViteDevServer
}
```

Текущее окружение разработки можно получить так же, как и в других хуках плагина, с помощью `this.environment`. Список `modules` теперь будет содержать только узлы модулей из текущего окружения. Каждое обновление окружения может определять разные стратегии обновления.

Этот хук также теперь вызывается для дополнительных событий наблюдения, а не только для `'update'`. Используйте `type`, чтобы различать их.

## Руководство по миграции

Отфильтруйте и уточните список затронутых модулей, чтобы HMR был более точным.

```js
handleHotUpdate({ modules }) {
  return modules.filter(condition)
}

// Миграция на:

hotUpdate({ modules }) {
  return modules.filter(condition)
}
```

Вернуть пустой массив и выполнить полную перезагрузку:

```js
handleHotUpdate({ server, modules, timestamp }) {
  // Вручную аннулировать модули
  const invalidatedModules = new Set()
  for (const mod of modules) {
    server.moduleGraph.invalidateModule(
      mod,
      invalidatedModules,
      timestamp,
      true
    )
  }
  server.ws.send({ type: 'full-reload' })
  return []
}

// Миграция на:

hotUpdate({ modules, timestamp }) {
  // Вручную аннулировать модули
  const invalidatedModules = new Set()
  for (const mod of modules) {
    this.environment.moduleGraph.invalidateModule(
      mod,
      invalidatedModules,
      timestamp,
      true
    )
  }
  this.environment.hot.send({ type: 'full-reload' })
  return []
}
```

Вернуть пустой массив и выполнить полную пользовательскую обработку HMR, отправляя пользовательские события клиенту:

```js
handleHotUpdate({ server }) {
  server.ws.send({
    type: 'custom',
    event: 'special-update',
    data: {}
  })
  return []
}

// Миграция на...

hotUpdate() {
  this.environment.hot.send({
    type: 'custom',
    event: 'special-update',
    data: {}
  })
  return []
}
```
