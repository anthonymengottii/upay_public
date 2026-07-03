# 📐 Decisões Técnicas de Infraestrutura

Registro estilo ADR (Architecture Decision Record) das principais decisões de infra do Upay: contexto, alternativas consideradas e consequências aceitas.

---

## 1. Multi-PSP com failover automático e circuit breaker por adquirente

**Contexto**: um gateway de pagamentos que depende de um único adquirente herda o risco de negócio desse adquirente. Suspensão de conta, instabilidade de API ou mudança de política derruba 100% do processamento.

**Decisão**: arquitetura de adaptadores (`PaymentProcessor`) com múltiplos PSPs registrados simultaneamente, roteamento por prioridade configurável e circuit breaker independente por adquirente. Quando o PSP de maior prioridade abre o circuito (falhas consecutivas), o sistema faz failover automático para o próximo ativo.

**Exemplo do tipo de risco que essa arquitetura mitiga**: se um adquirente perde acesso à sua conta (por exemplo, suspensão administrativa, mudança de política ou instabilidade prolongada), um sistema single-PSP fica sem capacidade de processar pagamentos até resolver o problema com aquele fornecedor específico. Com múltiplos PSPs já registrados e priorizados, outro adquirente assume automaticamente, sem downtime de processamento.

**Alternativas consideradas**:
- Integração direta com um único PSP: descartada pelo risco de ponto único de falha ilustrado acima.
- Camada de abstração só para PIX, cartão direto no PSP principal: descartada porque o mesmo risco se aplicaria ao método de maior volume.

**Consequências**:
- Cada novo PSP exige um adaptador dedicado (mapeamento de status, webhook, formato de request), custo de manutenção maior que integração única.
- Features acopladas a um PSP específico (por exemplo, conciliação de liquidação via endpoint proprietário de um adquirente) ficam inertes quando aquele PSP sai de operação. Precisam ser generalizadas por adquirente, não assumidas como fixas.
- Circuit breaker por PSP exige alerta observável (Sentry) para saber quando "nenhuma rota ativa" acontece. Sem isso, a resiliência vira um problema silencioso.

---

## 2. Redis para cache, rate limiting atômico e locks distribuídos

**Contexto**: contadores de rate limit e locks de job em memória de processo funcionam em single-instance, mas quebram sob concorrência real (múltiplos requests simultâneos) e não escalam para múltiplas instâncias/pods.

**Decisão**: Redis como camada compartilhada para cache, contadores de rate limit e locks de execução de jobs. Contadores usam `INCR` atômico (não read-then-write) para eliminar race conditions sob concorrência. Isso corrigiu bugs reais de rate limit de saque e de tentativas de OTP que podiam ser contornados por requests paralelos.

**Alternativas consideradas**:
- Rate limit em memória (Map/objeto no processo): descartado, não sobrevive a restart, não funciona com múltiplas instâncias, e read-then-write não é atômico sob concorrência.
- Lock só em memória para jobs (`isRunning` boolean): funciona em single-pod, mas quebra em deploy multi-pod (dois pods rodam o mesmo job simultaneamente). Ainda presente como debt conhecido em um job específico, não generalizado para todos.

**Consequências**:
- Redis vira dependência crítica: precisa estar disponível para rate limit e OTP funcionarem; falha do Redis deve ter fallback seguro (fail-closed em vez de deixar passar sem limite).
- TTL em todas as chaves de job/cache é obrigatório. Sem isso, chaves órfãs acumulam indefinidamente.

---

## 3. Roteamento por subdomínio (`app.*` vs `checkout.*`)

**Contexto**: o dashboard autenticado (merchant/admin) e o checkout público (cliente final, sem login) têm requisitos de segurança e CORS completamente diferentes. O checkout precisa ser embutível e acessível sem sessão, o dashboard não.

**Decisão**: separar por subdomínio dedicado (`app.[dominio].com` para a aplicação autenticada, `checkout.[dominio].com` para o checkout público) em vez de path-based routing (`/app` vs `/checkout`) no mesmo domínio.

**Alternativas consideradas**:
- Path-based routing no mesmo domínio: descartado, dificulta CORS granular (a mesma origin serviria tanto rotas autenticadas quanto públicas) e complica cache/CDN diferenciado por tipo de tráfego.

