# ACAP-Nodus: Open Implementation for SMEs

> **Nodus Protocol™ — an open, decentralized implementation of the WEF Agent Capability and Authorization Profile (ACAP)**
>
> Version: 1.0 • June 2026
> Author: Catafau (DIRCOM, Nodus Protocol)
> License: CC BY 4.0

---

## Taula de Continguts

1. [Resum Executiu](#1-resum-executiu)
2. [Context Polític: Dos Models de Governança](#2-context-polític-dos-models-de-governança)
3. [L'ACAP del WEF: Què és i Què Proposa](#3-lacap-del-wef-què-és-i-què-proposa)
4. [Mapatge ACAP ↔ Nodus Protocol](#4-mapatge-acap--nodus-protocol)
5. [Sistema d'Enforcement en Temps Real](#5-sistema-denforcement-en-temps-real)
6. [Política d'Enforcement al Relay](#6-política-denforcement-al-relay)
7. [Cicles de Vida i Phase Gates sobre Nostr](#7-cicles-de-vida-i-phase-gates-sobre-nostr)
8. [Complementarietat Estratègica](#8-complementarietat-estratègica)
9. [Riscos i Oportunitats](#9-riscos-i-oportunitats)
10. [Full de Ruta Recomanat](#10-full-de-ruta-recomanat)

---

## 1. Resum Executiu

El **World Economic Forum** ha publicat, amb **Capgemini**, el playbook *AI Agents in Action: A Playbook for Trusted Adoption, Authorization and Scaling* (maig 2026). La peça central és l'**ACAP (Agent Capability and Authorization Profile)**: un marc de 7 seccions (A-G) que tradueix política empresarial en autorització desplegable per a un agent d'IA.

Aquest document demostra que **el Nodus Protocol pot implementar l'ACAP del WEF sobre infraestructura oberta, descentralitzada i sense dependència de Big Tech** — fent-lo accessible per a pimes i empreses que no volen lligar-se a plataformes propietàries:

1. **Enforcement en temps real** via el relay Nostr (l'ACAP del WEF és un document per a auditoria *a posteriori*)
2. **Identitat criptogràfica descentralitzada** (claus BIP-340 Schnorr) versus "emerging standards" del WEF
3. **Machine-readable des del dia 1** (events JSON signats) versus "should evolve to policy-as-code" del WEF
4. **Un model de governança radicalment diferent**: open source, descentralitzat, no-capturable per Big Tech / Big Four

---

## 2. Context Polític: Dos Models de Governança

### 2.1 L'ACAP del WEF: Un Estàndard de les Big Four i Big Tech

L'ACAP ha estat desenvolupat pel **World Economic Forum** en col·laboració amb **Capgemini**, amb contribucions de **Microsoft, Google, Meta, Salesforce, Amazon (AWS), OpenAI, IBM, PwC, EY, BCG, ByteDance, SAP, ServiceNow, Snowflake, Hitachi, NEC**, i altres.

Això no és neutre. L'ACAP reflecteix una visió on:

- La governança d'agents es fa **dins de plataformes corporatives**
- L'autorització és un **document** (eventually policy-as-code, però dins del seu ecosistema)
- La identitat dels agents depèn de **registres centrals o platform-mediated**
- L'adopció passa per **consultories, auditories i eines de les mateixes empreses que defineixen l'estàndard**

En definitiva: **l'ACAP del WEF és un estàndard per a un món d'agents que viuen dins de plataformes propietàries.**

### 2.2 El Nodus Protocol: Un Model Des de la Base

El Nodus Protocol proposa un model **radicalment diferent**:

| Dimensió | WEF ACAP (Big Tech / Big Four) | Nodus Protocol |
|---|---|---|
| **Governança** | Top-down, platform-mediated | Peer-to-peer, relay-based |
| **Identitat** | "Emerging standards" (ANS, MCP-I) | Nostr keypairs (BIP-340, auto-sobirans) |
| **Codi** | Propietari (cada empresa implementa) | **Open Source** (CC BY 4.0) |
| **Enforcement** | Document per auditoria | Relay en temps real |
| **Propietat del protocol** | Organismes com WEF + corporacions | **Comunitat + estàndard obert** |
| **Captura regulatòria** | Alt risc (un estàndard que els mateixos Big Tech escriuen) | Baix risc (descentralitzat, sense gatekeeper) |
| **Cost d'entrada** | Alt (consultoria, llicències, plataformes) | Baix (qualsevol pot implementar-lo) |

### 2.3 La paradoxa política

L'ACAP del WEF és **alhora**:

- **Una amenaça**: si l'estàndard es consolida, les empreses miraran cap a solucions dins de l'ecosistema Big Tech/Big Four, ignorant protocols descentralitzats com Nodus.
- **Una oportunitat**: l'ACAP valida el *problema* (la governança d'agents necessita un marc estructurat) i el *què* (autorització progressiva, audit trail, human-in-the-loop). El Nodus pot ser **la implementació de referència** d'ACAP sobre infraestructura descentralitzada.

**L'estratègia correcta no és competir amb l'ACAP, sinó absorbir-lo:** demostrar que el Nodus Protocol és **la capa tècnica que l'ACAP necessita** per ser enforceable, auditable i verdaderament interoperable. I fer-ho des d'un model open source que cap Big Tech pot capturar.

---

## 3. L'ACAP del WEF: Què és i Què Proposa

### 3.1 Estructura de l'ACAP

L'ACAP és un document viu de **7 seccions** que es construeix en 3 fases:

| Secció | Propòsit |
|---|---|
| **A** — Identity & Scope | Identitat de l'agent, missió, límits explícits |
| **B** — Operating Context | Workflow, usuaris, sistemes, dades, jurisdiccions |
| **C** — Authority & Consequential Events | Permisos (permitted/conditional/prohibited), checkpoints |
| **D** — Controls & Enforcement | Com s'imposen els límits tècnicament |
| **E** — Evaluation Evidence & Promotion Gates | Evidència per desplegament i expansió d'autoritat |
| **F** — Monitoring, Incidents & Change Log | Telemetria, incidents, canvis versionats |
| **G** — Sign-offs & Re-authorization Cadence | Aprovacions, cadència de revisió |

### 3.2 Cicle de Vida (3 fases)

1. **System Design & Assessment** → ACAP A-C complet, D-E dissenyat, F no iniciat, G preparat
2. **Prepare & Deploy** → Controls implementats, evidència validada, monitoring activat
3. **Monitor & Improve** → Operació contínua, re-autorització, canvis controlats

Cada fase acaba amb un **phase gate** que requereix compliment i aprovació abans de progressar.

### 3.3 Deployment Context Tiers

| Nivell | Context |
|---|---|
| **L1** | Single-organization (trust boundary coneguda) |
| **L2** | Multi-organization, single-platform (trust transitive) |
| **L3** | Multi-platform, cross-boundary (sense autoritat central) |

---

## 4. Mapatge ACAP ↔ Nodus Protocol

### 4.1 Correspondència Secció a Kind

| ACAP Section | Nodus Kind | Descripció |
|---|---|---|
| **A** — Identity & Scope | `kind:34000` — `nodus:dw-profile` | DW profile: name, owner, entity_type, capabilities, limits, transports. El `d` tag conté el pubkey hex. |
| **A** (owner) | `kind:34001` — `nodus:org-relation` | Relació owner↔DW. `d` tag = `<owner_hex>-<dw_hex>`. |
| **B** — Operating Context | `kind:34000` (extensió) | Els camps `capabilities`, `limits` i l'estructura `transports` defineixen el context operacional. |
| **B** (jurisdictions) | `kind:34010` — `nodus:kyc-corp-claim` | KYC claim vinculant una entitat legal a la seva identitat criptogràfica. |
| **C** — Authority & Consequential Events | `kind:34002` — `nodus:policy` | **El Mandate.** Defineix capabilities, limits, hitl_required, auto_approve. IMMUTABLE. |
| **C** (HITL checkpoints) | `kind:10003` / `kind:10004` | HITL_REQUEST i HITL_RESPONSE. El checkpoint queda signat per l'humà. |
| **C** (consequential events register) | `kind:34002` (content) | El mateix mandate conté `hitl_required` i `limits` que actuen com a consequential events register. |
| **D** — Controls & Enforcement | `kind:34004` — `nodus:mcp-server-profile` | Perfils d'MCP Servers. Autorització de tools per DW. |
| **D** (orchestration) | `kind:10010`–`10013` | A2A Nostr-native. Orchestration dw↔dw amb verificació de mandate. |
| **E** — Evaluation Evidence | `kind:34003` — `nodus:audit-event` | Cada acció significativa queda registrada. Les promotion gates es reflecteixen en events de canvi de mandate. |
| **E** (promotion gates) | `kind:34002` (new version) | Un nou mandate signat per l'owner amb autoritat expandida = promocion gate superada. |
| **F** — Monitoring & Incidents | `kind:34003` (append-only) | L'audit log és immutable. Qualsevol incident queda registrat. |
| **F** (change log) | `kind:34002` (versioned) | Cada canvi de mandate és un nou event NIP-33. El `d` tag és el mateix; el `created_at` més recent és la versió activa. |
| **G** — Sign-offs | `kind:34006` — `nodus:emergency-resume` | Sign-off de l'owner per re-autorització. |
| **G** (revocation) | `kind:34005` — `nodus:emergency-stop` | Panic button. L'owner para TOTS els DWs del tenant. |
| **G** (re-authorization cadence) | Events `kind:34002` (created_at) | La cadència es defineix com a interval entre versions de mandate. |

### 4.2 És l'ACAP del WEF implementable sobre Nostr?

**Sí, completament.** I amb avantatges respecte a una implementació tradicional:

| Requisit ACAP | Com ho fa Nodus | Avantatge Nodus |
|---|---|---|
| Identity verificable | Clau BIP-340 (npub/nsec) | Auto-sobirana, cap PKI central |
| Authority no-additiva (multi-agent) | NIP-26 delegation + mandate ref | Verificable per qualsevol, sense servidor |
| Consequential events register | `hitl_required` array al mandate | Executable no només documentable |
| Checkpoints humans | `kind:10004` HITL_RESPONSE signat | Prova criptogràfica, no log |
| Audit immutable | `kind:34003` append-only | No es pot eliminar ni modificar |
| Emergency stop | `kind:34005` en <30 segons | Temps real, no consulta de document |
| Machine-readable | JSON signat des del dia 1 | Sense "evolution" pendent |
| Cross-organizational | Multi-relay federation | Nadiu a Nostr (no cal plataforma) |
| Agent passport | npub + kind:34010 KYC Claim | Portable, verificable, sense intermediari |

---

## 5. Sistema d'Enforcement en Temps Real

### 5.1 El problema de l'ACAP (WEF)

L'ACAP del WEF és un **document**. L'enforcement és:

- **A nivell organitzatiu:** consultants i auditors revisen l'ACAP periòdicament
- **A nivell tècnic:** cada organització implementa controls *ad hoc* (IAM, orchestration, logging)
- **A nivell de verificació:** normalment *a posteriori* — després d'un incident es consulta l'ACAP per veure si l'agent estava autoritzat

**El WEF ho sap.** Per això diu que l'ACAP "should evolve towards … policy-as-code format that enables version control, traceable change management and runtime enforcement." Però **no especifica com**.

### 5.2 La solució Nodus: 3 capes d'enforcement

```
┌──────────────────────────────────────────────────┐
│                                                   │
│   CAPA 1: RELAY WHITELIST                        │
│   (el relay rebutja npubs no autoritzats)         │
│                                                   │
├──────────────────────────────────────────────────┤
│                                                   │
│   CAPA 2: POLICY RELAY (kind:34002 enforcement)   │
│   (el relay avalua cada event contra el mandate)  │
│                                                   │
├──────────────────────────────────────────────────┤
│                                                   │
│   CAPA 3: AUDIT LOG IMMUTABLE (kind:34003)        │
│   (cada acció aprovada queda registrada)          │
│                                                   │
└──────────────────────────────────────────────────┘
```

#### Capa 1: Relay Whitelist (Perímetre)

El relay Nodus OS manté una **whitelist de npubs** que poden publicar-hi events.

- Un DW sense npub a la whitelist **no pot ni connectar-se**
- Això implementa els **Deployment Context Tiers** del WEF:
  - **L1 (single-org):** relay privat, whitelist de l'organització
  - **L2 (multi-org, same platform):** relay compartit, whitelist de trusted orgs
  - **L3 (cross-boundary):** federació de relays amb verificació creuada

**Verificació:**
```
DW connecta → relay check: npub ∈ whitelist?
  ├── No → CONNECTION REJECTED
  └── Sí → procedeix a Capa 2
```

#### Capa 2: Policy Engine (Enforcement Granular)

Quan un DW envia un event al relay, el Policy Engine (integrat al relay o com a proxy layer) **avalua l'acció contra el mandate vigent**:

```
1. L'event ve d'un npub amb kind:34000 (dw-profile) actiu?
   ├── No → REJECT (agent no registrat)
   └── Sí →

2. L'npub té un kind:34002 (mandate) vigent? (valid_from ≤ now < valid_until)
   ├── No → REJECT (sense autorització)
   └── Sí →

3. L'acció sol·licitada (tag "action" o deduïda del content) està a "capabilities"?
   ├── No → REJECT (acció no autoritzada)
   └── Sí →

4. L'acció està a "limits" (prohibited)?
   ├── Sí → REJECT (acció prohibida)
   └── No →

5. L'acció està a "hitl_required"?
   ├── Sí → REQUEREIX kind:10004 HITL_RESPONSE
   │   ├── Té HITL aprovat? → procedeix
   │   └── No → REJECT (checkpoint no superat)
   └── No →

6. TOT OK → SIGNA l'event i PROPAGA + kind:34003 (audit)
```

**On s'executa el Policy Engine?**

Dues opcions complementàries:

**Opció A: Policy Relay (recomanada per a entorns corporatius)**
Com descriu la secció 5.6 del Nodus SPEC. El DW **no té la clau**. Envia events sense signar al Policy Relay, que els signa només si el mandate ho permet.

```
DW (no key) ──unsigned event──► Policy Relay
                                │
                                ├── mandate OK? → SIGN → relay → network
                                └── mandate KO? → REJECT + log
```

**Opció B: Relay Write-Policy (plugin)**
El relay incorpora un plugin que avalua events **abans d'acceptar-los**. Fins i tot si el DW arriba amb un event ja signat, el relay pot rebutjar-lo si viola el mandate.

Totes dues opcions poden coexistir: el Policy Relay fa enforcement a nivell de signatura, i el relay plugin fa enforcement a nivell d'acceptació.

#### Capa 3: Audit Log Immutable (kind:34003)

Cada acció aprovada pel Policy Engine genera automàticament un `kind:34003` — l'audit log immutable.

Conté:
- Qui (npub del DW)
- Amb quina autoritat (event_id del mandate)
- Quan (created_at)
- Què va fer (hash de l'acció)
- En quin context (session_id, mandate ref)

**Propietats crítiques:**
- **Append-only:** el relay rebutja DELETE i UPDATE sobre kind:34003
- **Verificable:** qualsevol pot verificar la signatura
- **Encadenable:** cada event referència el mandate sota el qual es va executar

Això implementa la secció F de l'ACAP (Monitoring, Incidents & Change Log) **de forma molt més forta** que un document que es consulta manualment.

---

## 6. Política d'Enforcement al Relay

### 6.1 Regles de Write-Policy

Un relay Nodus OS conformant implementa les següents regles de write-policy:

| # | Regla | Origen |
|---|---|---|
| 1 | REJECT DELETE/UPDATE on kind:34002 | Nodus SPEC 5.1 |
| 2 | REJECT DELETE/UPDATE on kind:34003 | Nodus SPEC 5.1 |
| 3 | REJECT kind:34002, kind:34005 signats per entity_type:digital_worker | Nodus SPEC 5.9 |
| 4 | REJECT events de DW si kind:34005 actiu per al tenant | Nodus SPEC 5.6 |
| 5 | REJECT events si el mandate del DW (kind:34002) no cobreix l'acció | **Aquesta proposta (ACAP enforcement)** |
| 6 | REJECT events si no porten relay_proof tag (enforcement mode) | Nodus SPEC 5.6 |

### 6.2 Mapatge de Regles ACAP a Write-Policy

| Regla ACAP | Implementació al Relay |
|---|---|
| "Downstream agent operates under intersection of permissions" | El relay verifica NIP-26 delegation: el DW invocat només pot executar accions permeses pel seu mandate + el del DW invocant |
| "Consequential events require checkpoints" | El relay verifica que kind:10004 (HITL_RESPONSE) existeixi i sigui vàlid abans de propagar events d'accions marcades com a `hitl_required` |
| "Material changes require re-authorization" | El relay compara el `valid_until` del mandate amb la data actual. Si ha expirat, no propaga events |
| "Economic boundaries" | El relay manté un comptador de cost per sessió/període. Si se supera el `max_auto_cost_eur`, requereix HITL |
| "Observation in production" | El relay manté mètriques de drift (frequency d'errors, pattern de tool use) i alerta si es detecten anomalies |

### 6.3 Exemples d'Enforcement

#### Exemple 1: DW intenta enviar un email sense autorització

```
DW Athena envia kind:10002 amb action:send_email
  → Policy Engine consulta kind:34002 d'Athena
  → "send_email" NO està a capabilities
  → REJECT + kind:34003 (audit: "attempted unauthorized action")
  → Retorna error al DW
```

#### Exemple 2: DW intenta una acció HITL-required sense aprovació humana

```
DW Athena envia kind:10002 amb action:financial_transfer
  → Policy Engine consulta mandate
  → "financial_transfer" ∈ hitl_required
  → No hi ha kind:10004 associat a aquesta sessió
  → REJECT + genera kind:10003 (HITL_REQUEST) al relay per a l'owner
  → Espera kind:10004 o timeout
```

#### Exemple 3: Promocion Gate — expandir autoritat

```
Owner vol que Athena pugui fer transferències financeres fins a 1000€
  → Owner signa NOU kind:34002 amb "financial_transfer" a auto_approve
    i max_auto_cost_eur: 1000
  → El relay publica el nou mandate (NIP-33: mateix d tag)
  → El relay detecta el canvi: compara amb mandate anterior
  → Registra kind:34003: "authority expanded — financial_transfer auto-approved up to 1000€"
  → A partir d'aquest moment, Athena pot fer transferències ≤1000€ sense HITL
```

---

## 7. Cicles de Vida i Phase Gates sobre Nostr

### 7.1 ACAP Phase 1: System Design & Assessment → Nostr Events

| Pas ACAP | Event Nostr | Qui signa |
|---|---|---|
| A: Identity & Scope | `kind:34000` (dw-profile) | Owner |
| B: Operating Context | `kind:34000` (capabilities + limits) | Owner |
| C: Authority & Consequential Events | `kind:34002` (mandate, versió inicial) | Owner |
| D: Controls & Enforcement (design) | `kind:34004` (mcp-server-profiles) | Owner/IT |
| E: Evaluation Evidence (design) | No event — documentat al mandate | Owner |
| **Phase Gate 1** | `kind:34002` publicat + `kind:34006` (sign-off) | Owner + Risk Owner |

### 7.2 ACAP Phase 2: Prepare & Deploy → Nostr Events

| Pas ACAP | Event Nostr | Qui signa |
|---|---|---|
| D: Controls implemented | Validació que kind:34004 cobreix tots els MCP servers necessaris | Automàtic/IT |
| E: Evaluation satisfied | `kind:34003` amb evidència de test | DW + SME |
| F: Monitoring initialized | Primers `kind:34003` d'heartbeat | DW |
| G: Signed for production | `kind:34006` (resume + sign-off) | Deployment Owner + Risk Owner |
| **Phase Gate 2** | Totes les condicions complides → `kind:34006` amb d tag del tenant | Ambdues parts |

### 7.3 ACAP Phase 3: Monitor & Improve → Nostr Events

| Pas ACAP | Event Nostr | Qui signa |
|---|---|---|
| F: Observe behaviour | Stream de `kind:34003` | DW |
| C: Re-validate authority | `kind:34002` (nova versió si canvia) | Owner |
| D: Re-validate controls | `kind:34004` (actualització si canvia) | IT |
| E: Recalibrate thresholds | `kind:34002` (nou max_auto_cost, etc.) | Owner |
| G: Re-authorization | `kind:34006` (nou sign-off) | Risk Owner |
| **Decommission** | `kind:34005` (emergency stop) + owner retira kind:34002 | Owner |

### 7.4 Phase Gates Com a Events Verificables

Cada phase gate es materialitza com a un o més events Nostr:

```
Phase Gate 1 (design approved):
  kind:34002 (mandate signed by owner)
  └── amb tag ["acap_gate", "1"]

Phase Gate 2 (release validated):
  kind:34003 (evidence test results)
  kind:34006 (sign-off for production)
  └── ambdós amb tag ["acap_gate", "2"]

Phase Gate 3 (continued authorization):
  kind:34006 renovat segons cadència
  └── amb tag ["acap_gate", "3"]
```

**Avantatge sobre l'ACAP documental:** qualsevol pot verificar que un phase gate es va superar consultant el relay. No cal accedir a un document intern a l'organització.

---

## 8. Complementarietat Estratègica

### 8.1 On l'ACAP del WEF és bo (i el Nodus no ho toca)

L'ACAP és excel·lent en:

- **Definir rols humans:** adopter, SME, risk owner, supervisor, deployment owner — el Nodus no defineix roles humans, ni ha de fer-ho.
- **Processos d'aprovació:** qui aprova què, amb quina cadència — és feina de l'organització, no del protocol.
- **Criteris d'avaluació:** què constitueix "acceptable performance" — depèn del context de negoci.
- **Risk classification:** com mapejar el risc de l'agent a la taxonomia de risc de l'empresa.

**L'ACAP és el "què". El Nodus és el "com".** Són complementaris.

### 8.2 On el Nodus supera l'ACAP

| Àrea | ACAP (document) | Nodus (protocol) |
|---|---|---|
| Enforcement | A posteriori | En temps real |
| Identity | Depend de plataforma | Auto-sobirana |
| Audit trail | Logs (mutables) | Events signats (immutables) |
| Cross-org | "Governance challenge" | Federació de relays |
| Emergency stop | "Should be possible" | <30 segons, verificable |
| Verificació externa | Requereix accés a docs interns | Consulta al relay |
| Machine-readable | Objectiu futur | Des del primer event |
| Open source | No (cada empresa implementa) | CC BY 4.0 |

### 8.3 El factor polític: la batalla per la governança dels agents

L'ACAP del WEF és **inevitablement un estàndard de les Big Tech i Big Four**. No per malícia, sinó per composició: els qui seuen a la taula són Microsoft, Google, Meta, Capgemini, PwC, EY, BCG, Salesforce, SAP, ServiceNow, AWS, OpenAI, IBM...

> **Les empreses que defineixen l'estàndard de governança d'agents són les mateixes que venen les plataformes on els agents han de viure.**

Això no és un conflicte d'interessos tècnic — **és un problema de mercat**. Una pime que adopti l'ACAP tal com el WEF el planteja acabarà:
- Pagant consultoria a Capgemini o PwC per implementar-lo
- Comprant llicències a Microsoft o Salesforce per allotjar els agents
- Dependre d'AWS o Google per a la identitat i l'audit trail

**L'estàndard ACAP és bo. El model de negoci amb què s'implanta és el problema.**

El Nodus Protocol ofereix **una alternativa des de la base**:

- **Open Source (CC BY 4.0):** qualsevol pot implementar-lo, auditar-lo, contribuir-hi — sense llicències ni consultoria obligatòria
- **Descentralitzat:** no depèn d'una plataforma, d'un proveïdor de cloud, d'un walled garden. Cada organització controla el seu relay
- **Auto-sobirà:** cada empresa controla les seves claus criptogràfiques, la seva governança, el seu perímetre
- **Cost d'entrada baix:** un relay pot córrer en un Raspberry Pi. No cal infraestructura corporativa
- **No capturable:** cap Big Tech pot "comprar" Nostr o canviar el protocol unilateralment

**L'estratègia: Nodus com a implementació oberta de l'ACAP, impulsada per les associacions de pimes (SME United, SME Digital United, PIMEC), no per les Big Four.**

No es tracta de dir "l'ACAP està malament" sinó: **"l'ACAP descriu què cal fer. El Nodus demostra com fer-ho sense lligar-se a cap plataforma ni a cap consultora."**

> **Per a les pimes europees, no hi ha alternativa.** O implementen la governança d'agents dins de l'ecosistema Big Tech, o tenen una opció oberta, verificable i econòmicament accessible. El Nodus Protocol és aquesta opció.

---

## 9. Riscos i Oportunitats

### 9.1 Riscos

| Risc | Severitat | Mitigació |
|---|---|---|
| **L'ACAP es converteix en estàndard ISO/regulatori** (ex: ISO 42001) | Alta | Posicionar Nodus com a implementació oberta de referència abans que es consolidi |
| **Big Tech implementen ACAP dins dels seus walled gardens** (ex: Microsoft Copilot + ACAP) | Mitjana | Diferenciació: enforcement en temps real, open source, descentralització |
| **El WEF ignora Nodus** perquè no seiem a la taula | Mitjana | No cal seure a la taula del WEF. L'estratègia és anar a les institucions de pimes (SME United, PIMEC) i des d'allà fer força com a alternativa oberta |
| **Complexitat d'implementació** per a PIMES | Baixa-mitjana | Kits d'implementació, referències, documentació |

### 9.2 Oportunitats

| Oportunitat | Impacte | Acció |
|---|---|---|
| **Ser la primera implementació de referència d'ACAP** | Molt alt | Publicar aquest document + working example |
| **Posicionar Nodus com a "ACAP-compliant"** | Alt | Incorporar el mapatge a la documentació oficial (SPEC.md) |
| **Presència institucional** | Alt | "How Nodus Protocol implements WEF ACAP — an open alternative for SMEs" a esdeveniments d'SME United, PIMEC i comissió europea |
| **Atracció d'empreses que volen ACAP sense lligar-se a Big Tech** | Molt alt | Proposta de valor única: "ACAP sense vendor lock-in" |
| **Col·laboració amb el WEF** | Alt | Oferir Nodus com a sandbox per a experiments d'agent passports |
| **Finançament europeu** (Digital Europe, Horizon) | Alt | "Infraestructura oberta per a governança d'agents — compliant amb estàndards internacionals" |

---

## 10. Full de Ruta Recomanat

### Fase 1: Documentació (aquesta setmana)
- [x] Anàlisi de l'ACAP del WEF
- [x] Mapatge ACAP ↔ Nodus Kinds
- [ ] Publicar aquest document al repositori del Nodus Protocol (`docs/acap-nodus-compliance.md`)
- [ ] Preparar un technical brief de 2 pàgines per a distribució externa

### Fase 2: Implementació (pròximes 2-4 setmanes)
- [ ] Implementar el **Policy Engine** com a proxy layer (Opció B, compatible amb relay existent)
- [ ] Afegir al relay Nodus OS les **write-policy rules** de la secció 6.1
- [ ] Crear un **test suite** que demostri: creació de DW → assignació de mandate → acció permesa → acció denegada → HITL → promocion gate
- [ ] Publicar un **working example** amb un DW real que implementi el cicle ACAP complet

### Fase 3: Lobby Institucional (pròximes 4-8 setmanes)
- [ ] Contactar amb **SME United** (Brussel·les) — la veu de les pimes a Europa
- [ ] Contactar amb **SME Digital United** — el braç digital de SME United
- [ ] Contactar amb **PIMEC** — com a pont cap a les institucions europees de pimes
- [ ] Identificar altres **associacions de pimes** que puguin sumar-se al suport del protocol
- [ ] Posicionar Nodus com a **"Agent Governance per a pimes"** — un estàndard que no requereix contractar Capgemini, ni comprar Microsoft, ni dependre de Big Tech
- [ ] Publicar un **post tècnic** (LinkedIn/X): "We implemented the WEF ACAP framework on a decentralized protocol. Here's why it matters for SMEs."

### Fase 4: Expansió (pròxims 3-6 mesos)
- [ ] Oferir **certificació ACAP-Nodus** per a DWs
- [ ] Incorporar al Nodus SPEC com a **apèndix normatiu** (no canvia el protocol, mostra compatibilitat)
- [ ] Explorar **projecte europeu** (Horizon Europe / Digital Europe / Single Market Programme) per a "Open Agent Governance Infrastructure for SMEs"
- [ ] Publicar el mapatge com a **proposta d'estàndard obert** (IETF, W3C, o CEN-CENELEC via les associacions de pimes)

---

## Annex: Correspondència Ràpida

| ACAP | Nodus |
|---|---|
| A: Identity & Scope | `kind:34000` (dw-profile) |
| B: Operating Context | `kind:34000` (capabilities/limits/transports) + `kind:34010` (KYC) |
| C: Authority & Consequential Events | `kind:34002` (mandate) |
| D: Controls & Enforcement | `kind:34004` (MCP Server Profile) + `kind:10010-10013` (A2A) |
| E: Evaluation Evidence & Promotion Gates | `kind:34003` (audit) + versions de `kind:34002` |
| F: Monitoring, Incidents & Change Log | `kind:34003` (immutable audit stream) |
| G: Sign-offs & Re-authorization Cadence | `kind:34005/34006` (emergency stop/resume) |
| Agent Passport | npub + `kind:34010` (KYC Corp Claim) |
| Phase Gate 1 (design approved) | `kind:34002` publicat + tag `acap_gate:1` |
| Phase Gate 2 (release validated) | `kind:34006` sign-off + tag `acap_gate:2` |
| Phase Gate 3 (continued authorization) | `kind:34006` renovat + tag `acap_gate:3` |
| Enforcement (runtime) | Policy Relay + NIP-26 delegation verificació |
| Federation (L2/L3) | Multi-relay + `relay_hint` + `federation_scope` |

---

*Document creat per Catafau (DIRCOM, Nodus Protocol) per a Quirze Salomó. Juny 2026. Les mencions a CBCat, BBW i Democracy4All han estat eliminades en reconèixer que són iniciatives del passat. L'estratègia de lobby actual es centra en SME United, SME Digital United, PIMEC i altres associacions de pimes a Brussel·les.*
