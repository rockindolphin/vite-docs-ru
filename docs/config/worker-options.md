# Опции Worker

Если не указано иное, параметры в этом разделе применяются ко всем dev, build и preview.

## worker.format

- **Тип:** `'es' | 'iife'`
- **По умолчанию:** `'iife'`

Формат вывода для сборки worker.

## worker.plugins

- **Тип:** [`() => (Plugin | Plugin[])[]`](./shared-options#plugins)

Плагины Vite, которые применяются к сборкам worker. Обратите внимание, что [config.plugins](./shared-options#plugins) применяется только к worker в режиме разработки, его следует настраивать здесь вместо этого для сборки.
Функция должна возвращать новые экземпляры плагинов, так как они используются в параллельных сборках worker Rollup.

## worker.rollupOptions

- **Тип:** [`RollupOptions`](https://rollupjs.org/configuration-options/)

Опции Rollup для сборки worker.
