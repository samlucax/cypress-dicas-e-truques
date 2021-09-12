<h1 align="center">Cypress Dicas e Truques</h1>
<p align="center">Uma série de dicas e truques sobre Cypress, traduzido e adaptado do post "Cypress Tips and Tricks" do Gleb Bahmutov </p>

[Artigo original](https://glebbahmutov.com/blog/cypress-tips-and-tricks/ "Artigo original")
[Curso Gratuito de Cypress](https://youtube.com/playlist?list=PLnUo-Rbc3jjztMO4K8b-px4NE-630VNKY "Curso Gratuito de Cypress"): 

_Favorite este repositório e compartilhe esse material para ajudar outras pessoas_

# Dicas e Truques
- Leia a documentação
- Execute o Cypress em sua Integração Contínua
- Grave vídeos de sucesso e falha
- Mova o que for código comum para o pacote de utilitário
- Separe os testes em pacotes

---

Já faz algum tempo que usamos o Cypress para verificar nossos sites e aplicativos web. Aqui estão algumas dicas e truques que aprendemos.

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
