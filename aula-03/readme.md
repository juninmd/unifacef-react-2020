# Aula 03

## Links Úteis

* [Projeto Final](https://github.com/juninmd/unifacef-react-typescript)
* [Covid19](https://documenter.getpostman.com/view/10808728/SzS8rjbc?version=latest)
* [CEP](https://www.republicavirtual.com.br/cep/exemplos.php)
* [JSON2TS](http://www.json2ts.com/)

## Recursos

<https://react.semantic-ui.com/images/wireframe/short-paragraph.png>

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

> src/components/cep/index.tsx

```tsx
import React, { useState, useEffect } from 'react';
import { getZipCode, GetZipCode } from '../../apis/correios.api';
import { Card, Segment, Loader, Image, Icon } from 'semantic-ui-react';
import Swal from 'sweetalert2';

interface Props {
  zipCode?: number;
}

export default function Cep(props: Props) {

  const [address, setAddress] = useState<GetZipCode>();

  useEffect(() => {

    const addressBlank: GetZipCode = {
      bairro: '',
      cidade: '',
      logradouro: '',
      resultado: '',
      resultado_txt: '',
      tipo_logradouro: '',
      uf: ''
    }

    async function getNewAddress(zipCode: number) {
      try {
        const response = await getZipCode(zipCode);
        setAddress(response.data);
      } catch (error) {
        Swal.fire(error.message, '', 'warning')
        setAddress(addressBlank);
      }
    }

    if (props.zipCode?.toString().length === 8) {
      getNewAddress(props.zipCode);
    } else {
      setAddress(undefined);
    }
  }, [props])

  if (!address && !props.zipCode) {
    return <Segment><p>Informe um número de CEP</p></Segment>
  }

  if (!address?.cidade) {
    return <Segment>
      <Loader active={true} />
      <Image src='https://react.semantic-ui.com/images/wireframe/short-paragraph.png' />
    </Segment>
  }

  return (
    <Card>
      <Card.Content>
        <Card.Header>{address.cidade} - {address.uf}</Card.Header>
        <Card.Meta>{address.bairro}</Card.Meta>
        <Card.Description>
          {address.tipo_logradouro} {address.logradouro}
        </Card.Description>
      </Card.Content>
      <Card.Content extra={true}>
        <Icon name='tree' />{props.zipCode}
      </Card.Content>
    </Card>
  )

}
```

> src/containers/register/store.ts

```tsx
import { observable, action } from 'mobx';
import { assign } from '../../utils/object.util';

export default class RegisterStore {
  @observable zipcode?: number;
  @observable github?: string;

  @action handleForm = (event: any, select?: any) => {
    const { name, value } = select || event.target;
    assign(this, name, value);
  }
}

const register = new RegisterStore();
export { register };
```

> src/containers/register/index.tsx

```tsx
import * as React from 'react';
import { Container, Grid, Header, Form } from 'semantic-ui-react';
import { inject, observer } from 'mobx-react';
import NewRouterStore from '../../mobx/router.store';
import RegisterStore from './store';
import Cep from '../../components/cep';

interface Props {
  router: NewRouterStore;
  register: RegisterStore;
}

@inject('router', 'register')
@observer
export default class Register extends React.Component<Props> {

  render() {
    const { zipcode, handleForm } = this.props.register;

    return (
      <Container>
        <Grid divided='vertically'>
          <Grid.Row columns={2}>
            <Grid.Column>
              <Header color='blue' as='h2'>
                <Header.Content>
                  Register
                  <Header.Subheader>Simulação de formulário</Header.Subheader>
                </Header.Content>
              </Header>
            </Grid.Column>
          </Grid.Row>
        </Grid>
        <Form>
          <Form.Group widths='equal'>
            <Form.Field>
              <label>Informe o CEP:</label>
              <input value={zipcode || ''} maxLength={8} name='zipcode' onChange={handleForm} placeholder='Ex 14405123' />
            </Form.Field>
            <Form.Field>
              <Cep zipCode={zipcode} />
            </Form.Field>
          </Form.Group>
        </Form>
      </Container>
    )
  }
}
```

> src/components/github/index.tsx

```tsx
import React, { useState, useEffect } from 'react';
import { getUser, GetGithub } from '../../apis/github.api';
import { Card, Segment, Loader, Image, Icon } from 'semantic-ui-react';

interface Props {
  userName?: string;
}

export default function Github(props: Props) {

  const [profile, setProfile] = useState<GetGithub>();

  useEffect(() => {

    async function getProfile(userName: string) {
      try {
        const response = await getUser(userName);
        setProfile(response.data);
      } catch (error) {
        setProfile(undefined);
      }
    }

    if (props.userName) {
      getProfile(props.userName)
    } else {
      setProfile(undefined);
    }
  }, [props])

  if (!props.userName) {
    return <Segment>
      <p>Informe um perfil</p>
    </Segment>
  }

  if (!profile) {
    return <Segment>
      <Loader active={true} />
      <Image src='https://react.semantic-ui.com/images/wireframe/short-paragraph.png' />
    </Segment>
  }

  const openGithub = (url: string) => {
    window.open(url, 'blank');
  }

  return (
    <Card onClick={() => openGithub(profile.html_url)}>
      <Image size='small' src={profile.avatar_url} wrapped ui={false} />
      <Card.Content>
        <Card.Header>{profile.name}</Card.Header>
        <Card.Meta>{profile.login}</Card.Meta>
        <Card.Description>
          <p>{profile.bio}</p>
          <p>{profile.blog}</p>
          <p>{profile.company}</p>
          <p>{profile.email}</p>
          <p>@{profile.twitter_username}</p>
        </Card.Description>
      </Card.Content>
      <Card.Content extra={true}>
        <Icon name='tree' /> {profile.location}
      </Card.Content>
    </Card>
  )
}
```