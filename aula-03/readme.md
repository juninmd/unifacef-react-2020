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

> src/containers/register/index.tsx

```tsx
import * as React from 'react';
import { Container, Grid, Header, Form } from 'semantic-ui-react';
import { inject, observer } from 'mobx-react';
import NewRouterStore from '../../mobx/router.store';
import RegisterStore from './store';
import Cep from '../../components/cep';
import Github from '../../components/github';

interface Props {
  router: NewRouterStore;
  register: RegisterStore;
}

@inject('router', 'register')
@observer
export default class Register extends React.Component<Props> {

  render() {
    const { zipcode, handleForm, github } = this.props.register;

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
          <Form.Group widths='equal' style={{ alignItems: 'flex-end' }} >
            <Form.Field>
              <label>Informe o CEP:</label>
              <input value={zipcode || ''} maxLength={8} name='zipcode' onChange={handleForm} placeholder='Ex 14405123' />
            </Form.Field>
            <Form.Field>
              <Cep zipCode={zipcode} />
            </Form.Field>
          </Form.Group>
          <Form.Group widths='equal' style={{ alignItems: 'flex-end' }} >
            <Form.Field>
              <label>Informe o seu github:</label>
              <input value={github || ''} maxLength={100} name='github' onChange={handleForm} placeholder='Ex juninmd' />
            </Form.Field>
            <Form.Field>
              <Github userName={github} />
            </Form.Field>
          </Form.Group>
        </Form>
      </Container>
    )
  }
}
```

> src/apis/github.api.ts

```ts
import { configs } from '../configs';
import axios from 'axios';

const baseURL = configs.apis.github;

export interface GetGithub {
  login: string;
  id: number;
  node_id: string;
  avatar_url: string;
  gravatar_id: string;
  url: string;
  html_url: string;
  followers_url: string;
  following_url: string;
  gists_url: string;
  starred_url: string;
  subscriptions_url: string;
  organizations_url: string;
  repos_url: string;
  events_url: string;
  received_events_url: string;
  type: string;
  site_admin: boolean;
  name: string;
  company?: any;
  blog: string;
  location: string;
  email?: any;
  hireable?: any;
  bio?: any;
  twitter_username?: any;
  public_repos: number;
  public_gists: number;
  followers: number;
  following: number;
  created_at: Date;
  updated_at: Date;
}

export const getUser = (userName: string) => {
  return axios.request<GetGithub>({ baseURL, url: `users/${userName}` })
}
```

## Corrigir problema de rotas

Criar arquivo em

> public/_redirects

```text
/* /index.html 200
```

> src/apis/corona.api.ts

```ts
import axios from 'axios';
import { configs } from '../configs';

const baseURL = configs.apis.corona;

export interface IGlobal {
  NewConfirmed: number;
  TotalConfirmed: number;
  NewDeaths: number;
  TotalDeaths: number;
  NewRecovered: number;
  TotalRecovered: number;
}

export interface ICountry {
  Country: string;
  CountryCode: string;
  Slug: string;
  NewConfirmed: number;
  TotalConfirmed: number;
  NewDeaths: number;
  TotalDeaths: number;
  NewRecovered: number;
  TotalRecovered: number;
  Date: Date;
}

export interface ISummary {
  Global: IGlobal;
  Countries: ICountry[];
  Date: Date;
}

export const getSummary = () => {
  return axios.request<ISummary>({ baseURL, url: 'summary' })
}

export interface ICountryB {
  Country: string;
  Slug: string;
  ISO2: string;
}

export const getCountries = () => {
  return axios.request<ICountryB[]>({ baseURL, url: 'countries' });
}
```

> src/containers/store.ts

