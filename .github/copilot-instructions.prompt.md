# Regras para GitHub Copilot: Desenvolvimento Java/Kotlin & Paper/Bukkit

Este documento estabelece diretrizes para utilizar o GitHub Copilot, visando o desenvolvimento de projetos Java/Kotlin com Gradle para a plataforma Paper/Bukkit. O objetivo é promover boas práticas de desenvolvimento de software e de plugins, garantindo código limpo, eficiente e seguro.

## 1. Contexto e Boas Práticas Gerais de Java/Kotlin

O Copilot é excelente para completar código, mas precisa de contexto. Certifique-se de que seu código siga as convenções padrão para que o Copilot gere sugestões relevantes.

### 1.1. Nomenclatura e Convenções

* **Java:** Siga as [Code Conventions for the Java TM Programming Language](https://www.google.com/search?q=https://www.oracle.com/java/technologies/javase/codeconventions.html).
* **Kotlin:** Siga as [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html).
* **Nomenclatura:** Use nomes de variáveis, métodos e classes que sejam **descritivos e claros**. Isso ajuda o Copilot a entender a intenção do seu código.
  * **Ruim:** `int x = 10;`
  * **Bom:** `int playerCount = 10;`
* **Comentários:** Utilize comentários para explicar a *razão* por trás de decisões complexas ou trechos de código não óbvios. O Copilot pode usar esses comentários para gerar código ou sugestões mais alinhadas.

### 1.2. Documentação de Código (JavaDoc/KDoc)

Sempre documente classes, interfaces, métodos e campos públicos. O Copilot pode usar essa documentação para inferir o comportamento esperado e gerar código correspondente.

* **JavaDoc:** Utilize o padrão [JavaDoc](https://www.google.com/search?q=https://www.oracle.com/java/technologies/javase/javadoc-ug.html).
  * Exemplo:

    ```java
    /**
     * Representa um jogador no jogo.
     * Fornece métodos para interagir com o estado do jogador.
     */
    public class GamePlayer {
        /**
         * Retorna o nome do jogador.
         * @return O nome completo do jogador.
         */
        public String getName() { /* ... */ }
    }
    ```

* **KDoc:** Utilize o padrão [KDoc](https://www.google.com/search?q=https://kotlinlang.org/docs/reference/kdoc-syntax.html).
  * Exemplo:

    ```kotlin
    /**
     * Representa um jogador no jogo.
     * Fornece métodos para interagir com o estado do jogador.
     */
    class GamePlayer {
        /**
         * Retorna o nome do jogador.
         * @return O nome completo do jogador.
         */
        fun getName(): String { /* ... */ }
    }
    ```

### 1.3. Links para Documentações (Referência para o Copilot)

Ao programar, o Copilot "lê" o contexto. Ter essas documentações em mente (e, se possível, abertas em abas do navegador ou em comentários para o Copilot "digerir") pode ajudar a direcionar as sugestões.

* **Java SE API:** [Java 21 Platform SE API Specification](https://docs.oracle.com/en/java/javase/21/docs/api/index.html) (ou a versão do seu projeto).
* **Kotlin Standard Library:** [Kotlin Standard Library API](https://www.google.com/search?q=https://kotlinlang.org/api/latest/jvm/stdlib/index.html)

## 2. Boas Práticas para Desenvolvimento de Plugins Bukkit/Paper

### 2.1. Uso da API Bukkit/Paper

* **Contexto da API:** Quando estiver escrevendo código relacionado à API Bukkit/Paper, o Copilot se beneficiará de você iniciar o código com chamadas à API, eventos ou métodos comuns.
  * Exemplo: `Player p = event.getPlayer();` `Bukkit.getServer().getLogger().info("...");`
* **Eventos:** Ao implementar listeners de eventos, comece digitando o esqueleto do método `@EventHandler` ou o nome da classe do evento (`PlayerJoinEvent`, `BlockBreakEvent`).
* **Agendamento de Tarefas (Schedulers):** O Copilot pode sugerir `Bukkit.getScheduler().runTask(...)` ou `runTaskLaterAsync(...)`. Tenha em mente se a tarefa precisa ser síncrona ou assíncrona.
* **Manipulação de Configuração (YAML):** Ao trabalhar com `FileConfiguration` (ou `YamlConfiguration`), o Copilot pode ajudar a gerar `config.getString("path.to.value")`, etc.
  * **Referência:** [Bukkit API JavaDocs](https://hub.spigotmc.org/javadocs/bukkit/)
  * **Referência:** [Paper API JavaDocs](https://www.google.com/search?q=https://papermc.io/javadocs/paper/1.20.4/) (ou a versão do seu projeto).

### 2.2. Referências de APIs e Bibliotecas Adicionais

Para um contexto mais aprofundado, o Copilot pode se beneficiar de ter acesso ou de você estar ciente das seguintes documentações:

* **Nexo JD**: [JD-NexoMC](https://jd.nexomc.com/1.8/) - Para referências diretas da API Nexo.
* **Nexo Doc**: [NexoMC](https://docs.nexomc.com/) - Documentação geral e guias sobre a plataforma Nexo.
* **ACF Wiki Doc**: [Annotation Command Framework](https://github.com/aikar/commands/wiki/Using-ACF) - Essencial para o desenvolvimento de comandos com ACF.
* **TaskChain JD**: [TaskChain](https://taskchain.aikar.co/) - Para referências diretas da API TaskChain
* **TaskChain Wiki Doc**: [TaskChain](https://github.com/aikar/TaskChain/wiki/why-taskchain) - Essencial para uma abstração simplificada do *Bukkit Scheduler* para operações assíncronas.

### 2.3. Segurança e Performance

* **Operações Assíncronas:** Para operações longas (acesso a banco de dados, requisições HTTP, cálculos pesados), sempre considere usar threads assíncronas para não travar o *tick* principal do servidor. Comente suas intenções para o Copilot.
  * Exemplo Tradicional:

  ```java
  public class UglyExample {
    ExecutorService threadPool;
    public void someBigOperation(
      Object player,
      Object identifier
    ) {
      threadPool.submit(() -> {
        Object result = someLongBlockingRequest(
          player,
          identifier
        );
        if (result == null) {
            sendPlayerMessage(player, "Sorry, something failed!");
            return;
        }
        GameScheduler.runTaskOnMain(() -> {
          Object opResult = GameAPI.
            doSomethingToPlayerWithResult(
              player,
              result
            );
          if (opResult == null) {
              sendPlayerMessage(player, "Sorry, something failed!");
              return;
          }
          threadPool.submit(() -> {
              markOperationSuccessful(player, identifier, opResult);
          });
        });
      });
    }
  }
  ```

  * Exemplo com o TaskChain:

  ```java
  public class GoodExample {

    TaskChainFactory taskChain;

    public void someBigOperation(
      Object player, 
      Object identifier
    ) {
      taskChain.newChain()
        .asyncFirst(
          () -> someLongBlockingRequest(player, identifier)
        )
        .abortIfNull(
          ActionHandlers.MESSAGE, 
          player, "Sorry, something failed!"
        )
        .sync(
          (result) -> GameAPI.doSomethingToPlayerWithResult(player, result)
        )
        .abortIfNull(
          ActionHandlers.MESSAGE, 
          player, "Sorry, something failed!"
        )
        .async(
          (opResult) -> markOperationSuccessful(player, identifier, opResult)
        ).execute();
    }
  }
  ```

* **Evite Bloqueios:** Mantenha o *main thread* (tick do servidor) o mais livre possível. O Copilot pode sugerir bloqueios, mas você deve intervir se a operação for longa.

* **Tratamento de Exceções:** Utilize `try-catch` para lidar com exceções esperadas e logue as inesperadas. O Copilot pode sugerir blocos `try-catch` vazios; **evite-os** e sempre adicione um log ou tratamento adequado.

## 3. Integração com Gradle

O Copilot pode auxiliar na escrita de scripts Gradle, mas é crucial ter um `build.gradle(.kts)` bem estruturado.

### 3.1. Configuração do `build.gradle(.kts)`

* **Dependências:** Ao adicionar dependências, o Copilot pode sugerir versões. Sempre verifique a versão mais recente e compatível com seu projeto.
  * Exemplo para Paper:

    ```groovy
    repositories {
        mavenCentral()
        maven {
          url 'https://repo.papermc.io/repository/maven-public/'
        }
    }

    dependencies {
        compileOnly 'io.papermc.paper:paper-api:1.20.4-R0.1-SNAPSHOT' // ou a versão estável
    }
    ```

* **Plugins Gradle:** Ao adicionar plugins (`java`, `kotlin-jvm`, `shadowJar`), o Copilot pode ajudar com a sintaxe.

### 3.2. Referências Gradle

* [Gradle User Manual](https://docs.gradle.org/current/userguide/userguide.html)
* [Gradle Kotlin DSL Primer](https://docs.gradle.org/current/userguide/kotlin_dsl.html)

## 4. Geração de Mensagens de Commit com o Copilot

O Copilot pode ser um aliado poderoso para gerar mensagens de commit que sigam o padrão Conventional Commits.

### 4.1. Estrutura do Commit (Padrão Conventional Commits)

Utilize o padrão Conventional Commits para um histórico claro.

  ```txt
  <tipo>(<escopo opcional>): <descrição>

  [Corpo opcional]

  [Rodapé opcional]
  ```

### 4.2. Tipos Comuns

* **`feat`**: Nova funcionalidade. Ex: `feat(playerData): adicionar sistema de economia`
* **`fix`**: Correção de bug. Ex: `fix(commands): corrigir permissão do comando /home`
* **`docs`**: Apenas documentação. Ex: `docs: atualizar README com requisitos`
* **`refactor`**: Reestruturação de código. Ex: `refactor(events): otimizar tratamento de PlayerJoinEvent`
* **`chore`**: Tarefas de manutenção. Ex: `chore(deps): atualizar Paper API para 1.20.4`
* **`build`**: Alterações no build. Ex: `build: configurar ShadowJar para plugins`
* **`test`**: Adição/correção de testes. Ex: `test(economy): adicionar testes para transações`
* **`style`**: Formatação. Ex: `style: formatar arquivos Kotlin conforme convenções`

### 4.3. Dicas para Orientar o Copilot em Commits

* **Comentários Temporários:** Adicione comentários temporários nos seus arquivos ou no próprio campo de commit do Git no VS Code (se disponível) antes de pedir sugestões.

  ```txt
  feat: implementar novo sistema de lojas para o servidor
  Permite aos jogadores criar lojas para vender e comprar itens.
  ```

* **Granularidade:** Commits pequenos e focados facilitam a vida do Copilot. Se uma sugestão estiver muito genérica, seu commit provavelmente está fazendo muitas coisas.
* **Revisão Humana:** **Sempre revise e ajuste as sugestões do Copilot.** Ele pode entender o código, mas não o contexto completo da sua intenção ou das convenções específicas do seu projeto.
* **Imperativo:** A linha de assunto deve começar com um verbo no imperativo (ex: "Adicionar", "Corrigir", "Remover").
* **Corpo do Commit:** Use o corpo para explicar o *porquê* da mudança, os desafios encontrados, decisões e impactos.
* **Referências a Issues:** Se o commit está relacionado a um issue, inclua a referência no rodapé (ex: `Fixes #123`).
