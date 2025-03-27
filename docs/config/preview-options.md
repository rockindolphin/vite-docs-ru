# Опции предпросмотра

Если не указано иное, параметры в этом разделе применяются только к предпросмотру.

## preview.host

- **Тип:** `string | boolean`
- **По умолчанию:** [`server.host`](./server-options#server-host)

Укажите, на каких IP-адресах сервер должен слушать.
Установите это значение в `0.0.0.0` или `true`, чтобы слушать на всех адресах, включая LAN и публичные адреса.

Это можно установить через CLI, используя `--host 0.0.0.0` или `--host`.

::: tip ПРИМЕЧАНИЕ

Существуют случаи, когда другие серверы могут отвечать вместо Vite.
Смотрите [`server.host`](./server-options#server-host) для получения дополнительных сведений.

:::

## preview.allowedHosts

- **Тип:** `string | true`
- **По умолчанию:** [`server.allowedHosts`](./server-options#server-allowedhosts)

Имена хостов, на которые Vite разрешено отвечать.

Смотрите [`server.allowedHosts`](./server-options#server-allowedhosts) для получения дополнительных сведений.

## preview.port

- **Тип:** `number`
- **По умолчанию:** `4173`

Укажите порт сервера. Обратите внимание, что если порт уже используется, Vite автоматически попытается использовать следующий доступный порт, поэтому это может не быть фактическим портом, на котором в конечном итоге будет слушать сервер.

**Пример:**

```js
export default defineConfig({
  server: {
    port: 3030,
  },
  preview: {
    port: 8080,
  },
})
```

## preview.strictPort

- **Тип:** `boolean`
- **По умолчанию:** [`server.strictPort`](./server-options#server-strictport)

Установите в `true`, чтобы завершить работу, если порт уже используется, вместо автоматической попытки использовать следующий доступный порт.

## preview.https

- **Тип:** `https.ServerOptions`
- **По умолчанию:** [`server.https`](./server-options#server-https)

Включить TLS + HTTP/2.

Смотрите [`server.https`](./server-options#server-https) для получения дополнительных сведений.

## preview.open

- **Тип:** `boolean | string`
- **По умолчанию:** [`server.open`](./server-options#server-open)

Автоматически открывать приложение в браузере при запуске сервера. Когда значение является строкой, оно будет использоваться как путь URL. Если вы хотите открыть сервер в конкретном браузере, который вам нравится, вы можете установить переменную окружения `process.env.BROWSER` (например, `firefox`). Вы также можете установить `process.env.BROWSER_ARGS`, чтобы передать дополнительные аргументы (например, `--incognito`).

`BROWSER` и `BROWSER_ARGS` также являются специальными переменными окружения, которые вы можете установить в файле `.env`, чтобы настроить их. Смотрите [пакет `open`](https://github.com/sindresorhus/open#app) для получения дополнительных сведений.

## preview.proxy

- **Тип:** `Record<string, string | ProxyOptions>`
- **По умолчанию:** [`server.proxy`](./server-options#server-proxy)

Настройте пользовательские правила прокси для сервера предпросмотра. Ожидает объект пар {ключ: опции}. Если ключ начинается с `^`, он будет интерпретироваться как `RegExp`. Опция `configure` может быть использована для доступа к экземпляру прокси.

Использует [`http-proxy`](https://github.com/http-party/node-http-proxy). Полные опции [здесь](https://github.com/http-party/node-http-proxy#options).

## preview.cors

- **Тип:** `boolean | CorsOptions`
- **По умолчанию:** [`server.cors`](./server-options#server-cors)

Настройте CORS для сервера предпросмотра.

Смотрите [`server.cors`](./server-options#server-cors) для получения дополнительных сведений.

## preview.headers

- **Тип:** `OutgoingHttpHeaders`

Укажите заголовки ответа сервера.
