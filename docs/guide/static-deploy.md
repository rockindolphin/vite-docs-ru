# Развертывание статического сайта

Следующие руководства основаны на некоторых общих предположениях:

- Вы используете стандартное расположение выходных данных сборки (`dist`). Это расположение [можно изменить с помощью `build.outDir`](/config/build-options.md#build-outdir), и в этом случае вы можете экстраполировать инструкции из этих руководств.
- Вы используете npm. Вы можете использовать эквивалентные команды для запуска скриптов, если используете Yarn или другие менеджеры пакетов.
- Vite установлен как локальная зависимость разработки в вашем проекте, и вы настроили следующие npm-скрипты:

```json [package.json]
{
  "scripts": {
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

Важно отметить, что `vite preview` предназначен для предварительного просмотра сборки локально и не предназначен как продакшен-сервер.

::: tip ПРИМЕЧАНИЕ
Эти руководства предоставляют инструкции по выполнению статического развертывания вашего сайта Vite. Vite также поддерживает Server Side Rendering. SSR относится к фронтенд-фреймворкам, которые поддерживают запуск одного и того же приложения в Node.js, предварительный рендеринг в HTML и, наконец, гидратацию на клиенте. Ознакомьтесь с [Руководством по SSR](./ssr), чтобы узнать об этой функции. С другой стороны, если вы ищете интеграцию с традиционными серверными фреймворками, ознакомьтесь с [Руководством по интеграции с бэкендом](./backend-integration).
:::

## Сборка приложения

Вы можете запустить команду `npm run build` для сборки приложения.

```bash
$ npm run build
```

По умолчанию выходные данные сборки будут помещены в `dist`. Вы можете развернуть эту папку `dist` на любой предпочитаемой вами платформе.

### Тестирование приложения локально

После сборки приложения вы можете протестировать его локально, запустив команду `npm run preview`.

```bash
$ npm run preview
```

Команда `vite preview` запустит локальный статический веб-сервер, который обслуживает файлы из `dist` по адресу `http://localhost:4173`. Это простой способ проверить, хорошо ли выглядит продакшен-сборка в вашей локальной среде.

Вы можете настроить порт сервера, передав флаг `--port` в качестве аргумента.

```json [package.json]
{
  "scripts": {
    "preview": "vite preview --port 8080"
  }
}
```

Теперь команда `preview` запустит сервер по адресу `http://localhost:8080`.

## GitHub Pages

1. Установите правильный `base` в `vite.config.js`.

   Если вы развертываете на `https://<USERNAME>.github.io/` или на пользовательском домене через GitHub Pages (например, `www.example.com`), установите `base` в `'/'`. Альтернативно, вы можете удалить `base` из конфигурации, так как по умолчанию он равен `'/'`.

   Если вы развертываете на `https://<USERNAME>.github.io/<REPO>/` (например, ваш репозиторий находится по адресу `https://github.com/<USERNAME>/<REPO>`), то установите `base` в `'/<REPO>/'`.

2. Перейдите в настройки GitHub Pages в странице настроек репозитория и выберите источник развертывания как "GitHub Actions", это приведет вас к созданию рабочего процесса, который собирает и развертывает ваш проект, предоставляется пример рабочего процесса, который устанавливает зависимости и собирает с помощью npm:

   ```yml
   # Простой рабочий процесс для развертывания статического контента на GitHub Pages
   name: Deploy static content to Pages

   on:
     # Запускается при пушах в ветку по умолчанию
     push:
       branches: ['main']

     # Позволяет запускать этот рабочий процесс вручную из вкладки Actions
     workflow_dispatch:

   # Устанавливает разрешения GITHUB_TOKEN для развертывания на GitHub Pages
   permissions:
     contents: read
     pages: write
     id-token: write

   # Разрешает одно одновременное развертывание
   concurrency:
     group: 'pages'
     cancel-in-progress: true

   jobs:
     # Единое задание развертывания, так как мы просто развертываем
     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}
       runs-on: ubuntu-latest
       steps:
         - name: Checkout
           uses: actions/checkout@v4
         - name: Set up Node
           uses: actions/setup-node@v4
           with:
             node-version: 20
             cache: 'npm'
         - name: Install dependencies
           run: npm ci
         - name: Build
           run: npm run build
         - name: Setup Pages
           uses: actions/configure-pages@v4
         - name: Upload artifact
           uses: actions/upload-pages-artifact@v3
           with:
             # Загрузить папку dist
             path: './dist'
         - name: Deploy to GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v4
   ```

## GitLab Pages и GitLab CI

1. Установите правильный `base` в `vite.config.js`.

   Если вы развертываете на `https://<USERNAME or GROUP>.gitlab.io/`, вы можете опустить `base`, так как по умолчанию он равен `'/'`.

   Если вы развертываете на `https://<USERNAME or GROUP>.gitlab.io/<REPO>/`, например, ваш репозиторий находится по адресу `https://gitlab.com/<USERNAME>/<REPO>`, то установите `base` в `'/<REPO>/'`.

2. Создайте файл `.gitlab-ci.yml` в корне вашего проекта со следующим содержимым. Это будет собирать и развертывать ваш сайт каждый раз, когда вы вносите изменения в ваш контент:

   ```yaml [.gitlab-ci.yml]
   image: node:16.5.0
   pages:
     stage: deploy
     cache:
       key:
         files:
           - package-lock.json
         prefix: npm
       paths:
         - node_modules/
     script:
       - npm install
       - npm run build
       - cp -a dist/. public/
     artifacts:
       paths:
         - public
     rules:
       - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
   ```

## Netlify

### Netlify CLI

1. Установите [Netlify CLI](https://cli.netlify.com/).
2. Создайте новый сайт с помощью `ntl init`.
3. Разверните с помощью `ntl deploy`.

```bash
# Установить Netlify CLI
$ npm install -g netlify-cli

# Создать новый сайт в Netlify
$ ntl init

# Развернуть на уникальном URL предварительного просмотра
$ ntl deploy
```

Netlify CLI поделится с вами URL предварительного просмотра для проверки. Когда вы готовы перейти в продакшен, используйте флаг `prod`:

```bash
# Развернуть сайт в продакшен
$ ntl deploy --prod
```

### Netlify с Git

1. Отправьте ваш код в git-репозиторий (GitHub, GitLab, BitBucket, Azure DevOps).
2. [Импортируйте проект](https://app.netlify.com/start) в Netlify.
3. Выберите ветку, выходную директорию и настройте переменные окружения, если это необходимо.
4. Нажмите на **Deploy**.
5. Ваше приложение Vite развернуто!

После того, как ваш проект импортирован и развернут, все последующие пуши в ветки, отличные от продакшен-ветки, вместе с pull-запросами будут генерировать [Предварительные развертывания](https://docs.netlify.com/site-deploys/deploy-previews/), а все изменения, внесенные в Продакшен-ветку (обычно "main"), приведут к [Продакшен-развертыванию](https://docs.netlify.com/site-deploys/overview/#definitions).

## Vercel

### Vercel CLI

1. Установите [Vercel CLI](https://vercel.com/cli) и запустите `vercel` для развертывания.
2. Vercel обнаружит, что вы используете Vite, и включит правильные настройки для вашего развертывания.
3. Ваше приложение развернуто! (например, [vite-vue-template.vercel.app](https://vite-vue-template.vercel.app/))

```bash
$ npm i -g vercel
$ vercel init vite
Vercel CLI
> Success! Initialized "vite" example in ~/your-folder.
- To deploy, `cd vite` and run `vercel`.
```

### Vercel для Git

1. Отправьте ваш код в ваш git-репозиторий (GitHub, GitLab, Bitbucket).
2. [Импортируйте ваш проект Vite](https://vercel.com/new) в Vercel.
3. Vercel обнаружит, что вы используете Vite, и включит правильные настройки для вашего развертывания.
4. Ваше приложение развернуто! (например, [vite-vue-template.vercel.app](https://vite-vue-template.vercel.app/))

После того, как ваш проект импортирован и развернут, все последующие пуши в ветки будут генерировать [Предварительные развертывания](https://vercel.com/docs/concepts/deployments/environments#preview), а все изменения, внесенные в Продакшен-ветку (обычно "main"), приведут к [Продакшен-развертыванию](https://vercel.com/docs/concepts/deployments/environments#production).

Узнайте больше о [Интеграции с Git в Vercel](https://vercel.com/docs/concepts/git).

## Cloudflare Pages

### Cloudflare Pages через Wrangler

1. Установите [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/get-started/).
2. Аутентифицируйте Wrangler с вашей учетной записью Cloudflare с помощью `wrangler login`.
3. Запустите вашу команду сборки.
4. Разверните с помощью `npx wrangler pages deploy dist`.

```bash
# Установить Wrangler CLI
$ npm install -g wrangler

# Войти в учетную запись Cloudflare из CLI
$ wrangler login

# Запустить вашу команду сборки
$ npm run build

# Создать новое развертывание
$ npx wrangler pages deploy dist
```

После загрузки ваших ресурсов Wrangler предоставит вам URL предварительного просмотра для проверки вашего сайта. Когда вы войдете в панель управления Cloudflare Pages, вы увидите ваш новый проект.

### Cloudflare Pages с Git

1. Отправьте ваш код в ваш git-репозиторий (GitHub, GitLab).
2. Войдите в панель управления Cloudflare и выберите вашу учетную запись в **Account Home** > **Pages**.
3. Выберите **Create a new Project** и опцию **Connect Git**.
4. Выберите git-проект, который вы хотите развернуть, и нажмите **Begin setup**
5. Выберите соответствующий пресет фреймворка в настройках сборки в зависимости от выбранного вами фреймворка Vite.
6. Затем сохраните и разверните!
7. Ваше приложение развернуто! (например, `https://<PROJECTNAME>.pages.dev/`)

После того, как ваш проект импортирован и развернут, все последующие пуши в ветки будут генерировать [Предварительные развертывания](https://developers.cloudflare.com/pages/platform/preview-deployments/), если не указано иное в ваших [управлениях сборкой веток](https://developers.cloudflare.com/pages/platform/branch-build-controls/). Все изменения в Продакшен-ветке (обычно "main") приведут к Продакшен-развертыванию.

## Google Firebase

1. Убедитесь, что у вас установлен [firebase-tools](https://www.npmjs.com/package/firebase-tools).

2. Создайте файлы `firebase.json` и `.firebaserc` в корне вашего проекта со следующим содержимым:

   ```json [firebase.json]
   {
     "hosting": {
       "public": "dist",
       "ignore": [],
       "rewrites": [
         {
           "source": "**",
           "destination": "/index.html"
         }
       ]
     }
   }
   ```

   ```js [.firebaserc]
   {
     "projects": {
       "default": "<ВАШ_FIREBASE_ID>"
     }
   }
   ```

3. После выполнения `npm run build`, разверните с помощью команды `firebase deploy`.

## Surge

1. Сначала установите [surge](https://www.npmjs.com/package/surge), если еще не установлен.

2. Запустите `npm run build`.

3. Разверните на surge, введя `surge dist`.

Вы также можете развернуть на [пользовательском домене](http://surge.sh/help/adding-a-custom-domain), добавив `surge dist yourdomain.com`.

## Azure Static Web Apps

Вы можете быстро развернуть ваше Vite приложение с помощью сервиса Microsoft Azure [Static Web Apps](https://aka.ms/staticwebapps). Вам потребуется:

- Аккаунт Azure и ключ подписки. Вы можете создать [бесплатный аккаунт Azure здесь](https://azure.microsoft.com/free).
- Ваш код приложения, отправленный в [GitHub](https://github.com).
- Расширение [SWA Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps) в [Visual Studio Code](https://code.visualstudio.com).

Установите расширение в VS Code и перейдите в корень вашего приложения. Откройте расширение Static Web Apps, войдите в Azure и нажмите кнопку '+' для создания нового Static Web App. Вам будет предложено указать, какой ключ подписки использовать.

Следуйте мастеру, запущенному расширением, чтобы дать вашему приложению имя, выбрать пресет фреймворка и указать корень приложения (обычно `/`) и расположение собранных файлов `/dist`. Мастер создаст GitHub action в вашем репозитории в папке `.github`.

Action будет работать над развертыванием вашего приложения (следите за его прогрессом во вкладке Actions вашего репозитория) и, когда успешно завершится, вы сможете просмотреть ваше приложение по адресу, предоставленному в окне прогресса расширения, нажав кнопку 'Browse Website', которая появится после выполнения GitHub action.

## Render

Вы можете развернуть ваше Vite приложение как Static Site на [Render](https://render.com/).

1. Создайте [аккаунт Render](https://dashboard.render.com/register).

2. В [Панели управления](https://dashboard.render.com/) нажмите кнопку **New** и выберите **Static Site**.

3. Подключите ваш аккаунт GitHub/GitLab или используйте публичный репозиторий.

4. Укажите имя проекта и ветку.

   - **Build Command**: `npm install && npm run build`
   - **Publish Directory**: `dist`

5. Нажмите **Create Static Site**.

   Ваше приложение должно быть развернуто по адресу `https://<ИМЯ_ПРОЕКТА>.onrender.com/`.

По умолчанию любой новый коммит, отправленный в указанную ветку, автоматически запустит новое развертывание. [Auto-Deploy](https://render.com/docs/deploys#toggling-auto-deploy-for-a-service) можно настроить в настройках проекта.

Вы также можете добавить [пользовательский домен](https://render.com/docs/custom-domains) к вашему проекту.

<!--
  ПРИМЕЧАНИЕ: Разделы ниже зарезервированы для дополнительных платформ развертывания, не перечисленных выше.
  Не стесняйтесь отправить PR, который добавляет новый раздел со ссылкой на руководство по развертыванию вашей платформы,
  при условии, что оно соответствует следующим критериям:

  1. Пользователи должны иметь возможность развернуть свой сайт бесплатно.
  2. Бесплатные предложения должны размещать сайт бессрочно и не ограничиваться по времени.
     Предложение ограниченного количества вычислительных ресурсов или количества сайтов взамен допустимо.
  3. Связанные руководства не должны содержать вредоносного контента.

  Команда Vite может изменять критерии и проверять текущий список время от времени.
  Если раздел удаляется, мы свяжемся с авторами оригинального PR перед этим.
-->

## Flightcontrol

Разверните ваш статический сайт с помощью [Flightcontrol](https://www.flightcontrol.dev/?ref=docs-vite), следуя этим [инструкциям](https://www.flightcontrol.dev/docs/reference/examples/vite?ref=docs-vite).

## Kinsta Static Site Hosting

Разверните ваш статический сайт с помощью [Kinsta](https://kinsta.com/static-site-hosting/), следуя этим [инструкциям](https://kinsta.com/docs/react-vite-example/).

## xmit Static Site Hosting

Разверните ваш статический сайт с помощью [xmit](https://xmit.co), следуя этому [руководству](https://xmit.dev/posts/vite-quickstart/).