**Consequências**:
- CORS pode ter lista explícita de origens permitidas por finalidade, reduzindo superfície de ataque.
- Branding white-label (domínio customizado por instância) fica mais natural de configurar por subdomínio do que por path.
- Exige configuração de DNS/certificado para dois subdomínios em vez de um domínio único, custo operacional a mais por instância white-label.

---

## 4. Docker Compose para deploy self-hosted (além de Vercel/Render)

**Contexto**: como solução white-label, alguns parceiros preferem (ou exigem, por compliance) rodar a infraestrutura em ambiente próprio em vez de depender de PaaS de terceiros (Vercel/Render).

**Decisão**: manter Dockerfiles multi-stage (build enxuto de produção) para API e frontend, mais um `docker-compose.yml` que orquestra Postgres, Redis, API e Nginx, como caminho alternativo de deploy, sem abandonar o fluxo padrão em PaaS gerenciado para quem não precisa de self-hosted.

**Alternativas consideradas**:
- Só PaaS gerenciado (Vercel + Render): descartado como única opção por limitar parceiros com exigência de infraestrutura própria (dados sensíveis, compliance setorial).

**Consequências**:
- Dois caminhos de deploy para manter em paralelo (imagem Docker e configuração de PaaS), mais superfície de configuração para testar.
- Containers rodam non-root com `HEALTHCHECK` definido: decisão de hardening para não repetir o padrão comum de container rodando como root em produção.

---

## 5. Prisma + PostgreSQL com migrações versionadas

**Contexto**: um gateway de pagamentos tem integridade referencial crítica (transação, comissão, saldo, auditoria conectados entre si) e precisa de histórico de schema auditável e reversível.

**Decisão**: PostgreSQL como banco relacional único de sistema (sem poliglota de persistência), Prisma como ORM com migrações versionadas em código. Nenhuma alteração de schema é feita manualmente em produção sem migration correspondente no repositório.

**Alternativas consideradas**:
- NoSQL para dados de alto volume (transações, logs): descartado, o domínio exige transações ACID reais entre saldo, comissão e status de pagamento (`prisma.$transaction` atômico), o que um documento NoSQL não garante nativamente.

**Consequências**:
- Drift de schema entre ambientes (migration aplicada manualmente via SQL direto em dev, sem migration formal) já aconteceu e precisou de uma migration de consolidação retroativa. Reforça a disciplina de nunca alterar schema fora do fluxo de migration, mesmo sob pressão de prazo.
- Toda nova feature com necessidade de coluna nova (por exemplo, limite de transação por empresa) segue o mesmo padrão: migration versionada + `migrate resolve` quando aplicada fora de banda.

---

## 6. AuditLog imutável e LGPD: decisão dirigida por compliance, não por arquitetura

**Contexto**: como gateway de pagamentos operando no Brasil, a plataforma está sujeita à Resolução BACEN 4.658/2018 (retenção de logs de segurança) e à LGPD (direito ao esquecimento). Essas não são preferências técnicas, são obrigações regulatórias com prazo e formato definidos por lei.

**Decisão**: `AuditLog` sem operações de UPDATE/DELETE a nível de aplicação (somente INSERT), com retenção mínima de 1825 dias (5 anos, conforme BACEN 4.658/2018). Para atender à LGPD sem violar a imutabilidade do audit log, o fluxo `deleteMyData` **anonimiza** (não deleta) os registros de `AuditLog`, `Session` e `Transaction`. Preserva o histórico para compliance bancária enquanto remove os dados pessoais identificáveis.

**Alternativas consideradas**:
- Hard delete completo dos dados do usuário a pedido de erasure: descartado, violaria a obrigação de retenção do BACEN 4.658/2018, que tem precedência sobre o direito ao esquecimento da LGPD quando há obrigação legal concorrente.

**Consequências**:
- Toda nova tabela que armazena dado pessoal precisa decidir explicitamente sua estratégia de erasure (anonimização vs exclusão) no momento em que é criada. Não é algo que se resolve depois sem risco de inconsistência.
- Auditores/parceiros que pedem "prova de exclusão de dados" precisam ser informados de que a anonimização, não a exclusão física, é o mecanismo usado, e por quê.

---

> _Documento vivo, atualizado conforme novas decisões de infraestrutura são tomadas no projeto._
