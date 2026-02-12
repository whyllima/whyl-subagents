# Business & Product Agents (`whyll-agents-biz`)

Agentes especializados em produto, UX, negocios, conteudo, gestao de projetos e mais. Inclui o **Discovery Pipeline** para validacao de ideias.

**Prefixo:** `@whyll-agents-biz:<agente>`

---

## Discovery Pipeline

Cadeia de validacao de ideias de produto:

```
product-manager -> ux-researcher -> business-analyst -> content-marketer
   (ideia)       (problema real?)    (requisitos/viab.)   (posicionamento)
```

---

## Agentes

### `product-manager`

Especialista em estrategia de produto, priorizacao de features e decisoes data-driven. Gera e valida ideias de produto, define problem statements e value hypotheses. Usa frameworks como RICE, Jobs to be Done, Kano Model e OKRs. E o primeiro agente no Discovery Pipeline.

**Exemplos:**

```text
@whyll-agents-biz:product-manager Define product idea and value hypothesis for a "habit tracker" app

@whyll-agents-biz:product-manager Create a PRD for a new notifications system with RICE prioritization

@whyll-agents-biz:product-manager Analyze competitor landscape and define positioning for our SaaS dashboard
```

---

### `ux-researcher`

Especialista em pesquisa de UX, testes de usabilidade e insights baseados em dados. Valida se a dor do usuario e real atraves de entrevistas, surveys, analytics e testes A/B. Segundo agente no Discovery Pipeline, consome o problem statement do product-manager e produz relatorio de validacao.

**Exemplos:**

```text
@whyll-agents-biz:ux-researcher Validate if users really struggle with our onboarding flow

@whyll-agents-biz:ux-researcher Design a usability test plan for the new checkout experience

@whyll-agents-biz:ux-researcher Create user personas and journey map for the e-commerce platform
```

---

### `business-analyst`

Especialista em levantamento de requisitos, melhoria de processos e analise de viabilidade. Traduz ideias validadas em requisitos, especificacoes e avaliacao de viabilidade tecnica/comercial. Terceiro agente no Discovery Pipeline.

**Exemplos:**

```text
@whyll-agents-biz:business-analyst Translate the habit tracker idea into requirements and viability assessment

@whyll-agents-biz:business-analyst Map the current order fulfillment process and identify bottlenecks

@whyll-agents-biz:business-analyst Create a cost-benefit analysis for migrating from monolith to microservices
```

---

### `content-marketer`

Especialista em estrategia de conteudo, SEO, marketing multicanal e otimizacao de conversao. Define posicionamento, narrativa e messaging hierarchy a partir da value proposition validada. Quarto agente no Discovery Pipeline.

**Exemplos:**

```text
@whyll-agents-biz:content-marketer Define positioning statement and messaging for the new SaaS product launch

@whyll-agents-biz:content-marketer Create a content strategy with SEO pillars for our developer tools blog

@whyll-agents-biz:content-marketer Plan an email nurture sequence for trial users with 5 touchpoints
```

---

### `project-manager`

Especialista em planejamento de projetos, gestao de recursos, mitigacao de riscos e comunicacao com stakeholders. Domina Waterfall, Agile, PRINCE2 e abordagens hibridas. Foca em entregar projetos no prazo, dentro do orcamento e com qualidade.

**Exemplos:**

```text
@whyll-agents-biz:project-manager Create a project plan with WBS, timeline and risk register for the platform migration

@whyll-agents-biz:project-manager Plan resource allocation for 3 parallel development streams over Q2

@whyll-agents-biz:project-manager Build a stakeholder communication matrix and status reporting cadence
```

---

### `scrum-master`

Especialista em facilitacao agil, transformacao Scrum e melhoria continua. Facilita cerimonias (planning, standup, review, retro), remove impedimentos, monitora velocity e promove equipes auto-organizadas. Conhece SAFe, LeSS e Spotify Model para scaling.

**Exemplos:**

```text
@whyll-agents-biz:scrum-master Facilitate sprint planning for the next 2-week sprint with capacity analysis

@whyll-agents-biz:scrum-master Design a retrospective format to address team communication issues

@whyll-agents-biz:scrum-master Assess agile maturity of the team and propose improvement plan
```

---

### `customer-success-manager`

Especialista em retencao de clientes, onboarding, adocao de produto e crescimento de receita. Monitora health scores, previne churn, identifica oportunidades de upsell e constroi programas de advocacy. Foca em maximizar lifetime value e satisfacao do cliente.

**Exemplos:**

```text
@whyll-agents-biz:customer-success-manager Create an onboarding playbook for enterprise customers with 90-day milestones

@whyll-agents-biz:customer-success-manager Build a churn prevention strategy with early warning indicators and intervention playbooks

@whyll-agents-biz:customer-success-manager Design a quarterly business review template with ROI metrics and expansion opportunities
```

---

### `sales-engineer`

Especialista em pre-vendas tecnicas, POCs e arquitetura de solucoes. Traduz tecnologia complexa em valor de negocio para prospects. Conduz demos, responde RFPs, resolve objecoes tecnicas e acelera o ciclo de vendas.

**Exemplos:**

```text
@whyll-agents-biz:sales-engineer Prepare a technical demo script for our API platform targeting enterprise prospects

@whyll-agents-biz:sales-engineer Create a POC plan with success criteria for a Fortune 500 integration

@whyll-agents-biz:sales-engineer Build a competitive comparison matrix against top 3 competitors with technical differentiators
```

---

### `technical-writer`

Especialista em documentacao tecnica clara e acessivel. Cria API references, guias de usuario, tutoriais e troubleshooting guides. Foca em information architecture, progressive disclosure e exemplos praticos que reduzem tickets de suporte.

**Exemplos:**

```text
@whyll-agents-biz:technical-writer Create API documentation for the authentication endpoints with request/response examples

@whyll-agents-biz:technical-writer Write a getting started guide for new developers integrating our SDK

@whyll-agents-biz:technical-writer Create a troubleshooting guide for common deployment issues with step-by-step solutions
```

---

### `legal-advisor`

Especialista em direito tecnologico, compliance e mitigacao de riscos. Cobre contratos, propriedade intelectual, privacidade de dados (GDPR/CCPA), termos de servico e conformidade regulatoria. Fornece orientacao juridica pratica que viabiliza inovacao dentro de limites legais seguros.

**Exemplos:**

```text
@whyll-agents-biz:legal-advisor Draft a privacy policy compliant with GDPR and CCPA for our SaaS platform

@whyll-agents-biz:legal-advisor Review the SLA contract template and identify liability risks

@whyll-agents-biz:legal-advisor Create a data processing agreement (DPA) for our European customers
```

---

### `wordpress-master`

Arquiteto WordPress de elite para desenvolvimento full-stack, otimizacao de performance e solucoes enterprise. Domina temas/plugins customizados, Gutenberg/FSE, multisite, WooCommerce, headless WordPress com REST/GraphQL, seguranca e scaling para milhoes de visitantes.

**Exemplos:**

```text
@whyll-agents-biz:wordpress-master Optimize site performance to achieve < 1.5s load time and pass Core Web Vitals

@whyll-agents-biz:wordpress-master Create a custom Gutenberg block for a pricing table with dynamic data from ACF

@whyll-agents-biz:wordpress-master Setup headless WordPress with REST API and Next.js frontend including JWT authentication
```
