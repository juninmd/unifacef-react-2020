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

## OneSignal

Vamos criar uma conta no OneSignal, que será responsável por utilizar os push notifications do site

```text
https://onesignal.com/
```

![imagem](./imagens/onesignal.jpg)