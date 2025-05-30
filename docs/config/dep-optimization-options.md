# Опции оптимизации зависимостей

- **Связано:** [Предварительная сборка зависимостей](/guide/dep-pre-bundling)

Если не указано иное, опции в этом разделе применяются только к оптимизатору зависимостей, который используется только в режиме разработки.

## optimizeDeps.entries

- **Тип:** `string | string[]`

По умолчанию Vite будет просматривать все ваши `.html` файлы, чтобы обнаружить зависимости, которые необходимо предварительно собрать (игнорируя `node_modules`, `build.outDir`, `__tests__` и `coverage`). Если указано `build.rollupOptions.input`, Vite будет просматривать эти точки входа вместо этого.

Если ни одно из этих условий не подходит для ваших нужд, вы можете указать пользовательские точки входа с помощью этой опции — значение должно быть [`tinyglobby` шаблоном](https://github.com/SuperchupuDev/tinyglobby) или массивом шаблонов, которые относительны от корня проекта Vite. Это перезапишет вывод по умолчанию. Только папки `node_modules` и `build.outDir` будут игнорироваться по умолчанию, когда `optimizeDeps.entries` явно определен. Если другие папки необходимо игнорировать, вы можете использовать шаблон игнорирования как часть списка точек входа, помеченный начальным `!`. Если вы не хотите игнорировать `node_modules` и `build.outDir`, вы можете указать их с помощью буквальных строковых путей (без шаблонов `tinyglobby`).

## optimizeDeps.exclude

- **Тип:** `string[]`

Зависимости, которые следует исключить из предварительной сборки.

:::warning CommonJS
Зависимости CommonJS не следует исключать из оптимизации. Если ESM зависимость исключена из оптимизации, но имеет вложенную зависимость CommonJS, то зависимость CommonJS должна быть добавлена в `optimizeDeps.include`. Пример:

```js twoslash
import { defineConfig } from 'vite'
// ---cut---
export default defineConfig({
  optimizeDeps: {
    include: ['esm-dep > cjs-dep'],
  },
})
```

:::

## optimizeDeps.include

- **Тип:** `string[]`

По умолчанию связанные пакеты, не находящиеся внутри `node_modules`, не предварительно собираются. Используйте эту опцию, чтобы принудительно предварительно собрать связанный пакет.

**Экспериментально:** Если вы используете библиотеку с множеством глубоких импортов, вы также можете указать завершающий шаблон glob, чтобы предварительно собрать все глубокие импорты сразу. Это позволит избежать постоянной предварительной сборки каждый раз, когда используется новый глубокий импорт. [Оставьте отзыв](https://github.com/vitejs/vite/discussions/15833). Например:

```js twoslash
import { defineConfig } from 'vite'
// ---cut---
export default defineConfig({
  optimizeDeps: {
    include: ['my-lib/components/**/*.vue'],
  },
})
```

## optimizeDeps.esbuildOptions

- **Тип:** [`Omit`](https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys)`<`[`EsbuildBuildOptions`](https://esbuild.github.io/api/#general-options)`,
| 'bundle'
| 'entryPoints'
| 'external'
| 'write'
| 'watch'
| 'outdir'
| 'outfile'
| 'outbase'
| 'outExtension'
| 'metafile'>`

Опции, которые передаются esbuild во время сканирования и оптимизации зависимостей.

Некоторые опции исключены, так как их изменение не будет совместимо с оптимизацией зависимостей Vite.

- `external` также исключен, используйте опцию `optimizeDeps.exclude` Vite

## optimizeDeps.force

- **Тип:** `boolean`

Установите в `true`, чтобы принудительно предварительно собрать зависимости, игнорируя ранее кэшированные оптимизированные зависимости.

## optimizeDeps.holdUntilCrawlEnd

- **Экспериментально:** [Оставьте отзыв](https://github.com/vitejs/vite/discussions/15834)
- **Тип:** `boolean`
- **По умолчанию:** `true`

Когда включено, он будет удерживать первые результаты оптимизированных зависимостей до тех пор, пока все статические импорты не будут просканированы при холодном старте. Это позволяет избежать необходимости полной перезагрузки страницы, когда обнаруживаются новые зависимости и они вызывают генерацию новых общих чанков. Если все зависимости найдены сканером плюс явно определенные в `include`, лучше отключить эту опцию, чтобы позволить браузеру обрабатывать больше запросов параллельно.

## optimizeDeps.disabled

- **Устарело**
- **Экспериментально:** [Оставьте отзыв](https://github.com/vitejs/vite/discussions/13839)
- **Тип:** `boolean | 'build' | 'dev'`
- **По умолчанию:** `'build'`

Эта опция устарела. Начиная с Vite 5.1, предварительная сборка зависимостей во время сборки была удалена. Установка `optimizeDeps.disabled` в `true` или `'dev'` отключает оптимизатор, а настройка в `false` или `'build'` оставляет оптимизатор включенным во время разработки.

Чтобы полностью отключить оптимизатор, используйте `optimizeDeps.noDiscovery: true`, чтобы запретить автоматическое обнаружение зависимостей и оставьте `optimizeDeps.include` неопределенным или пустым.

:::warning
Оптимизация зависимостей во время времени сборки была **экспериментальной** функцией. Проекты, пробующие эту стратегию, также удалили `@rollup/plugin-commonjs`, используя `build.commonjsOptions: { include: [] }`. Если вы это сделали, предупреждение подскажет вам повторно включить его для поддержки только CJS пакетов во время сборки.
:::

## optimizeDeps.needsInterop

- **Экспериментально**
- **Тип:** `string[]`

Принудительно ESM совместимость при импорте этих зависимостей. Vite может правильно определить, когда зависимость нуждается в совместимости, поэтому эта опция обычно не требуется. Однако различные комбинации зависимостей могут привести к тому, что некоторые из них будут предварительно собраны по-разному. Добавление этих пакетов в `needsInterop` может ускорить холодный старт, избегая полной перезагрузки страниц. Вы получите предупреждение, если это произойдет для одной из ваших зависимостей, предлагая добавить имя пакета в этот массив в вашей конфигурации.
