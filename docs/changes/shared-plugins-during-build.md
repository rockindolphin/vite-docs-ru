# Общие плагины во время сборки

::: tip Обратная связь
Оставьте нам обратную связь в [обсуждении обратной связи по API окружений](https://github.com/vitejs/vite/discussions/16358)
:::

См. [Общие плагины во время сборки](/guide/api-environment.md#shared-plugins-during-build).

Затрагивает: `Авторы плагинов Vite`

::: warning Будущее изменение по умолчанию
`builder.sharedConfigBuild` был впервые представлен в `v6.0`. Вы можете установить его в true, чтобы проверить, как ваши плагины работают с общей конфигурацией. Мы рассматриваем изменение значения по умолчанию в будущей мажорной версии, как только экосистема плагинов будет готова.
:::

## Мотивация

Выравнивание пайплайнов плагинов dev и build.

## Руководство по миграции

Чтобы иметь возможность делиться плагинами между окружениями, состояние плагина должно быть привязано к текущему окружению. Плагин следующей формы будет подсчитывать количество трансформированных модулей во всех окружениях.

```js
function CountTransformedModulesPlugin() {
  let transformedModules
  return {
    name: 'count-transformed-modules',
    buildStart() {
      transformedModules = 0
    },
    transform(id) {
      transformedModules++
    },
    buildEnd() {
      console.log(transformedModules)
    },
  }
}
```

Если мы вместо этого хотим подсчитывать количество трансформированных модулей для каждого окружения, нам нужно хранить карту:

```js
function PerEnvironmentCountTransformedModulesPlugin() {
  const state = new Map<Environment, { count: number }>()
  return {
    name: 'count-transformed-modules',
    perEnvironmentStartEndDuringDev: true,
    buildStart() {
      state.set(this.environment, { count: 0 })
    }
    transform(id) {
      state.get(this.environment).count++
    },
    buildEnd() {
      console.log(this.environment.name, state.get(this.environment).count)
    }
  }
}
```

Чтобы упростить этот паттерн, Vite экспортирует помощник `perEnvironmentState`:

```js
function PerEnvironmentCountTransformedModulesPlugin() {
  const state = perEnvironmentState<{ count: number }>(() => ({ count: 0 }))
  return {
    name: 'count-transformed-modules',
    perEnvironmentStartEndDuringDev: true,
    buildStart() {
      state(this).count = 0
    }
    transform(id) {
      state(this).count++
    },
    buildEnd() {
      console.log(this.environment.name, state(this).count)
    }
  }
}
```
