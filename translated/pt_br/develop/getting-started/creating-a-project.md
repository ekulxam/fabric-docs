---
title: Criando um Projeto
description: Um guia passo a passo sobre como criar um novo projeto de mod usando o Fabric Template Mod Generator.
authors:
  - Cactooz
  - IMB11
  - radstevee
  - Thomas1034
---

O Fabric fornece uma maneira fácil de criar um novo projeto de mod usando o Fabric Template Mod Generator - se quiser, você pode criar manualmente um novo projeto usando o repositório de mods de exemplo. Você deve consultar a seção [Criação manual de projetos](#manual-project-creation).

## Gerando um Projeto {#generating-a-project}

Você pode usar o [Fabric Template Mod Generator](https://fabricmc.net/develop/template/) para gerar um novo projeto para seu mod - você deve preencher os campos obrigatórios, como o nome do mod, o nome do pacote e a versão do Minecraft para qual você deseja desenvolver.

O nome do pacote deve ser em letras minúsculas, separado por pontos e único para evitar conflitos com pacotes de outros programadores. It is typically formatted as a reversed internet domain, such as `com.example.example-mod`.

:::warning IMPORTANT

Make sure you remember your mod's ID! Whenever you find `example-mod` in these docs, especially in file paths, you will have to replace it with your own.

For example, if your mod ID was **`my-cool-mod`**, instead of _`resources/assets/example-mod`_ use **`resources/assets/my-cool-mod`**.

:::

![Visualização do gerador](/assets/develop/getting-started/template-generator.png)

If you either want to use Kotlin, or Fabric's Yarn mappings instead of the default Mojang Mappings, or want to add data generators, you can select the appropriate options in the `Advanced Options` section.

::: info

Code examples given on this site use [Mojang's official names](../migrating-mappings/#mappings). If your mod is not using the same mappings that these docs are written in, you will need to convert the examples using sites like [mappings.dev](https://mappings.dev/) or [Linkie](https://linkie.shedaniel.dev/mappings?namespace=yarn&translateMode=ns&translateAs=mojang_raw&search=).

:::

![Seção de opções avançadas](/assets/develop/getting-started/template-generator-advanced.png)

Após preencher os campos obrigatórios, clique no botão `Gerar`, e o gerador vai criar um novo projeto para você usar na forma de um arquivo zip.

You should extract this zip file to a location of your choice, and then open the extracted folder in your IDE.

::: tip

You should follow these rules when choosing the path to your project:

- Avoid cloud storage directories (for example Microsoft OneDrive)
- Avoid non-ASCII characters (for example emoji, accented letters)
- Avoid spaces

An example of a "good" path may be: `C:\Projects\YourProjectName`

:::

## Criação Manual de Projeto {#manual-project-creation}

:::info PREREQUISITES

Você precisará do [Git](https://git-scm.com/) instalado para clonar o repositório de mod de exemplo.

:::

Se você não puder usar o Fabric Template Mod Generator, você pode criar um novo projeto manualmente seguindo estas etapas.

Primeiro, clone o repositório de mod de exemplo usando o Git:

```sh
git clone https://github.com/FabricMC/fabric-example-mod/ example-mod
```

This will clone the repository into a new folder called `example-mod`.

You should then delete the `.git` folder from the cloned repository, and then open the project. Se a pasta `.git` não aparecer, você deve habilitar a exibição de itens ocultos no seu gerenciador de arquivos.

Once you've opened the project in your IDE, it should automatically load the project's Gradle configuration and perform the necessary setup tasks.

Novamente, como mencionado anteriormente, se você receber uma notificação falando sobre um script de compilação do Gradle, você deve clicar no botão `Importar projeto do Gradle`.

### Modificando o Modelo {#modifying-the-template}

Depois que o projeto for importado, você deve modificar os detalhes do projeto para corresponder aos detalhes do seu mod:

- Modifique o arquivo `gradle.properties`do projeto para alterar as propriedades `maven_group`e `archive_base_name` para corresponder aos detalhes do seu mod.
- Modifique o arquivo `fabric.mod.json` para alterar as propriedades `id`, `name` e `description` para corresponder aos detalhes do seu mod.
- Certifique-se de atualizar as versões do Minecraft, os mapeamentos, o Loader e o Loom - todos os quais podem ser consultados através de <https://fabricmc.net/develop/> - para que correspondam às versões que você deseja visar.

Obviamente, você pode alterar o nome do pacote e a classe principal do mod para corresponder aos detalhes do seu mod.
