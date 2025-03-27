---
title: Vite 6.0 вышел!
author:
  name: Команда Vite
date: 2024-11-26
sidebar: false
head:
  - - meta
    - property: og:type
      content: website
  - - meta
    - property: og:title
      content: Анонс Vite 6
  - - meta
    - property: og:image
      content: https://vite.dev/og-image-announcing-vite6.png
  - - meta
    - property: og:url
      content: https://vite.dev/blog/announcing-vite6
  - - meta
    - property: og:description
      content: Анонс релиза Vite 6
  - - meta
    - name: twitter:card
      content: summary_large_image
---

# Vite 6.0 вышел!

_26 ноября 2024_

![Обложка анонса Vite 6](/og-image-announcing-vite6.png)

Сегодня мы делаем еще один большой шаг в истории Vite. Команда [Vite](/team), [контрибьюторы](https://github.com/vitejs/vite/graphs/contributors) и партнеры экосистемы рады объявить о выпуске Vite 6.

Это был насыщенный год. Принятие Vite продолжает расти, с еженедельными загрузками через npm, увеличившимися с 7,5 миллионов до 17 миллионов с момента выпуска Vite 5 год назад. [Vitest](https://vitest.dev) не только все больше нравится пользователям, но и начинает формировать свою собственную экосистему. Например, [Storybook](https://storybook.js.org) имеет новые возможности тестирования, работающие на Vitest.

Новые фреймворки также присоединились к экосистеме Vite, включая [TanStack Start](https://tanstack.com/start), [One](https://onestack.dev/), [Ember](https://emberjs.com/) и другие. Веб-фреймворки развиваются все быстрее. Вы можете ознакомиться с улучшениями, которые делают ребята в [Astro](https://astro.build/), [Nuxt](https://nuxt.com/), [SvelteKit](https://kit.svelte.dev/), [Solid Start](https://www.solidjs.com/blog/introducing-solidstart), [Qwik City](https://qwik.builder.io/qwikcity/overview/), [RedwoodJS](https://redwoodjs.com/), [React Router](https://reactrouter.com/), и список продолжается.

Vite используется OpenAI, Google, Apple, Microsoft, NASA, Shopify, Cloudflare, GitLab, Reddit, Linear и многими другими. Два месяца назад мы начали вести список [компаний, использующих Vite](https://github.com/vitejs/companies-using-vite). Мы рады видеть, как многие разработчики отправляют нам PR для добавления своих компаний в список. Трудно поверить, насколько выросла экосистема, которую мы построили вместе, с тех пор как Vite сделал свои первые шаги.

![Еженедельные загрузки Vite через npm](/vite6-npm-weekly-downloads.png)

## Ускорение экосистемы Vite

В прошлом месяце сообщество собралось на третью конференцию [ViteConf](https://viteconf.org/24/replay), снова организованную [StackBlitz](https://stackblitz.com). Это была самая большая конференция Vite с широким представительством разработчиков из экосистемы. Среди других объявлений, Эван Ю объявил о [VoidZero](https://staging.voidzero.dev/posts/announcing-voidzero-inc), компании, посвященной созданию открытой, высокопроизводительной и унифицированной цепочки инструментов разработки для экосистемы JavaScript. VoidZero стоит за [Rolldown](https://rolldown.rs) и [Oxc](https://oxc.rs), и их команда делает значительные успехи, быстро готовя их к использованию в Vite. Посмотрите ключевое выступление Эвана, чтобы узнать больше о следующих шагах для будущего Vite на Rust.

<YouTubeVideo videoId="EKvvptbTx6k?si=EZ-rFJn4pDW3tUvp" />

[Stackblitz](https://stackblitz.com) представил [bolt.new](https://bolt.new), приложение Remix, которое сочетает Claude и WebContainers и позволяет вам создавать, редактировать, запускать и развертывать полнофункциональные приложения. Нейт Вейнер объявил о [One](https://onestack.dev/), новом React-фреймворке на Vite для веб и нативных платформ. Storybook продемонстрировал свои последние функции [тестирования](https://youtu.be/8t5wxrFpCQY?si=PYZoWKf-45goQYDt) на Vitest. И многое другое. Мы призываем вас посмотреть [все 43 доклада](https://www.youtube.com/playlist?list=PLqGQbXn_GDmnObDzgjUF4Krsfl6OUKxtp). Докладчики приложили значительные усилия, чтобы поделиться с нами тем, над чем работает каждый проект.

Vite также получил обновленную главную страницу и чистый домен. Вам следует обновить ваши URL-адреса, чтобы они указывали на новый домен [vite.dev](https://vite.dev). Новый дизайн и реализация были сделаны VoidZero, теми же людьми, которые сделали их сайт. Спасибо [Висенте Родригесу](https://bento.me/rmoon) и [Саймону Ле Маршанту](https://marchantweb.com/).

## Следующий мажорный релиз Vite здесь

Vite 6 - это самый значительный мажорный релиз со времен Vite 2. Мы стремимся сотрудничать с экосистемой, чтобы продолжать расширять наши общие возможности через новые API и, как обычно, более отполированную базу для построения.

Быстрые ссылки:

- [Документация](/)
- Переводы: [简体中文](https://cn.vite.dev/), [日本語](https://ja.vite.dev/), [Español](https://es.vite.dev/), [Português](https://pt.vite.dev/), [한국어](https://ko.vite.dev/), [Deutsch](https://de.vite.dev/)
- [Руководство по миграции](/guide/migration)
- [Changelog на GitHub](https://github.com/vitejs/vite/blob/main/packages/vite/CHANGELOG.md#600-2024-11-26)

Если вы новичок в Vite, мы предлагаем сначала прочитать руководства [Начало работы](/guide/) и [Возможности](/guide/features).

Мы хотим поблагодарить более чем [1000 контрибьюторов Vite Core](https://github.com/vitejs/vite/graphs/contributors) и сопровождающих и контрибьюторов плагинов Vite, интеграций, инструментов и переводов, которые помогли нам создать этот новый мажорный релиз. Мы приглашаем вас принять участие и помочь нам улучшить Vite для всей экосистемы. Узнайте больше в нашем [Руководстве по внесению вклада](https://github.com/vitejs/vite/blob/main/CONTRIBUTING.md).

Для начала мы предлагаем помочь с [рассмотрением проблем](https://github.com/vitejs/vite/issues), [проверкой PR](https://github.com/vitejs/vite/pulls), отправкой PR с падающими тестами на основе открытых проблем и поддержкой других в [Обсуждениях](https://github.com/vitejs/vite/discussions) и [форуме помощи](https://discord.com/channels/804011606160703521/1019670660856942652) Vite Land. Если вы хотите поговорить с нами, присоединяйтесь к нашему [сообществу Discord](http://chat.vite.dev/) и поздоровайтесь в канале [#contributing](https://discord.com/channels/804011606160703521/804439875226173480).

Для получения последних новостей об экосистеме Vite и ядре Vite, следите за нами в [Bluesky](https://bsky.app/profile/vite.dev), [X](https://twitter.com/vite_js) или [Mastodon](https://webtoo.ls/@vite).

## Начало работы с Vite 6

Вы можете использовать `pnpm create vite` для быстрого создания приложения Vite с вашим предпочтительным фреймворком или поиграть онлайн с Vite 6, используя [vite.new](https://vite.new). Вы также можете запустить `pnpm create vite-extra` для получения доступа к шаблонам других фреймворков и сред выполнения (Solid, Deno, SSR и стартеры для библиотек). Шаблоны `create vite-extra` также доступны, когда вы запускаете `create vite` под опцией `Others`.

Шаблоны-стартеры Vite предназначены для использования в качестве игровой площадки для тестирования Vite с различными фреймворками. При создании вашего следующего проекта вам следует обратиться к стартеру, рекомендованному каждым фреймворком. `create vite` также предоставляет ярлык для настройки правильных стартеров некоторыми фреймворками, такими как `create-vue`, `Nuxt 3`, `SvelteKit`, `Remix`, `Analog` и `Angular`.

## Поддержка Node.js

Vite 6 поддерживает Node.js 18, 20 и 22+, как и Vite 5. Поддержка Node.js 21 была прекращена. Vite прекращает поддержку старых версий после их [EOL](https://endoflife.date/nodejs). EOL для Node.js 18 наступает в конце апреля 2025 года, после чего мы можем выпустить новый мажорный релиз для повышения требуемой версии Node.js.

## Экспериментальный API окружения

Vite становится более гибким с новым API окружения. Эти новые API позволят авторам фреймворков предложить опыт разработки, более близкий к продакшену, и для экосистемы - поделиться новыми строительными блоками. Ничего не меняется, если вы создаете SPA; когда вы используете Vite с одним клиентским окружением, все работает как раньше. И даже для пользовательских SSR-приложений Vite 6 обратно совместим. Основная целевая аудитория для API окружения - это авторы фреймворков.

Для конечных пользователей, которым интересно, [Sapphi](https://github.com/sapphi-red) написал отличное [Введение в API окружения](https://green.sapphi.red/blog/increasing-vites-potential-with-the-environment-api). Это отличное место для начала и понимания, почему мы пытаемся сделать Vite еще более гибким.

Если вы автор фреймворка или сопровождающий плагина Vite и хотели бы использовать новые API, вы можете узнать больше в [Руководствах по API окружения](https://main.vite.dev/guide/api-environment).

Мы хотим поблагодарить всех, кто участвовал в определении и реализации новых API. История начинается с того, как Vite 2 принял схему небандлированного SSR dev, впервые предложенную [Ричем Харрисом](https://github.com/Rich-Harris) и командой [SvelteKit](https://svelte.dev/docs/kit). Затем SSR-трансформация Vite позволила [Энтони Фу](https://github.com/antfu/) и [Пуйе Парсе](https://github.com/pi0) создать vite-node и улучшить [Dev SSR в Nuxt](https://antfu.me/posts/dev-ssr-on-nuxt). Энтони пошел использовать vite-node для питания [Vitest](https://vitest.dev), а [Владимир Шеремет](https://github.com/sheremet-va) продолжал улучшать его как часть своей работы по сопровождению Vitest. В начале 2023 года Владимир начал работать над переносом vite-node в Vite Core, и мы выпустили его как Runtime API в Vite 5.1 год спустя. Отзывы от партнеров экосистемы (особое спасибо команде Cloudflare) подтолкнули нас к более амбициозной переработке окружений Vite. Вы можете узнать больше об этой истории в [выступлении Патака на ViteConf 24](https://www.youtube.com/watch?v=WImor3HDyqU?si=EZ-rFJn4pDW3tUvp).

Все в команде Vite участвовали в определении нового API, который был разработан совместно с отзывами от многих проектов в экосистеме. Спасибо всем участникам! Мы призываем вас принять участие, если вы создаете фреймворк, плагин или инструмент поверх Vite. Новые API являются экспериментальными. Мы будем работать с экосистемой, чтобы пересмотреть, как будут использоваться новые API, и стабилизировать их для следующего мажорного релиза. Если у вас есть вопросы или отзывы, есть [открытое обсуждение на GitHub здесь](https://github.com/vitejs/vite/discussions/16358).

## Основные изменения

- [Значение по умолчанию для `resolve.conditions`](/guide/migration#default-value-for-resolve-conditions)
- [JSON stringify](/guide/migration#json-stringify)
- [Расширенная поддержка ссылок на ресурсы в HTML-элементах](/guide/migration#extended-support-of-asset-references-in-html-elements)
- [postcss-load-config](/guide/migration#postcss-load-config)
- [Sass теперь использует современный API по умолчанию](/guide/migration#sass-now-uses-modern-api-by-default)
- [Настройка имени выходного файла CSS в режиме библиотеки](/guide/migration#customize-css-output-file-name-in-library-mode)
- [И другие изменения, которые должны затронуть только немногих пользователей](/guide/migration#advanced)

Также есть новая страница [Критические изменения](/changes/), которая перечисляет все запланированные, рассматриваемые и прошлые изменения в Vite.

## Миграция на Vite 6

Для большинства проектов обновление до Vite 6 должно быть простым, но мы советуем просмотреть [подробное руководство по миграции](/guide/migration) перед обновлением.

Полный список изменений находится в [Changelog Vite 6](https://github.com/vitejs/vite/blob/main/packages/vite/CHANGELOG.md#500-2024-11-26).

## Благодарности

Vite 6 является результатом долгих часов работы нашего сообщества контрибьюторов, сопровождающих нижестоящих проектов, авторов плагинов и [Команды Vite](/team). Мы ценим людей и компании, спонсирующие разработку Vite. Vite создан для вас [VoidZero](https://voidzero.dev) в партнерстве со [StackBlitz](https://stackblitz.com/), [Nuxt Labs](https://nuxtlabs.com/) и [Astro](https://astro.build). Спасибо спонсорам на [GitHub Sponsors Vite](https://github.com/sponsors/vitejs) и [Open Collective Vite](https://opencollective.com/vite).
