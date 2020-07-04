# Aula 03

## Links Úteis

* [Projeto Final](https://github.com/juninmd/unifacef-react-typescript)
* [Covid19](https://documenter.getpostman.com/view/10808728/SzS8rjbc?version=latest)

## Ementa

* Ajustando variáveis de ambiente do projeto
* Push Notification com OneSignal
* Criação de componentes personalizados React
* Pesquisa de Cep
* Pesquisa de Perfil Github
* Utilizando API **Corona**
* Autenticação
* Testes
  * Jest

## Envs

Coloque todas as variáveis do seu projeto dentro do arquivo .env,
é só pesquisar em tudo que criamos no projeto que comece com

```env
process.env.REACT_APP_*
```

Ficando assim por exemplo

```.env
REACT_APP_SENTRY_DSN=<seu token do sentry>
REACT_APP_CEP_URL=http://cep.republicavirtual.com.br/web_cep.php
REACT_APP_ECONOMIA_URL=https://economia.awesomeapi.com.br/json/all
REACT_APP_GITHUB_URL=https://api.github.com
REACT_APP_STAR_WARS_URL=https://star-wars-api-unifacef.herokuapp.com
REACT_APP_CORONA_URL=https://api.covid19api.com
REACT_APP_ONE_SIGNAL=<seu token do one signal>
```

Vamos apagar o .env.local

## Ajustando APIS

> economy.api.ts

```tsx
import axios from 'axios';
import { configs } from '../configs';

export const getPrice = () => {
  return axios.request({ url: configs.apis.economia });
}
```

> star-wars.api.ts

```tsx
import axios from 'axios';
import { configs } from '../configs';

const baseURL = configs.apis.starWars;

export const getFilms = () => {
  return axios.request({ baseURL, url: 'films' })
}

export const getFilmById = (id: number) => {
  return axios.request({ baseURL, url: `films/${id}` })
}
```

> src/configs/index.ts

```ts
export const configs = {
  apis: {
    economia: process.env.REACT_APP_ECONOMIA_URL,
    starWars: process.env.REACT_APP_STAR_WARS_BASE_URL
  },
  sentry: process.env.REACT_APP_SENTRY_DSN
}
```

> src/plugins/sentry.plugin.ts

```ts
import { configureScope, init } from '@sentry/browser'
import { configs } from '../configs';

(() => {
  // Desativa o plugin localhost
  if (window.location.hostname === 'localhost' ||
    window.location.hostname === '127.0.0.1') {
    return;
  }

  const { sentry } = configs;

  init({ dsn: sentry });

  configureScope(scope => {
  })
})();
```

## OneSignal

Vamos criar uma conta no OneSignal, que será responsável por utilizar os push notifications do site

```text
https://onesignal.com/
```

![imagem](./imagens/onesignal.jpg)

> src/plugins/one-signal.plugin.ts

```ts
import OneSignal from 'react-onesignal';
import { configs } from '../configs';

const options = { autoRegister: true, autoResubscribe: true, notifyButton: { enable: true } }

OneSignal.initialize(configs.onesignal, options);

try {
    OneSignal.registerForPushNotifications();
} catch (error) {
}
```
