# Clima App

Aplicativo Android de previsão do tempo, feito em **Kotlin** com **Jetpack Compose**.
Os dados vêm da API pública [Open-Meteo](https://open-meteo.com/en/docs), que não exige chave de acesso.

O projeto é didático: mostra, na prática, como organizar um app Android em camadas,
consumir uma API REST, guardar dados localmente, fazer sincronização em segundo plano
e enviar notificações.

## O que o app faz

- **Busca de cidades**: digite o nome de uma cidade e veja os resultados encontrados
  (com país e região, para diferenciar cidades de mesmo nome).
- **Detalhes do tempo**: ao abrir uma cidade, mostra a temperatura atual e a previsão
  de máxima e mínima diária.
- **Favoritos**: marque cidades como favoritas. Elas ficam salvas no banco local (Room),
  então a lista continua disponível mesmo sem internet.
- **Sincronização em segundo plano**: um worker (WorkManager) atualiza periodicamente
  o tempo das cidades favoritas, somente quando há conexão.
- **Notificações**: se a temperatura de uma cidade favorita atingir o limiar configurado,
  o app envia uma notificação local.
- **Configurações**: permite escolher o intervalo de sincronização e o limiar
  de temperatura para notificação.
- **Tema visual**: tema Material 3 com tipografia própria.

## Tecnologias

| Área                  | Tecnologia                                   |
|-----------------------|----------------------------------------------|
| Linguagem             | Kotlin                                       |
| Interface             | Jetpack Compose + Material 3                 |
| Navegação             | Navigation Compose                           |
| Injeção de dependência| Hilt (com KSP)                               |
| Rede                  | Retrofit + Gson                              |
| Banco local           | Room                                         |
| Preferências          | SharedPreferences (via `SettingsRepository`) |
| Assíncrono            | Coroutines + Flow                            |
| Background            | WorkManager                                  |

## Arquitetura

O código fica em `app/src/main/java/com/example/climaapp/` e é separado em camadas:

```
climaapp/
├── ui/              Telas (Compose) e ViewModels
│   ├── search/      Busca de cidades
│   ├── details/     Previsão de uma cidade
│   ├── favorites/   Lista de favoritos
│   ├── settings/    Configurações de sincronização e notificação
│   ├── navigation/  Rotas e NavHost
│   └── theme/       Cores, tipografia e tema
├── domain/          Regras do app: modelos (City, WeatherForecast)
│                    e a interface WeatherRepository
├── data/            Implementação dos dados
│   ├── remote/      Serviços Retrofit (geocodificação e previsão) e DTOs
│   ├── local/       Entidades, DAO e banco Room (AppDatabase)
│   ├── repository/  WeatherRepositoryImpl: junta API e banco
│   └── settings/    Preferências do usuário
├── work/            SyncWorker, agendamento e notificações
├── di/              Módulos do Hilt (rede, banco, repositório, preferências)
├── MainActivity.kt
└── ClimaApplication.kt
```

Fluxo de uma tela: a **Screen** (Compose) observa o estado de um **ViewModel**,
que chama o **WeatherRepository** (interface do domínio). A implementação
(**WeatherRepositoryImpl**) decide se busca na API (Retrofit) ou no banco local (Room).
As dependências entre as camadas são entregues pelo Hilt.

## Como rodar no Android Studio

### Requisitos

- Android Studio (versão recente, com suporte ao Gradle 9.x)
- Conexão com a internet (para o Gradle baixar dependências e para o app acessar a API)
- No **SDK Manager**, instale a plataforma **Android 17 (API 37)**, que é o `compileSdk` do projeto
- Um emulador ou celular com **Android 7.0 (API 24)** ou superior

### Passos

1. Descompacte o arquivo `.zip` e abra a pasta `clima-app-main` no Android Studio
   (**File > Open**). Não abra a pasta `app` isoladamente.
2. Aguarde o **Gradle Sync** terminar. Na primeira vez pode demorar alguns minutos.
3. Se o Gradle pedir um JDK, use o JDK embutido do Android Studio (JBR) ou deixe o
   Gradle baixar o JDK 25 automaticamente (configurado em
   `gradle/gradle-daemon-jvm.properties`).
4. Escolha um emulador ou celular conectado na barra superior e clique em **Run ▶**
   (ou pressione `Shift + F10`).

### Testes

- Testes unitários: `app/src/test/`
- Testes instrumentados: `app/src/androidTest/`

Para rodar pelo terminal, na raiz do projeto: `./gradlew assembleDebug`
(no Windows: `gradlew.bat assembleDebug`).

## Estrutura do projeto

- `app/`: código do aplicativo, recursos (`res/`) e manifesto
- `gradle/libs.versions.toml`: versões centralizadas de todas as bibliotecas
- `build.gradle.kts` e `settings.gradle.kts`: configuração do Gradle
