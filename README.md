<h1 align="center">Cypress Dicas e Truques</h1>
<p align="center">Uma série de dicas e truques sobre Cypress, traduzido e adaptado do post "Cypress Tips and Tricks" do Gleb Bahmutov </p>

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fsamlucax%2Fcypress-dicas-e-truques&count_bg=%23FBCA16&title_bg=%232C2C2C&icon=skyliner.svg&icon_color=%23FBCA16&title=views&edge_flat=false)](https://hits.seeyoufarm.com)

- [Artigo original](https://glebbahmutov.com/blog/cypress-tips-and-tricks/ "Artigo original")
- [Curso Gratuito de Cypress](https://youtube.com/playlist?list=PLnUo-Rbc3jjztMO4K8b-px4NE-630VNKY "Curso Gratuito de Cypress")

_Favorite este repositório e compartilhe esse material para ajudar outras pessoas_

# Dicas e Truques
- [Leia a documentação](#leia-a-documenta%C3%A7%C3%A3o "Leia a documentação")
- [Execute o Cypress em sua Integração Contínua](#execute-o-cypress-em-sua-integra%C3%A7%C3%A3o-cont%C3%ADnua "Execute o Cypress em sua Integração Contínua")
- [Grave vídeos de sucesso e falha](#grave-v%C3%ADdeos-de-sucesso-e-falha "Grave vídeos de sucesso e falha")
- [Mova o que for código comum para o pacote de utilitário](#mova-o-que-for-c%C3%B3digo-comum-para-o-pacote-de-utilit%C3%A1rio "Mova o que for código comum para o pacote de utilitário")
- [Separe os testes em pacotes](#separe-os-testes-em-pacotes "Separe os testes em pacotes")
- [Faça com que os erros Javascript sejam úteis](#fa%C3%A7a-com-que-os-erros-de-javascript-sejam-%C3%BAteis "Faça com que os erros Javascript sejam úteis")
- [Use nomes de teste ao criar dados](#use-nomes-de-teste-ao-criar-dados "Use nomes de teste ao criar dados")
- [Obtenha o status do teste](#obtenha-o-status-do-teste "Obtenha o status do teste")
- [Explore o ambiente com o debugger](#explore-o-ambiente-com-o-debugger "Explore o ambiente com o debugger")
- [Execute os arquivos de teste um por um](#execute-os-arquivos-de-teste-um-por-um "Execute os arquivos de teste um por um")

---

_Em construção, próximas dicas e truques serão publicados ao longo da semana_

#COMPARTILHE

---

Já faz algum tempo que usamos o Cypress para verificar nossos sites e aplicativos web. 

Aqui estão algumas dicas e truques que aprendemos.

## Leia a documentação

O Cypress tem uma documentação excelente em https://docs.cypress.io/, incluindo [exemplos](https://docs.cypress.io/docs/all-example-apps "exemplos"), [guias](https://on.cypress.io/guides "guias"), [vídeos](https://www.cypress.io/explore/ "vídeos") e até mesmo um [canal de bate-papo](https://gitter.im/cypress-io/cypress "canal de bate-papo"). Também existe a funcionalidade de "Pesquisar" (Ctrl/CMD + K) que ajudar a encontrar rapidamente as documentações necessárias. A pesquisa é muito boa e cobre os documentos, as postagens do blog e os exemplos.

Recentemente, alguns exemplos comuns foram centralizados em um [único lugar](https://github.com/cypress-io/cypress-example-recipes "único lugar"). Se você ainda tiver problemas com o Cypress, pode pesquisar pelos [itens](https://github.com/cypress-io/cypress/issues "problemas") abertos e fechados (originalmente chamados de issues).

Você também pode ler outros posts sobre Cypress em [tags/cypress](https://glebbahmutov.com/blog/tags/cypress/ "tags/cypress").

## Execute o Cypress em sua Integração Contínua

Contanto que o projeto tenha uma *key*, você pode executar os testes em suas própria [integração contínua](https://on.cypress.io/continuous-integration "integração contínua").
Descobri que a execução dentro do Docker é a mais fácil, e até criei uma [imagem Docker não oficial](https://hub.docker.com/r/bahmutov/cypress-image/ "imagem Docker não oficial") com todas as dependências.

**Extra**: entenda como usar uma [imagem Docker oficial com o Cypress](https://docs.cypress.io/examples/examples/docker "imagem Docker oficial com o Cypress")

Observe que o `cypress ci` ... carregará vídeos gravados e imagens (se capturá-los estiver habilitado, link aqui  enabled) para a nuvem Cypress, enquanto o  `cypress run` … não fará.

**Extra**: entenda a diferença atualizada dos [comandos de execução](https://docs.cypress.io/guides/guides/command-line#Commands "comandos de execução") do Cypress

## Grave vídeos de sucesso e falha

Quando a equipe do Cypress lançou a funcionalidade de captura de vídeo na versão [0.17.11](https://github.com/cypress-io/cypress/wiki/changelog#01711-11162016 "0.17.11"), logo ela se tornou a minha funcionalidade favorita. Eu configurei o [GitLab CI](https://about.gitlab.com/gitlab-ci/ "GitLab CI") para guardar os artefatos de execuções de teste "mal sucedidas" por 1 semana, enquanto mantive os vídeos de execuções de teste "bem-sucedidas" por apenas 3 dias.

``` yaml
artifacts:
  when: on_failure
  expire_in: '1 week'
  untracked: true
  paths:
    - cypress/videos
    - cypress/screenshots
artifacts:
  when: on_success
  expire_in: '3 days'
  untracked: true
  paths:
    - cypress/screenshots
```

Sempre que um teste falha, assisto ao vídeo da falha lado a lado com o vídeo da última execução de teste bem-sucedida. As diferenças nas execuções do teste são rapidamente descobertas.

## Mova o que for código comum para o pacote de utilitário

O código de teste deve ser projetado assim como o código de produção. Para evitar a duplicação de utilitários, aconselhamos mover tudo o que for de uso comum para seus próprios módulos NPM. Por exemplo, usamos estilos CSS locais com nomes de classe aleatórios em nosso código da web.

```html
<div class="table-1b4a2 people-5ee98">
...
</div>
```
Isso torna os seletores muito longos, pois precisamos usar prefixos.

```js
cy.get('[class*=people-]') // FEIO!
```

Ao criar uma pequena função auxiliar, aliviamos as dores de cabeça do seletor.

```js
import {classPrefix} from '@team/cypress-utils/css-names'
// classPrefix('foo') returns "[class*=foo-]"
cy.get(classPrefix('people'))
```

O código de teste se tornou muito mais legível.

## Separe os testes em pacotes

Usar um único arquivo `cypress/integration/spec.js` para testar uma webapp grande, rapidamente se torna difícil e demorado. Separamos nosso código em vários arquivos. Além disso, mantemos os "arquivos do código fonte" na pasta `src`  e construímos os vários pacotes em `cypress/integration` usando a ferramenta [kensho/multi-cypress](https://github.com/kensho/multi-cypress "kensho/multi-cypress").

A ferramenta `multi-cypress` assume que o Docker e o GitLab CI são usados para executar os testes. Cada teste estará dentro de seu próprio "contexto", portanto, 10 arquivos de testes podem ser executados de uma só vez (assumindo que pelo menos 10 executores do GitLab CI estejam disponíveis).

A propósito, o comando para executar um arquivo específico é `cypress run --spec "spec/filename.js"`

## Faça com que os erros de JavaScript sejam úteis

Como eu disse antes, o verdadeiro valor de um teste não é quando ele passa, mas quando ele falha. Além disso, mesmo um teste de aprovação pode gerar erros do lado do cliente que não são travamentos e não falham no teste. Por exemplo, costumamos usar o serviço de monitoramento de exceção [Sentry](https://sentry.io/ "Sentry"), onde encaminhamos todos os erros do lado do cliente.

Se ocorrer um erro do lado do cliente durante a execução do teste E2E Cypress, precisamos deste contexto adicional: qual teste está sendo executado, quais etapas já foram concluídas antes de o erro ser relatado, etc.

Carregando uma pequena biblioteca [send-test-info](https://github.com/bahmutov/send-test-info "send-test-info") antes de cada teste, podemos capturar o nome do teste, seu nome de arquivo de especificação e até mesmo os logs de teste do Cypress de forma estruturada. Esta informação enviada para o Sentry a cada falha. Por exemplo, a exceção abaixo mostra o nome do teste quando o erro foi relatado.

<imagem></imagem>

Da mesma forma, os passos do teste são registrados de forma estrutural e enviadas com a exceção, permitindo que qualquer desenvolvedor tenha uma noção rápida de quando o erro ocorreu durante a execução do teste.

<imagem></imagem>


## Use nomes de teste ao criar dados

Frequentemente, como parte do teste E2E, criamos itens / usuários / cadastros. Eu gosto de colocar o título completo do teste nos dados criados para ver facilmente qual teste criou qual dado. Dentro do teste, podemos obter rapidamente o título completo (que inclui o título completo do pai e o nome do teste atual) a partir do contexto.

Também gosto de criar números aleatórios para adicionar uma diferença a mais

```js
const uuid = () => Cypress._.random(0, 1e6)
describe('foo', () => {
  // note que precisamos usar "function" ao invés da arrow function
  // para podermos usar o "this" que vem do context
  it('bar', function () {
    const id = uuid()
    const testName = this.test.fullTitle()
    const name = `test name: ${testName} - ${id}`
    // name será "test name: foo bar - <random>"
    makeItem({name}) // usa o nome do teste para criar um item único
  })
})
```

## Obtenha o status do teste

Se você usar  `function () {}` como o corpo do teste, ou um corpo de um *hook*, você pode encontrar muitas propriedades interessantes no objeto `this`. Por exemplo, você pode ver o *status* do teste: passou ou falhou. Para ver todos eles, coloque a palavra-chave `debugger` e execute o teste com DevTools aberto.

```js
afterEach(function () {
  console.log(this)
  debugger
})
```

Há um objeto para o teste e para o *hook* atual

<imagem></imagem>

Informações no objeto de contexto de teste  ↑

O objeto  `this.currentTest` contém o status de todo o teste

<imagem></imagem>

Verifique se o teste passou ou falhou  ↑


## Explore o ambiente com o debugger

Você pode pausar a execução do teste usando a palavra-chave `debugger`. Certifique-se de que o DevTools esteja aberto!

```js
it('bar', function () {
  debugger
  // explore "this" context
})
```

## Execute os arquivos de teste um por um
Se você separar os testes do [Cypress](https://www.cypress.io/ "Cypress") E2E em vários arquivos de *spec*, logo terá muitos arquivos. Se você precisar executar os scripts um por um na linha de comando, eu escrevi um pequeno utilitário [run-each-cypress-spec](https://github.com/bahmutov/run-each-cypress-spec "run-each-cypress-spec")

```sh
npm install -g run-each-cypress-spec
run-specs
...
running cypress/integration/a-spec.js
Started video recording: /Users/gleb/cypress/videos/p259n.mp4
...
running cypress/integration/b-spec.js
Started video recording: /Users/gleb/cypress/videos/62er1.mp4

```

Se você precisa de variáveis de ambiente (como urls, senhas), pode injetá-las facilmente usando [as-a](https://github.com/bahmutov/as-a "as-a")).

```sh
as-a cy run-specs
```
