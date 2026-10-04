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
As dependências do JUnit têm scope "test" e não entram no JAR final.