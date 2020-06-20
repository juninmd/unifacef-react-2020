# Aula 02

## Links úteis

* [Projeto Final](https://github.com/juninmd/unifacef-react-typescript)
* [IndexDb](https://developer.mozilla.org/pt-BR/docs/Web/API/IndexedDB_API)
* [Cookie](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Cookie)
* [LocalStorage](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/Window.localStorage)
* [SessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
* [PWA](https://medium.com/@victorsilvamatheus.i/como-transformar-sua-aplica%C3%A7%C3%A3o-reactjs-em-um-pwa-e-ser%C3%A1-que-voc%C3%AA-deve-fazer-isso-567a8552c96d)
* <https://whatwebcando.today/>
* <https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=pt-BR>
* <https://developer.mozilla.org/pt-BR/docs/Web/API/IndexedDB_API/Usando_IndexedDB>
* <chrome://flags/#enable-desktop-pwas-without-extensions>
* <https://developer.mozilla.org/pt-BR/docs/Mozilla/Add-ons/WebExtensions/manifest.json>
* <https://www.google.com/chrome/>

## Ementa

* Star Wars
* Elementos Html5
* Storages
  * Cookies
  * LocalStorage
  * SessionStorage
  * Indexed DB
* PWA

## Star Wars

Crie uma nova pasta chamada star-wars dentro de apis

Crie um novo arquivo

```text
src/apis/star-wars.api.ts
```

```tsx
import axios from 'axios';

const baseURL = 'https://star-wars-api-unifacef.herokuapp.com'; // trocar por env de ambiente

export const getFilms = () => {
  return axios.request({ baseURL, url: 'films' })
}

export const getFilmById = (id: number) => {
  return axios.request({ baseURL, url: `films/${id}` })
}
```

Adicione as variáveis de ambiente nos seguintes arquivos

.env
.env.local

```env
REACT_APP_STAR_WARS_BASE_URL=https://star-wars-api-unifacef.herokuapp.com
```

Atualize a constante baseURL

```tsx
const baseURL = process.env.REACT_APP_STAR_WARS_BASE_URL;
```

Crie uma nova pasta chamada star-wars dentro de containers

Crie um novo arquivo

```text
src/containers/star-wars/store.ts
```

```tsx
import { action, observable } from 'mobx';
import { getFilms } from '../../apis/star-wars.api';

export default class StarWarsStore {
  @observable films: any[] = [];

  @action buildFilms = async () => {
    const { data } = await getFilms();
    this.films = data;
  }

}
const starWars = new StarWarsStore();
export { starWars };
```

Adicione a store na lista de stores

```text
src/mobx/index.ts
```

```tsx
import { router } from './router.store';
import { home } from '../containers/home/store';
import { combustivel } from '../containers/combustivel/store';
import { starWars } from '../containers/star-wars/store';

export {
  router,
  home,
  combustivel,
  starWars
};
```

Crie um novo arquivo

```text
src/containers/star-wars/index.tsx
```

```tsx
import * as React from 'react';
import { Container, Card, Grid, Header, Image } from 'semantic-ui-react';
import { inject, observer } from 'mobx-react';
import NewRouterStore from '../../mobx/router.store';
import StarWarsStore from './store';

interface Props {
  router: NewRouterStore;
  starWars: StarWarsStore;
}

@inject('router', 'starWars')
@observer
export default class StarWars extends React.Component<Props> {

  async componentDidMount() {
    const { buildFilms } = this.props.starWars;
    await buildFilms();
  }

  render() {

   const { films } = this.props.starWars;

    return (
      <Container>
        <Grid divided='vertically'>
          <Grid.Row columns={2}>
            <Grid.Column>
              <Header color='blue' as='h2'>
                <Header.Content>
                  Star Wars
                 <Header.Subheader>Lista de filmes</Header.Subheader>
                </Header.Content>
              </Header>
            </Grid.Column>
          </Grid.Row>
        </Grid>
        <Card.Group itemsPerRow={2}>
          {films.map((film, index) => {
            return (
              <Card key={index}>
                <Image src={film.photo} wrapped ui={false} size='small' />
                <Card.Content>
                  <Card.Meta>{film.title}</Card.Meta>
                  <Card.Description>Episode {film.episode_id.toString()}</Card.Description>
                </Card.Content>
              </Card>)
          })}
        </Card.Group>
      </Container>
    );
  }
}

```

Crie uma nova pasta chamada star-wars-details dentro de containers

Crie um novo arquivo

```text
src/containers/star-wars/store.ts
```

Corrigir componente de Loading

```text
src/apis/axios.ts
```

```ts
import { loadingOn, loadingOff } from '../components/loading';
import axios from 'axios';

axios.interceptors.request.use((config) => {
  loadingOn();
  return config;
}, (error) => {
  return Promise.reject(error);
});

axios.interceptors.response.use((config) => {
  loadingOff();
  return config;
}, (error) => {
  loadingOff();
  return Promise.reject(error);
});
```

---

## PWA

Vamos conhecer um pouco dos recursos de PWA

Instale a extensão LightHouse
<https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=pt-BR>
![imagem](./imagens/pwa.png)
  
Habilitem as opções no chrome

```text
chrome://flags/#enable-desktop-pwas-local-updating
chrome://flags/#enable-desktop-pwas-local-updating-throttle-persistence
chrome://flags/#enable-desktop-pwas-tab-strip
chrome://flags/#enable-desktop-pwas-without-extensions
chrome://flags/#enable-desktop-minimal-ui
```

O Que podemos fazer hoje?

<https://whatwebcando.today/>

## MobX

<https://chrome.google.com/webstore/detail/mobx-developer-tools/pfgnfdagidkfgccljigdamigbcnndkod>

## Import Cost

Extensão VsCode `wix.vscode-import-cost`

## Forks

Caso queira copiar o projeto final e sincronizar com seu fork

```bash
git remote add upstream https://github.com/juninmd/unifacef-react-typescript.git
git pull upstream
```

<https://app.onesignal.com/apps/new>