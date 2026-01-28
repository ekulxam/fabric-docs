---
title: Estrutura de Projeto
description: Uma visão geral da estrutura de um projeto de mod do Fabric.
authors:
  - IMB11
---

Esta página abordará a estrutura de um projeto de mod do Fabric e a finalidade de cada arquivo e pasta do projeto.

## `fabric.mod.json` {#fabric-mod-json}

O arquivo `fabric.mod.json` é o arquivo principal que descreve seu mod para o Fabric Loader. Ele contém informações como ID do mod, versão e dependências.

Os campos mais importantes no arquivo `fabric.mod.json` são:

- `id`: O ID do mod, que deve ser único.
- `name`: O nome do mod.
- `environment`: O ambiente em que seu mod é executado, como `client`, `server` ou `*` para ambos.
- `entrypoints`: Os pontos de entrada que seu mod fornece, como `main` ou `client`.
- `depends`: Os mods dos quais seu mod depende.
- `mixins`: Os mixins que seu mod oferece.

You can see an example `fabric.mod.json` file below - this is the `fabric.mod.json` file for the mod that powers this documentation site.

:::details `fabric.mod.json` of the Example Mod

@[code lang=json](@/reference/latest/src/main/resources/fabric.mod.json)

:::

## Pontos de Entrada {#entrypoints}

Como mencionado anteriormente, o arquivo `fabric.mod.json` contém um campo chamado `entrypoints` — esse campo é usado para especificar os pontos de entrada que seu mod oferece.

O gerador de mod de modelo cria os dois pontos de entrada `main` e `cliente` por padrão:

- O ponto de entrada `main` é usado para código comum, e está contido em uma classe que implementa `ModInitializer`
- O ponto de entrada `client` é usado para código específico do cliente, e sua classe implementa `ClientModInitializer`

Esses pontos de entrada são chamados respectivamente quando o jogo começa.

Aqui está um exemplo de um ponto de entrada `main` simples que registra uma mensagem no console quando o jogo começa:

@[code lang=java transcludeWith=#entrypoint](@/reference/latest/src/main/java/com/example/docs/ExampleMod.java)

## `src/main/resources` {#src-main-resources}

A pasta `src/main/resources` é usada para armazenar os recursos que seu mod usa, como texturas, modelos e sons.

É também o local do `fabric.mod.json` e de quaisquer arquivos de configuração de mixin que seu mod utiliza.

Assets are stored in a structure that mirrors the structure of resource packs - for example, a texture for a block would be stored in `assets/example-mod/textures/block/block.png`.

## `src/client/resources` {#src-client-resources}

A pasta `src/client/resources` é usada para armazenar recursos específicos do cliente, como texturas, modelos e sons usados apenas no lado do cliente.

## `src/main/java` {#src-main-java}

A pasta `src/main/java` é usada para armazenar o código-fonte Java do seu mod — ela existe nos ambientes cliente e servidor.

## `src/client/java` {#src-client-java}

A pasta `src/client/java` é usada para armazenar código-fonte Java específico do cliente, como código de renderização ou lógica do lado do cliente — como provedores de cores de bloco.
