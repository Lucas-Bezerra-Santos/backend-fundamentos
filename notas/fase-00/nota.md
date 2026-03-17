# Fase 0 — Notas de Estudo

---

## 1. Diferenças de Mindset Front vs Back

**Fonte:** Fireship - Backend Roadmap

### Frontend pensa em:
- **UI/UX** — como o usuário vê e interage
- **Estado local** — React state, stores, cache do browser
- **Renderização** — DOM, virtual DOM, repaints
- **Responsividade** — layout, breakpoints, animações

### Backend pensa em:
- **Dados** — modelagem, persistência, integridade
- **Concorrência** — múltiplos usuários ao mesmo tempo, race conditions
- **Durabilidade** — dados não podem se perder (ACID, replicação)
- **Segurança** — autenticação, autorização, sanitização de inputs
- **Escala** — horizontal vs vertical, load balancing, caching

### Mudança de mentalidade chave:
- No front, você **reage a eventos do usuário**
- No back, você **processa requisições de N clientes simultaneamente**
- Front: "como mostrar isso?" → Back: "como garantir que isso funcione para 10.000 usuários ao mesmo tempo?"
- O backend é **stateless por requisição** mas precisa gerenciar **estado persistente** (banco de dados)
- Erros no front afetam 1 usuário; erros no back podem afetar **todos**

---

## 2. Como a Internet Funciona (DNS, TCP/IP, HTTP)

**Fonte:** CS50 - How the Internet Works

### Camadas da comunicação:

**IP (Internet Protocol)**
- Cada dispositivo tem um endereço IP (ex: `192.168.1.1`)
- Responsável por **endereçar e rotear** pacotes entre máquinas
- Dados são quebrados em **pacotes** pequenos que podem seguir caminhos diferentes

**TCP (Transmission Control Protocol)**
- Garante que os pacotes cheguem **completos e na ordem correta**
- Faz o **handshake de 3 vias**: SYN → SYN-ACK → ACK
- Retransmite pacotes perdidos — **confiabilidade**
- Usa **portas** para direcionar ao serviço certo (80 = HTTP, 443 = HTTPS)

**DNS (Domain Name System)**
- Traduz nomes legíveis (`google.com`) para IPs (`142.250.80.46`)
- Funciona como uma **agenda telefônica** da internet
- Hierarquia: Root → TLD (.com, .br) → Authoritative DNS
- Resultado é **cacheado** no browser e no SO para performance

**HTTP (HyperText Transfer Protocol)**
- Protocolo de **requisição-resposta** entre cliente e servidor
- Métodos: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`
- Status codes: `200` OK, `301` redirect, `404` not found, `500` server error
- **Stateless** — cada requisição é independente (cookies/tokens mantêm sessão)
- HTTPS = HTTP + TLS (criptografia)

---

## 3. Modelo Cliente-Servidor em Profundidade

**Fonte:** Hussein Nasser - Client Server Architecture

### O modelo básico:
- **Cliente** — quem faz o pedido (browser, app mobile, CLI, outro servidor)
- **Servidor** — quem recebe, processa e responde

### Anatomia de uma requisição:
```
Cliente → [Request] → Servidor → [Processa] → [Response] → Cliente
```

### Conceitos importantes:

**Request/Response cycle**
- Cliente envia: método + URL + headers + body (opcional)
- Servidor responde: status code + headers + body

**Stateless vs Stateful**
- HTTP é stateless — servidor não lembra do cliente entre requisições
- Estado é mantido via: cookies, tokens JWT, sessions server-side

**Tipos de comunicação:**
| Padrão | Descrição | Exemplo |
|---|---|---|
| **Request/Response** | Cliente pede, servidor responde | REST API |
| **Polling** | Cliente pergunta repetidamente | Chat simples |
| **Long Polling** | Servidor segura a conexão até ter dados | Notificações |
| **WebSockets** | Conexão bidirecional persistente | Chat real-time |
| **Server-Sent Events** | Servidor envia dados unidirecionalmente | Feed de updates |

**Escalando servidores:**
- **Vertical** — mais CPU/RAM na mesma máquina
- **Horizontal** — mais máquinas + load balancer
- O load balancer distribui requisições entre instâncias

---

## 4. O que Acontece Quando Você Digita uma URL

**Fonte:** Alex Xu - What happens when you type a URL

### Passo a passo completo:

1. **Digitou `https://example.com/page`**
   - Browser verifica cache local (já resolveu esse DNS antes?)

2. **Resolução DNS**
   - Cache do browser → cache do SO → resolver do ISP → root DNS → TLD → authoritative
   - Resultado: IP do servidor (ex: `93.184.216.34`)

3. **Conexão TCP**
   - Handshake de 3 vias (SYN → SYN-ACK → ACK)
   - Estabelece conexão confiável entre cliente e servidor

4. **Handshake TLS (se HTTPS)**
   - Negocia algoritmo de criptografia
   - Servidor envia certificado SSL
   - Chaves de sessão são trocadas
   - A partir daqui, tudo é criptografado

5. **Requisição HTTP**
   - Browser envia: `GET /page HTTP/1.1` + headers (Host, User-Agent, Accept, cookies...)

6. **Servidor processa**
   - Web server (Nginx/Apache) recebe a requisição
   - Roteia para a aplicação (Node, Python, Go...)
   - Aplicação pode consultar banco de dados, cache (Redis), outros serviços
   - Monta a resposta

7. **Resposta HTTP**
   - Servidor retorna: `200 OK` + headers + body (HTML, JSON...)
   - Headers incluem: Content-Type, Cache-Control, Set-Cookie...

8. **Browser renderiza**
   - Faz parsing do HTML → constrói DOM
   - Encontra CSS/JS → faz novas requisições
   - Renderiza a página na tela

### Diagrama mental:
```
URL → DNS → IP → TCP → TLS → HTTP Request → Server → DB/Cache → HTTP Response → Render
```

### O que muda na visão backend:
- Você agora se preocupa com os passos **6 e 7** — o que acontece dentro do servidor
- Como rotear, como processar, como responder rápido, como escalar
