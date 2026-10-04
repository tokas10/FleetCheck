# FleetCheck – Build Systems Lab

This project is intentionally incomplete. Follow the worksheet in the order given.

Expected final application output:

```
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 2
Average mileage: 37000 km
```

Do not copy the solution POM. The objective is to observe how each build change alters the result.


### Evidência 1
[ERROR] /C:/Users/silva/Desktop/FleetCheck_Starter/src/main/java/pt/upt/fleetcheck/App.java:[4,38] package com.fasterxml.jackson.databind does not exist

Causado pelo import da linha 4 do App.java: `import com.fasterxml.jackson.databind.ObjectMapper;`
(a depenência jackson-databind não estava declarada no pom.xml)


### Passo 2
[ERROR] FleetServiceTest.vehicleExactlyAtIntervalNeedsService:12 expected: <true> but was: <false>

Porque é uma falha melhor que a do passo 1?
No passo 1 o projeto nem compilava por causa de um problema de configuração da build (dependência em falta)
Agora a build já resolve as dependencias, compila e chega ate a fase de testes, que deteta um defeito
de comportamento real: um veiculo com exatamente 10 000 km desde a revisão não era considerado a precisar
de revisão -> A build passou a funcionar como "quality gate" e impede que código com bugs seja empacotado

Depois de corrigir > para >= em FleetService.needsService, a build passa (Tests run: 2, Failures: 0, BUILD SUCCESS)


### Passo 3 – mvn dependency:tree
pt.upt.softwarequality:fleetcheck:jar:1.0.0
+- com.fasterxml.jackson.core:jackson-databind:jar:2.22.2:compile
|  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.22:compile
|  \- com.fasterxml.jackson.core:jackson-core:jar:2.22.2:compile
\- org.junit.jupiter:junit-jupiter:jar:5.14.4:test
   (+ dependências transitivas do JUnit, todas com scope test)

jackson-databind é dependência direta (declarada no pom.xml).
jackson-core e jackson-annotations são transitivas: o Maven resolveu-as
automaticamente porque o jackson-databind precisa delas.
As dependências do JUnit têm scope "test" e não entram no JAR final

### Passo 4 – JAR executável
JAR normal: `java -jar target/fleetcheck-1.0.0.jar` → "no main manifest attribute, in target/fleetcheck-1.0.0.jar"

Com o Shade plugin: `java -jar target/fleetcheck-1.0.0-all.jar`
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 2
Average mileage: 37000 km

O que mudou com o Shade plugin?
O JAR por omissao so tem as classes do projeto e não tem Main-Class no manifest, por isso não é
executável nem inclui o Jackson. O Shade cria um "fat/uber JAR" (fleetcheck-1.0.0-all.jar) que junta
as classes do projeo com as dependencias de runtime (jackson-databind, jackson-core,
jackson-annotations) e escreve Main-Class: pt.upt.fleetcheck.App no MANIFEST.MF.
Os avisos de "overlapping resources" (META-INF/LICENSE, NOTICE) são normais 

### Passo 5 – Maven Wrapper
Gerado com `mvn wrapper:wrapper` (mvnw, mvnw.cmd, .mvn/wrapper/), fixado no Maven 3.9.16.
`.\mvnw.cmd clean verify` -> BUILD SUCCESS

Que suposição escondida sobre o ambiente o wrapper removeu?
Que cada máquina (de cada developer ou do servidor de CI) tem o Maven instalado, no PATH e numa versão
compatível. Com o wrapper, a versão do Maven fica fixada no projeto e é descarregada automaticamente,
por isso todos usam exatamente a mesma versão.
A propriedade project.build.outputTimestamp fixa a data gravada no JAR, para builds reprodutíveis.

### Passo 6 – GitHub Actions
Workflow "Build and Verify" com sucesso (Status: Success, artefacto fleetcheck-build):
https://github.com/tokas10/FleetCheck/actions/runs/37222989604

### Passo 7 – SBOM (CycloneDX)
`.\mvnw.cmd clean verify` → "CycloneDX: Creating BOM version 1.6 with 3 component(s)" → target/bom.json
O bom.json contém jackson-databind, jackson-core e jackson-annotations

Porque é que o SBOM contem componentes que nao escrevi no pom.xml?
Porque o SBOM descreve o grafo de dependências efetivamente resolvido pelo Maven e não so as dependências
declaradas. Só declarei jackson-databind, mas este depende de jackson-core e jackson-annotations,
que o Maven resolve como dependências transitivas e que acabam dentro do JAR entregue
O SBOM lista tudo o que faz parte do software (util para detetar vulnerabilidades e verificar licenças)
O JUnit não aparece porque tem scope test e não faz parte do artefacto final
### Nota sobre o teste
A pasta src/test do projeto inicial estava vazia, por isso o passo 2 não podia falhar na fase de testes.
Criei src/test/java/pt/upt/fleetcheck/FleetServiceTest.java com um teste de fronteira (veículo com exatamente
10 000 km desde a revisão), que expôs o defeito em FleetService.needsService (> em vez de >=).

### Versão Gradle
Repositório: https://github.com/tokas10/FleetCheck-Gradle (evidências da parte 8 no README desse projeto)
