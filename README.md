# 🛡️ API Gateway Reativo com Rate Limiting (Spring Cloud + Redis)

Neste projeto, construí um **API Gateway Reativo** para atuar como porta de entrada e proteger microsserviços contra acessos abusivos e picos de tráfego. 

O foco principal foi a implementação de um **Rate Limiter**. Caso um cliente tente enviar requisições acima do limite configurado, o Gateway intercepta a chamada, protege o servidor de destino e devolve o status HTTP `429 Too Many Requests`.

Este é um projeto simples que desenvolvi focando nos meus estudos de arquitetura de software. O objetivo principal foi colocar a mão na massa e **agregar conhecimento** sobre como proteger microsserviços contra acessos abusivos e picos de tráfego usando um **API Gateway Reativo**. 

 o Gateway atua como porta de entrada, protege o servidor de destino e devolve o status HTTP `429 Too Many Requests` se o limite de tráfego for estourado.

## 🛠️ Tecnologias Utilizadas
* **Java 21**
* **Spring Cloud Gateway (WebFlux):** Utilizei a arquitetura não-bloqueante (reativa) baseada no Netty para suportar alta concorrência sem travar threads.
* **Redis:** Atua como o cérebro do algoritmo, armazenando as permissões de acesso por IP em milissegundos.
* **Docker Compose:** Para provisionar a infraestrutura do Redis de forma isolada.
* **JSONPlaceholder:** API pública (Fake REST API) que utilizei como "microsserviço alvo" para simular o roteamento real das requisições.

## ⚙️ Arquitetura e Configurações

A configuração do roteamento foi feita utilizando a sintaxe moderna do ecossistema WebFlux (`server.webflux.routes`). Toda requisição feita para o caminho `/users/**` no meu Gateway é automaticamente roteada para a API externa do **JSONPlaceholder**.

Para proteger essa rota, utilizei uma combinação de recursos nativos e código customizado:

1. **O Filtro Nativo (`RequestRateLimiter`):** Utilizei o filtro padrão do próprio Spring Cloud Gateway. Ele embute a lógica do algoritmo "Token Bucket" e se comunica com o Redis para fazer a contagem e o bloqueio automático.
2. **O Código Customizado (`ipKeyResolver`):** Como o filtro nativo não sabe diferenciar os usuários, criei uma classe de configuração implementando a interface `KeyResolver`. Utilizei programação reativa (`Mono.just(...)`) para instruir o filtro a extrair o Endereço de IP (`RemoteAddress`) de cada requisição. Assim, o bloqueio atua individualmente por IP.

## 🚀 Passo a Passo: Como rodar e testar

Para ver o Rate Limiter em ação na sua máquina, siga estes passos:

**Passo 1: Subir a infraestrutura (Redis)**
Na raiz do projeto, inicie o container do Redis em segundo plano:
```bash
docker compose up -d
```
**Passo 2: Iniciar o Gateway**
Rode a aplicação Spring Boot pela sua IDE ou terminal. O Gateway iniciará na porta `8080`.

**Passo 3: Validar a Rota (Sucesso)**
Faça uma requisição simples (via navegador ou Postman) para:
`http://localhost:8080/users`
O Gateway vai rotear a requisição e devolver a lista de usuários lá do JSONPlaceholder com o status `200 OK`.

**Passo 4: Teste de Stress (Simulando o Ataque)**
Para testar o bloqueio, você precisa fazer requisições mais rápido do que o limite permitido.
1. Abra o Postman e salve a requisição `GET http://localhost:8080/users` em uma Collection.
2. Use a ferramenta **Collection Runner** do Postman.
3. Configure para **20 Iterations** (iterações) com **0ms de Delay** (atraso).
4. Rode o teste. Você verá os primeiros acessos retornando `200 OK` e, logo em seguida, o Redis cortará o acesso, retornando `429 Too Many Requests` em todas as requisições subsequentes.