```ts
import { action, observable } from 'mobx';
import { getCountries, getSummary, ISummary, ICountryB } from '../../apis/corona.api';
import { assign } from '../../utils/object.util';

export default class CoronaStore {

  @observable countryCode: string = '';

  @observable summary?: ISummary;

  @observable countries: ICountryB[] = [];

  @action handleForm = (event: any, select?: any) => {
    const { name, value } = select || event.target;
    assign(this, name, value);
  }

  @action getCountries = async () => {
    try {
      const { data } = await getCountries();
      this.countries = data;
    } catch (error) {
      this.countries = [];
    }
  }

  @action getSummary = async () => {
    try {
      const { data } = await getSummary();
      this.summary = data;
    } catch (error) {
      this.summary = undefined;
    }
  }
}
const corona = new CoronaStore();
export { corona };
```

!Danger! (código parcial)

> src/containers/corona/index.tsx

```tsx
import * as React from 'react';
import { Container, Grid, Header, Form, Dropdown, Card } from 'semantic-ui-react';
import { inject, observer } from 'mobx-react';
import NewRouterStore from '../../mobx/router.store';
import CoronaStore from './store';

interface Props {
  router: NewRouterStore;
  corona: CoronaStore
}
@inject('router', 'corona')
@observer
export default class Corona extends React.Component<Props>{


  async componentDidMount() {
    const { getCountries, getSummary } = this.props.corona;
    await Promise.all([getCountries(), getSummary()]);
  }

  render() {
    const { summary, countryCode, handleForm } = this.props.corona;

    return (
      <Container>
        <Grid divided='vertically'>
          <Grid.Row columns={2}>
            <Header color='blue' as='h2'>
              <Header.Content>
                Corona
              <Header.Subheader>Sumário Mundial {(new Date()).toLocaleDateString()}</Header.Subheader>
              </Header.Content>
            </Header>
          </Grid.Row>
        </Grid>

        <Grid>
          <Grid.Row>
            <Card.Group>

            </Card.Group>
          </Grid.Row>
        </Grid>

      </Container>
    )
  }

}
```

> src/components/summary/index.tsx

```tsx
import React from 'react';
import { Card, Feed, Grid } from 'semantic-ui-react';
import { IGlobal } from '../../apis/corona.api';

interface Props {
  global?: IGlobal;
}

export default function Summary(props: Props) {

  const { global } = props;

  if (!global) {
    return <></>
  }

  return (
    <Card color='blue' centered={true} fluid={true}>
      <Card.Content>
        <Card.Header>Mundial</Card.Header>
        <Card.Description>
          <Grid columns={2}>
            <Grid.Column>
              <Feed>
                <Feed.Event>
                  <Feed.Content>
                    <Feed.Date content='Casos' />
                    <Feed.Summary>{global.NewConfirmed}</Feed.Summary>
                  </Feed.Content>
                </Feed.Event>
                <Feed.Event>
                  <Feed.Content>
                    <Feed.Date content='Mortes' />
                    <Feed.Summary>{global.NewDeaths}</Feed.Summary>
                  </Feed.Content>
                </Feed.Event>
                <Feed.Event>
                  <Feed.Content>
                    <Feed.Date content='Recuperações' />
                    <Feed.Summary>{global.NewRecovered}</Feed.Summary>
                  </Feed.Content>
                </Feed.Event>
              </Feed>
            </Grid.Column>
            <Grid.Column>
              <Feed>
                <Feed.Event>
                  <Feed.Content>
                    <Feed.Date content='Total Casos' />
                    <Feed.Summary>{global.TotalConfirmed}</Feed.Summary>
                  </Feed.Content>
                </Feed.Event>
                <Feed.Event>
                  <Feed.Content>
                    <Feed.Date content='Total Mortes' />
                    <Feed.Summary>{global.TotalDeaths}</Feed.Summary>
                  </Feed.Content>
                </Feed.Event>
                <Feed.Event>
                  <Feed.Content>
                    <Feed.Date content='Total Recuperações' />
                    <Feed.Summary>{global.TotalRecovered}</Feed.Summary>
                  </Feed.Content>
                </Feed.Event>
              </Feed>
            </Grid.Column>
          </Grid>
        </Card.Description>
      </Card.Content>
    </Card>
  )
}
```
