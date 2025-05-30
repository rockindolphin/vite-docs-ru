# Сравнения

## WMR

[WMR](https://github.com/preactjs/wmr) от команды Preact предлагает аналогичный набор функций, и поддержка Vite 2.0 для интерфейса плагинов Rollup была вдохновлена им.

WMR был в основном разработан для проектов [Preact](https://preactjs.com/) и предлагает больше встроенных функций, таких как предварительная рендеринг. По своему объему это скорее мета-фреймворк Preact, с тем же акцентом на компактный размер, как и сам Preact. Если вы используете Preact, WMR, вероятно, предложит более согласованный опыт.

## @web/dev-server

[@web/dev-server](https://modern-web.dev/docs/dev-server/overview/) (ранее `es-dev-server`) - отличный проект, и настройка сервера Vite 1.0 на основе Koa была вдохновлена им.

`@web/dev-server` по своему объему несколько менее обширен. Он не предлагает официальные интеграции с фреймворками и требует ручной настройки конфигурации Rollup для продакшен-сборки.

В целом, Vite является более мнением / обобщающим инструментом, который нацелен на предоставление немедленно готового рабочего процесса. Тем не менее, проект `@web` включает в себя множество других отличных инструментов, которые также могут быть полезны пользователям Vite.

## Snowpack

[Snowpack](https://www.snowpack.dev/) также был нативным ESM-разработчиком без бандлинга, очень похожим на Vite. Проект больше не поддерживается. Команда Snowpack теперь работает над [Astro](https://astro.build/), генератором статических сайтов на основе Vite. Команда Astro теперь является активным участником экосистемы и помогает улучшать Vite.

Помимо различных деталей реализации, оба проекта имеют много технических преимуществ по сравнению с традиционными инструментами. Предварительная сборка зависимостей в Vite также была вдохновлена Snowpack v1 (теперь [`esinstall`](https://github.com/snowpackjs/snowpack/tree/main/esinstall)). Некоторые основные различия между двумя проектами описаны в [руководстве по сравнению v2](https://v2.vite.dev/guide/comparisons).
