# Agenda ADVEC Sobradinho — Documento de Continuidade

> Última atualização: 17/novembro/2026 (restaurar eventos fixos cancelados)
>
> Este documento serve para retomar o projeto em conversas futuras. Está dividido em **STATUS ATUAL**, **STACK TÉCNICO**, **REGRAS DE NEGÓCIO** e **WORKFLOWS**. Anexe esse arquivo + o `Agenda.html` atual no início de uma nova conversa pra o Claude entender o contexto sem você precisar reexplicar tudo.

---

## 🆕 NOVIDADES DESTA VERSÃO (sessão de 17/nov/2026)

> ✅ **Não exige nada no Firebase Console.** Usa a coleção `cancelled` e a regra que já existe (`allow create, delete: if isEditor()`).

### Restaurar evento fixo cancelado 🆕
1. **Problema resolvido:** ao excluir (🗑) um culto fixo num dia (ex.: Culto da Vitória de 10/12), ele ficava cancelado e não havia botão para trazê-lo de volta (a função `uncancelRecurring` já existia, mas sem tela).
2. Agora, no **dia aberto** e nos **cards da visão Semana**, editores/admin veem uma faixa tracejada **"Cancelado neste dia"** com o nome, o horário e o botão **↩ Restaurar**.
3. Restaurar apaga o cancelamento (`cancelled/{slotId}`) **e o override** (`overrides/{slotId}`) daquela data, então o evento volta **com os parâmetros padrão** do culto (horário, local, descrição), como as demais datas. Pregador/observação/flyer editados só naquela data são zerados.
4. Vale para **cultos fixos e eventos recorrentes criados pela tela** (qualquer `slotId` cancelado). Eventos manuais excluídos continuam sem restauração (são apagados de verdade).
5. Código: `getCancelledRecurringForDate()`, `cancelledBoxHtml()`, `bindRestoreHandlers()` (antes de `eventCardHtml`); CSS `.cancelled-box`, `.restore-btn`. Chamados em `renderDayModal()` e `renderWeekView()`.
6. Limite: na visão **Mês** não aparece (só dentro do dia aberto, clicando no dia).

---

## Sessão de 21/out/2026 (anterior)

> ✅ **Não exige nada no Firebase Console.** Tudo fica em `config/ministryOptions`, coberto pela regra de `config` que já existe.

### Observações aparecem na visão Mês (computador) 🆕
1. **Problema resolvido:** as observações (ex.: lista de GVVs com líder e anfitrião) só apareciam abrindo o dia.
2. Na pílula do mês, se o evento tiver observação, ela aparece abaixo do título/pregador numa faixa clara, **respeitando as quebras de linha**. Mostra até 8 linhas; o resto fica no tooltip e no dia aberto.
3. **Só no computador** (tela ≥ 1024px). No celular nada muda (a pílula vira bolinha).
4. Visão **Semana** já mostrava as observações nos cards — sem mudança.
5. CSS: `.event-pill .pill-notes` (dentro de `@media (min-width: 1024px)`). Gerado em `renderCalendar()` (variável `notesTxt`).

### Observações padrão nos eventos recorrentes 🆕
1. Na aba **Eventos Recorrentes**, novo campo **"Observações padrão (opcional)"** (textarea, aceita várias linhas).
2. O texto aparece em **todas as datas** do evento (card, dia aberto e pílula do mês no computador).
3. Continua editável **por data**: abrir o evento no dia e mudar as observações grava um override só daquela data.
   - ⚠️ Datas que já tinham sido editadas antes (override salvo) mantêm o que foi salvo nelas, inclusive observação em branco — ajuste nessas datas se precisar.
4. Guardado em `config/series` → campo **`notes`** em cada item. Gerado em `getRecurringEventsForDate` (`if (sr.notes) t.notes = sr.notes`).

### Tipo de evento "Sem local" 🆕
1. Na aba **Tipos de Eventos** (admin), ao lado de "Pede pregador(a)", cada tipo tem a etiqueta **"Pede local" ⇄ "✓ Sem local"**.
2. Tipos marcados: some o **"Local a definir"** do card, e o local deixa de contar para o evento parecer "vazio" (`isRecurringEmpty`). Se alguém preencher um local, aparece normal.
3. Guardado em **`config/ministryOptions`** → `{ noPreacher: [...], noLocation: [...] }` (mesmo documento, gravação com merge). Helpers: `ministryNoLocation()`, `setMinistryNoLocation()`, `noLocationToggleHtml()`, `ministryOptionTogglesHtml()` (as duas etiquetas juntas).

---

## Sessão de 20/out/2026 (anterior)

> ✅ **Não exige nada no Firebase Console.** Os eventos ficam em `config/series`, coberto pela Security Rule de `config` que já existe.

### Tipo de evento "Sem pregador(a)" 🆕
1. **Problema resolvido:** eventos sem pregador/professor apareciam com "Pregador(a) a definir".
2. Na aba **Tipos de Eventos** (admin), cada tipo (fixo ou personalizado) tem uma etiqueta clicável **"Pede pregador(a)" ⇄ "✓ Sem pregador(a)"**.
3. Tipos marcados: somem "Pregador(a) a definir", "Professor(a) a definir" e "Lição a definir" (pílula do mês e card), e o evento não fica com aparência de "vazio" (`isRecurringEmpty` retorna false). Se alguém preencher um nome, aparece normal.
4. Guardado em **`config/ministryOptions`** → `{ noPreacher: [chaves] }` (regra de `config`, sem regra nova; entra no backup). Helpers: `ministryNoPreacher()`, `setMinistryNoPreacher()`, `noPreacherToggleHtml()`.
5. 💡 Dica: para um evento sem pregador, crie um **tipo personalizado** próprio e marque-o, em vez de marcar um tipo geral como "Cultos".

### Eventos recorrentes criados pela tela 🆕
1. **Problema resolvido:** não havia como incluir um evento fixo novo sem mexer no HTML. Ex.: evento **quarta sim, quarta não**.
2. **Nova aba "Eventos Recorrentes"** (⚙ → somente admin). Campos: nome, tipo de evento, repetição, dia da semana, começa em, termina em (opcional), horário, local padrão, pregador(a) padrão (opcional). Mostra **prévia das próximas 8 datas** enquanto preenche.
3. **Repetições disponíveis:** toda semana · a cada 2 semanas (quinzenal) · a cada 3 · a cada 4 · uma vez por mês (1ª, 2ª, 3ª, 4ª ou **última** ocorrência do dia no mês).
4. **Quinzenal:** a contagem parte da 1ª data do dia escolhido a partir de "Começa em" (âncora = `seriesAnchorDate`).
5. **Cada data funciona como culto fixo:** `templateId = ser_<id>`, slotId `ser_<id>_YYYY-MM-DD` → pregador por data, local, observações, flyer, cancelar dia avulso e "🔑 Aplicar nesta e nas próximas" funcionam igual.
6. **Editar / encerrar / excluir:** ✏️ carrega no formulário. Para **encerrar**, preencher "Termina em" (último dia). Se mudar dia/frequência de um evento que já começou, o sistema pergunta se deve **preservar o passado** (encerra o antigo ontem e cria um novo a partir de hoje). 🗑 Excluir tira o evento de **todas** as datas, inclusive passadas.
   - ⚠️ Ao preservar o passado, o evento "novo" ganha outro `id` — escalas por data que já estivessem feitas para datas futuras do antigo não são levadas junto (refazer na data).
7. **Onde fica guardado:** `config/series` → `{ items: [ { id, title, time, ministry, location, preacher, notes 🆕, freq ("weekly"|"monthly"), interval (1-4), nth (1-4 ou -1), weekday (0-6), start, end } ] }`. Entra no backup junto com `config`.
8. Helpers: `getRecurringSeries()`, `seriesOccursOn()`, `seriesPhrase()`, `seriesNextDates()`, `saveRecurringSeries()`, `prevDayStr()`. Geração em `getRecurringEventsForDate` (bloco "EVENTOS RECORRENTES CRIADOS PELA TELA").

---

## Sessão de 16/set/2026 (anterior)

> ✅ **Esta versão NÃO exige nada no Firebase Console.** A nova funcionalidade guarda tudo em `config/recurring`, coberto pela Security Rule de `config` que já existe. Se aquela regra ainda não tiver sido publicada, a agenda continua funcionando (usa as regras padrão do código) — só não dá para salvar mudanças novas pela tela.

### Cultos fixos podem mudar de DIA DA SEMANA 🆕
1. **Problema resolvido:** o "🔑 Aplicar nesta e nas próximas ocorrências" só replicava descrição e horário — não o **dia**. Trocar um culto fixo de dia exigia mexer no HTML.
2. **Nova aba "Cultos Fixos"** (⚙ → somente admin): move um culto fixo para outro dia da semana **a partir de uma data**, ou **encerra** o culto (deixa de aparecer daquela data em diante). Tudo de uma vez, sem abrir dia por dia.
3. **Mudança já aplicada (padrão do código), valendo de 06/10/2026:**
   - **Culto da Palavra** — quarta → **terça**
   - **Encontro de Crescimento das Vitoriosas** — quinta → **quarta**
   - **Culto da Vitória** — segunda → **quinta**
   - **Santa Ceia mensal** — 3ª quarta → **3ª terça** (o Culto da Palavra dá lugar a ela na terça, como dava na quarta)
4. **Nada é perdido.** A regra vale só de `from` em diante: todas as semanas anteriores continuam idênticas (pregadores, observações, cancelamentos, flyers). Cada mudança aparece numa lista e pode ser **desfeita** (🗑), voltando tudo ao estado anterior. Os `templateId` não mudaram (`seg_vit_2000`, `qua_palavra_2000`, `qui_vitoriosas_2000`), então nenhum override/cancelamento antigo é órfão.
5. **"Levar junto o que já está escalado"** (checkbox, ligado por padrão): ao mover de dia, copia pregador(a), lição, local, observações e flyer das **próximas** ocorrências do dia antigo para as datas do dia novo, na mesma semana. É aditivo — não apaga o dia antigo nem sobrescreve o que já existir no dia novo.
6. **Santa Ceia mensal virou movível também.** A regra "3ª quarta" foi generalizada para "**3ª ocorrência do dia escolhido**" (`isThirdWeekdayOfMonth`), então ela funciona em qualquer dia da semana. Na aba aparece como "Santa Ceia (3ª do mês)" e o seletor mostra "3ª terça do mês" em vez de "toda terça".
7. **Cultos que NÃO podem ser movidos:** Santa Ceia Regional da quinta e os cultos de domingo (dependem de outras âncoras).
8. **Santa Ceia Regional (BSB) continua na quinta após a 3ª QUARTA** — não acompanhou a Santa Ceia local, por ser evento da Advec Brasília. ⚠️ Em alguns meses as duas passam a cair em semanas diferentes (1ª vez: **set/2027** — Regional 16/09, Santa Ceia local 21/09). Se um dia isso precisar mudar, é em `isThursdayAfterSantaCeia()`.
9. **Efeitos colaterais conhecidos da virada de 06/10** (tudo ajustável na tela ou cancelando o dia avulso):
   - **Semana de 05–11/10:** a segunda 05/10 ainda é anterior à data de corte, então tem **Culto da Vitória na segunda 05/10 e também na quinta 08/10**. Para não ter os dois, mude a data da regra para 05/10 na aba Cultos Fixos, ou cancele o evento do dia 05/10.
   - **Semana da Santa Ceia:** a terça é da **Santa Ceia** (sem Culto da Palavra, como era na quarta), a quarta fica só com o **Encontro de Crescimento**, e a quinta tem **Culto da Vitória + Santa Ceia Regional**.
10. **Tela de Configurações maior no computador** — a janela do ⚙ deixou de ter largura fixa de 960px: virou a classe `.modal-lg` (CSS junto do `.modal`), com **1180px a partir de 1024px de tela** e **1360px a partir de 1440px**, altura até 88–90vh. No celular nada muda.
11. **Onde fica guardado:** `config/recurring` → `{ rules: [ { id, templateId, weekday (0-6 ou null), from } ] }`. Enquanto o documento não existir, valem os padrões do código (`RECURRING_RULES_DEFAULTS`). Entra no backup junto com `config`.

---

## Sessão de 13/jun/2026 (anterior)

> ⚠️ **ATENÇÃO — esta versão EXIGE uma mudança no Firebase Console** (as anteriores não exigiam): publicar a Security Rule da nova coleção `config`. Sem isso, o admin não consegue salvar o cabeçalho. Ver "Security Rules". O resto do app degrada com elegância (cabeçalho cai nos valores padrão).

### Navegação & visão
1. **Abre sempre no mês/semana atual** — ao carregar, a agenda já vai para o mês de hoje; em visão semanal, abre na semana corrente. Refatorado de `goToToday` numa função `setTodayView()` chamada uma vez no carregamento (com fallback para maio/2026 se a data for anterior ao início da agenda).

### Eventos recorrentes
2. **Santa Ceia Regional (BSB)** — evento novo: **toda quinta logo após a Santa Ceia da 3ª quarta**, 20h, local **Advec Brasília**, ministério `ADVEC_BSB` (tarja laranja), pregador padrão **Pr. Cesar Carreira** (editável por data). Cai sempre entre os dias 16–22 (mesmo mês). Helper `isThursdayAfterSantaCeia(date)` (a quarta da véspera é a 3ª quarta).
3. **Seminário Teológico removido** das quintas (`qui_seminario_1900` deletado). Quinta agora tem só: Encontro das Vitoriosas (20h) + Santa Ceia Regional (nas quintas após a SC da quarta).
4. **Pregadores padrão nas Santas Ceias** (todos pré-preenchidos mas **editáveis por data**, via novo parâmetro `opts.preacher` em `template()`):
   - **Domingo — Santa Ceia (Transmissão)** (09:30 e 18:30): **Pr. Silas Malafaia**
   - **3ª quarta — Santa Ceia** (20h): **Pr. Márcio Nascimento**, local **Advec Sobradinho**
   - **Quinta — Santa Ceia Regional** (20h): **Pr. Cesar Carreira**, local **Advec Brasília**
   > ℹ️ Se algum desses nomes não estiver no cadastro de Pregadores, ele aparece normal no calendário e mostra "(não cadastrado)" só no seletor ao editar. Pode cadastrá-los na aba Pregadores(as) para sumir o aviso.

### Imagens — fotos de pregadores e flyers de eventos
5. **Foto do pregador(a)** no cadastro (aba Pregadores): botão de câmera para adicionar/trocar, avatar redondo na lista, e lightbox para **ver / baixar / remover**. Campo `photo` na coleção `people`.
6. **Flyer do evento**: no modal de qualquer evento (manual ou recorrente) há o campo "Flyer" com **adicionar / ver / baixar / trocar / remover**. Quem tem flyer mostra **miniatura no card** e um **selo visível na pílula do mês**. Campo `flyer` em `events` e `overrides`.
   - ⚠️ O selo do mês foi corrigido nesta sessão: antes o ícone ficava *dentro* do título e era cortado pelo "…" quando o nome era longo. Agora o título trunca no seu próprio span (`.pill-title`) e o flyer é um **selo branco com ícone navy e sombra, fixo no fim da linha** (`.pill-flyer-badge`), que nunca é cortado.
7. **Arquitetura de imagens (importante):** as imagens são guardadas como **base64 (data URL) dentro do próprio documento do Firestore** — **não** usa Firebase Storage, então **nenhuma configuração extra de Storage/bucket é necessária**. Como o Firestore limita **1 MB por documento**, toda imagem é redimensionada/comprimida no navegador antes de salvar:
   - Fotos de pregador: lado máx. **512px**, alvo **< 250 KB**
   - Flyers: lado máx. **1400px**, alvo **< 750 KB**
   - Helpers: `fileToResizedDataUrl()`, `pickImageFile()`, `downloadDataUrl()`, `slugFilename()`, `openImageLightbox()`. As imagens entram no backup junto com seus documentos.

### Cabeçalho editável (admin) 🆕
8. **Nova aba "Cabeçalho"** em Configurações (somente admin) para editar a identidade exibida no topo:
   - Nome da igreja (ex.: ADVEC Sobradinho)
   - Ano (ex.: 2026)
   - Subtítulo cursivo do ano (ex.: "Ano da")
   - Tema do ano em destaque (ex.: ORAÇÃO)
   - Texto do versículo + referência (ex.: ORAI SEM CESSAR / 1 TS. 5:17)
   - Nome do **Pastor Dirigente** e do **Vice**
   - Persiste em **`config/site`** no Firestore; reflete no topbar, no `<title>` da aba e na **Programação Semanal (PDF)**. Funções `getSiteConfig()` (defaults + salvo), `applyHeaderConfig()` (aplica no DOM) e `saveSiteConfig()` (grava, admin-only). Defaults em `SITE_CONFIG_DEFAULTS`.
   - **Resolve a pendência antiga "Refatorar identidade num bloco CHURCH_CONFIG"** — agora a identidade é editável pela tela, sem mexer no HTML.
   - O botão "Restaurar padrão" chegou a existir e foi **removido** a pedido (sobrou só "Salvar cabeçalho").
9. **`config` incluído no backup** (`BACKUP_COLLECTIONS` e `BACKUP_RESTORABLE`). O export de backup ficou **tolerante**: se uma coleção falhar (ex.: `config` antes das rules), pula com aviso em vez de abortar o backup inteiro.
10. **Endurecimento**: binding do botão salvar do modal de evento agora usa `?.` (não quebra mais no modo somente-leitura, onde o botão não existe).

### Sessão de 01/jun/2026 (anterior)
1. **Correção da regra de avisos pela Santa Ceia** — o PDF semanal agora respeita que os avisos do mês só trocam **no culto de Santa Ceia do 1º domingo**. A semana cujo domingo é a própria Santa Ceia ainda mostra os avisos do mês anterior. Bug corrigido: a função `getNoticeForWeek` usava `>=` e mostrava o mês novo cedo demais; trocado para `>`.
2. **Favicon redondo com anel dourado** — favicon SVG no `<head>` que recorta a logo da ADVEC num círculo e desenha o anel dourado (`#C9A14A`). Reaproveita a logo base64; nenhum arquivo extra. Inclui `apple-touch-icon`.
3. **QR Codes para divulgação** — 3 PNGs (cartão completo, QR com logo, QR simples) apontando para a URL de produção.
4. **Migração de hospedagem Netlify → GitHub Pages**.
5. **Prévia do WhatsApp (Open Graph): DESCARTADA** — a versão de produção **não** contém Open Graph (a imagem do cartão é enviada manualmente junto com o link).
6. **Edição de eventos FIXOS (descrição + horário) + replicação admin** — descrição e horário dos recorrentes viraram editáveis (salvam override por data); só admin vê **"🔑 Aplicar nesta e nas próximas ocorrências"** (`applyRecurringToFuture`).
7. **Correção dos horários fixos de domingo (estrutura nova):** Santa Ceia manhã 10:00 → **09:30**; Celebração manhã 10:00 → **10:15**.

### Mudanças estruturais (15/mai → hoje)
1. **Tipos de Evento personalizados (`customMinistries`)** — admin cria/renomeia/exclui tipos próprios.
2. **Períodos Especiais (`specialPeriods`)** — faixas de datas com nome e cores que destacam o período no calendário.
3. **Backup / Restauração** — aba (admin) que exporta/restaura coleções em JSON (exceto `users`).
4. **🚚 MIGRAÇÃO Netlify → GitHub Pages** — Netlify pausou por limite do plano free (créditos por deploy, abr/2026). Novo endereço: `https://advecsobradinho.github.io/agendaoficial/`.

---

## 📌 STATUS ATUAL

**Aplicação em produção e funcionando — hospedada no GitHub Pages.**

- 🌐 URL produção: `https://advecsobradinho.github.io/agendaoficial/`
- ☁️ Hosting: **GitHub Pages** (grátis; deploy não consome créditos)
- 📦 Repositório: usuário GitHub `advecsobradinho`, repo `agendaoficial` (Public), branch `main`, pasta `/root`
- 🔐 Admin owner: `wllnbrgs@gmail.com` (auto-promovido a admin no primeiro login)
- 🔥 Backend: Firebase (Auth + Firestore)
- 📂 Projeto Firebase: `agenda-advec-sobradinho`
- ❌ Netlify: **abandonado** (não migrar nada de volta)

### Pendências conhecidas

1. **⚠️ Publicar a Security Rule de `config`** no Firebase Console (NOVO nesta versão — sem isso o cabeçalho editável não salva). Ver "Security Rules".
2. **Confirmar `advecsobradinho.github.io` nos Authorized Domains do Firebase** (Authentication → Settings → Authorized domains) — sem isso o login Google falha.
3. **Confirmar Security Rules das coleções** `customMinistries` e `specialPeriods` no Console.
4. **Endereço com subpasta** `/agendaoficial/` — se quiser URL mais curta, renomear o repo para `advecsobradinho.github.io` (opcional).
5. **Testar em mobile** (layout, touch targets, performance).
6. (Opcional) Cadastrar **Pr. Silas Malafaia** e **Pr. Cesar Carreira** na aba Pregadores para tirar o "(não cadastrado)" do seletor.

> ⚠️ **Armadilha resolvida (Windows):** ao renomear o arquivo no Windows com extensões ocultas, o `index.html` virou `index.html.html` e o GitHub Pages mostrava o README. Sempre confirme que o arquivo é **exatamente `index.html`** (um `.html` só) na raiz do repo. **Dica:** o Claude já entrega o arquivo nomeado `index.html`, então não precisa renomear.

---

## 🛠️ STACK TÉCNICO

### Arquivo principal
- **`Agenda.html`** (~332 KB, ~7.168 linhas) — single-file vanilla JS, sem build, sem dependências locais.
- No GitHub Pages, é entregue/colocado como `index.html` na raiz do repositório `agendaoficial`.

### Favicon (ícone da aba)
- Favicon **SVG redondo** embutido no `<head>`: recorta a logo base64 num círculo (`clipPath`) e desenha o anel dourado (`#C9A14A`). Sem arquivo externo.
- `apple-touch-icon` aponta para a logo JPEG base64.
- ⚠️ Favicon é fortemente cacheado — após atualizar, usar Ctrl+F5.
- ℹ️ O `<title>` da aba agora é dinâmico: `"{nome da igreja} — Agenda"` (vem da config do cabeçalho).

### Bibliotecas (via CDN, ES modules)
- Firebase v10.13.0: `firebase-app.js` (initializeApp), `firebase-auth.js` (Google + Email/Password), `firebase-firestore.js` (onSnapshot + CRUD).
  - ℹ️ **Não** usa `firebase-storage.js` — imagens vão em base64 no Firestore.
- Google Fonts: **Fraunces**, **Manrope**, **Allura** (cursiva do banner), **Oswald** (o "2026").

### Paleta de cores (CSS vars)
- `--navy-deep: #0B3A5B`, `--navy: #11507F`, `--gold: #C9A14A`, `--gold-light: #E5C97D`, `--gold-soft: #F1E4BD`, `--cream: #FAF6EC`, `--paper: #FFFDF7`
- Body: `#E8DCBE`; Gradient topbar: `#143240 → #264C5E → #2E5C70`

---

## 🔥 FIREBASE — Configuração

### `firebaseConfig` (hardcoded no Agenda.html)
```js
const firebaseConfig = {
  apiKey: "AIzaSyCbPaoAz3p2FcbWg1pnCMdlj6asE6m7pf0",
  authDomain: "agenda-advec-sobradinho.firebaseapp.com",
  projectId: "agenda-advec-sobradinho",
  storageBucket: "agenda-advec-sobradinho.firebasestorage.app",
  messagingSenderId: "91451080116",
  appId: "1:91451080116:web:4bf3679da1ce6a7b1fc6e1"
};
const ADMIN_EMAIL = "wllnbrgs@gmail.com";
```

### Authentication providers ativos
- **Google** (recomendado) e **Email/Password** (cadastro aberto + admin aprova)

### Authorized domains
- `localhost`, `agenda-advec-sobradinho.firebaseapp.com`, `advecsobradinho.github.io` (**produção atual**), `netlify.app` (legado — pode remover)

### Firestore — Localização
- `southamerica-east1` (São Paulo)

### Firestore — Coleções

| Coleção | Doc ID | Campos |
|---|---|---|
| `users/{uid}` | UID do Auth | `email`, `displayName`, `role`, `approved`, `createdAt` |
| `events/{auto-id}` | Auto | `title`, `date`, `time`, `ministry`, `preacher`, `professor`, `lesson`, `location`, `notes`, **`flyer`** 🆕, `isManual`, `featuredInWeekly`, `showInWeeklyProgram`, `createdAt`, `createdBy`, `updatedAt`, `updatedBy` |
| `overrides/{slotId}` | slotId do recorrente (ex: `qua_edp_2000_2026-05-13`) | `preacher`, `professor`, `lesson`, `location`, `notes`, **`flyer`** 🆕, `featuredInWeekly`, `showInWeeklyProgram`, `updatedAt`, `updatedBy` |
| `cancelled/{slotId}` | slotId | `cancelledAt`, `cancelledBy` |
| `people/{masterKey}` | normalize(name) sem acentos lowercase | `name`, `isGuest`, `isPinned`, **`photo`** 🆕 (base64 ou null), `_key` |
| `locations/{masterKey}` | normalize(name) | `name`, `_key` |
| `ministryLabels/{key}` | Chave do MINISTRIES (ex: CULTOS) | `label` (custom) |
| `customMinistries/{key}` | `CUSTOM_<SLUG>` | `label`, `color`, `text` |
| `specialPeriods/{auto-id}` | Auto | `name`, `startDate`, `endDate`, `bgColor`, `textColor`, `updatedAt` |
| `monthlyNotices/{YYYY-MM}` | ano-mês (ex: `2026-05`) | `biblicalReading`, `prayerMotives`, `monthVerse`, `verseReference`, `updatedAt`, `updatedBy` |
| **`config/site`** | fixo `site` | `churchName`, `year`, `yearScript`, `yearShout`, `verseTitle`, `verseRef`, `leaderName`, `viceName`, `updatedAt`, `updatedBy` |
| **`config/recurring`** | fixo `recurring` | `rules` (array de `{ id, templateId, weekday (0-6 ou null), from }`), `updatedAt`, `updatedBy` |
| **`config/ministryOptions`** | fixo `ministryOptions` | `noPreacher` (tipos que não pedem pregador), **`noLocation`** 🆕 (tipos que não pedem local), `updatedAt`, `updatedBy` |
| **`config/series`** 🆕 | fixo `series` | `items` (eventos recorrentes criados pela tela — ver Novidades), `updatedAt`, `updatedBy` |

> ℹ️ `config/recurring` usa a **mesma regra de `config` já existente** — não há Security Rule nova nesta versão.
>
> ℹ️ `flyer` (events/overrides) e `photo` (people) são **data URLs base64** — não precisam de regra de campo específica; as regras de coleção existentes já as cobrem.

### Firestore — Roles
- `admin` — pode tudo (usuários, tipos de evento, backup, **cabeçalho/config**, **dia dos cultos fixos**)
- `editor` — CRUD de eventos, pessoas (inclui foto), locais, períodos especiais, avisos (não usuários, não ministry labels/custom, não backup, **não config**)
- `viewer` — somente leitura (pode **ver/baixar** fotos e flyers, mas não editar)
- `pending` — bloqueado

### Security Rules (⚠️ confirmar/publicar no Console)

> 🆕 **OBRIGATÓRIO nesta versão:** adicionar a regra de `config` (linha marcada `// 🆕 13/jun`). As de `customMinistries`/`specialPeriods` também devem estar publicadas.

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function userDoc() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data;
    }
    function isAuthed() { return request.auth != null; }
    function isAdmin() { return isAuthed() && userDoc().role == 'admin'; }
    function isEditor() { return isAuthed() && (userDoc().role == 'admin' || userDoc().role == 'editor'); }
    function isApproved() { return isAuthed() && userDoc().approved == true; }
    function isOwnerEmail() { return isAuthed() && request.auth.token.email == 'wllnbrgs@gmail.com'; }

    match /users/{uid} {
      allow read: if isAuthed() && request.auth.uid == uid;
      allow read: if isAdmin();
      allow update, delete: if isAdmin();
      allow create: if isAuthed() && request.auth.uid == uid && (
        ( isOwnerEmail()
          && request.resource.data.role == 'admin'
          && request.resource.data.approved == true )
        ||
        ( request.resource.data.role == 'pending'
          && request.resource.data.approved == false )
      );
      allow update: if isAuthed() && request.auth.uid == uid && isOwnerEmail();
    }

    match /events/{id}          { allow read: if isApproved(); allow create, update, delete: if isEditor() && isApproved(); }
    match /overrides/{id}       { allow read: if isApproved(); allow create, update, delete: if isEditor() && isApproved(); }
    match /cancelled/{id}       { allow read: if isApproved(); allow create, delete: if isEditor() && isApproved(); }
    match /people/{id}          { allow read: if isApproved(); allow create, update, delete: if isEditor() && isApproved(); }
    match /locations/{id}       { allow read: if isApproved(); allow create, update, delete: if isEditor() && isApproved(); }
    match /ministryLabels/{id}  { allow read: if isApproved(); allow create, update, delete: if isAdmin(); }
    match /customMinistries/{id}{ allow read: if isApproved(); allow create, update, delete: if isAdmin(); }
    match /specialPeriods/{id}  { allow read: if isApproved(); allow create, update, delete: if isEditor() && isApproved(); }
    match /monthlyNotices/{id}  { allow read: if isApproved(); allow create, update, delete: if isEditor() && isApproved(); }
    match /config/{id}          { allow read: if isApproved(); allow create, update, delete: if isAdmin(); }  // 🆕 13/jun — cabeçalho editável (admin)
  }
}
```

---

## 📅 REGRAS DE NEGÓCIO

### Janela ativa da agenda
- `AGENDA_START = "2026-05-01"` — agenda começa em maio/2026
- `NEW_STRUCTURE_DATE = "2026-05-17"` — divide estrutura antiga/nova
- Janeiro–abril/2026: inativos (banner amarelo)
- 🆕 Ao abrir, o app vai para o **mês/semana atual** (`setTodayView()`); se hoje for antes de maio/2026, cai em maio/2026.

### Semana
- A semana é **Segunda → Domingo**. `weekStart` no state armazena a SEGUNDA-feira (YYYY-MM-DD).

### Eventos recorrentes (gerados em runtime — `getRecurringEventsForDate`)
- **Domingos antes de 17/05:** 09:30 + 18:30 Culto de Celebração (1º domingo = Santa Ceia Transmissão, pregador **Pr. Silas Malafaia**).
- **Domingos a partir de 17/05 (estrutura nova):**
  - Domingo comum: 09:00 EDP (`dom_edp_0900`) + **10:15** Celebração Manhã (`dom_cel_1000`) + 18:30 Celebração Noite (`dom_cel_1830`)
  - 1º domingo (Santa Ceia): 09:00 EDP + **09:30** Santa Ceia Transmissão (`dom_sc_1000`) + 18:30 Santa Ceia Transmissão (`dom_sc_1830`) — ambas com **Pr. Silas Malafaia** padrão (editável)
- **Exceção:** se 1º domingo for dia 01, Santa Ceia vai pro 2º domingo.
- **Culto da Vitória** (`seg_vit_2000`): dia original **segunda**; 🆕 **a partir de 06/10/2026, quinta**
- **Culto da Palavra** (`qua_palavra_2000`): dia original **quarta** (desde 17/05); 🆕 **a partir de 06/10/2026, terça**. Não acontece na quarta da Santa Ceia (3ª quarta) — regra mantida só enquanto ele estiver na quarta.
- **Encontro de Crescimento das Vitoriosas** (`qui_vitoriosas_2000`): dia original **quinta**; 🆕 **a partir de 06/10/2026, quarta**
- **Quartas antes de 17/05:** 20:00 EDP (`qua_edp_2000`, preso à quarta)
- **Santa Ceia mensal** (`qua_santaceia_2000`): 20:00, local **Advec Sobradinho**, pregador **Pr. Márcio Nascimento** (ambos editáveis por data). Acontece na **3ª ocorrência do dia em vigor** (sempre entre os dias 15 e 21). Dia original: **3ª quarta**; 🆕 **a partir de 06/10/2026, 3ª terça**. O Culto da Palavra **não acontece no dia dela**, onde quer que ela esteja.
- 🆕 **Seminário Teológico removido** (desde jun/2026).
- **Quinta após a Santa Ceia da 3ª quarta:** 🆕 20:00 **Santa Ceia Regional** (`qui_sc_regional_2000`, ministério `ADVEC_BSB`) — local **Advec Brasília**, pregador **Pr. Cesar Carreira** (editável). Helper `isThursdayAfterSantaCeia(date)`.

### 🆕 Mudança de dia dos cultos fixos (`config/recurring`)
- `MOVABLE_RECURRING` lista os 4 cultos que podem trocar de dia: Vitória, Palavra, Encontro de Crescimento e Santa Ceia mensal (esta com `monthly: true` = 3ª ocorrência do mês). Santa Ceia Regional e domingos ficam fixos.
- Regra = `{ id, templateId, weekday, from }`. `weekday` 0-6, ou `null` = culto **encerrado** a partir de `from`.
- Para cada data, vale a regra do template com o **maior `from` <= data** (`getRecurringRuleFor`). Sem regra → dia original.
- Helpers: `getRecurringRules()` (salvo ou `RECURRING_RULES_DEFAULTS`), `effectiveDowFor()`, `occursOnDate()`, `saveRecurringRules()`, `transferRecurringSchedule()` (copia a escala futura para o dia novo).
- Listener `onSnapshot(doc(fbDb,"config","recurring"))` com erro **silencioso** — sem as rules publicadas, usa os padrões do código.
- ⚠️ Como as regras só valem de `from` em diante, **o passado nunca muda**. Desfazer a regra restaura o comportamento anterior.

> 🆕 **Pregador padrão em recorrente:** a função `template(id, title, time, ministry, editableFields, opts)` aceita `opts.preacher` (e `opts.location`). O valor entra como base do template e é sobrescrito por override por data (merge `{...template, ...override}`), então continua editável.

### Editar evento fixo (descrição + horário) — `applyRecurringToFuture`
- Título/horário dos recorrentes são fixos no código, mas sobrescrevíveis via override por data.
- Descrição e horário editáveis (editor+admin) → grava override só na data. Só admin vê **"🔑 Aplicar nesta e nas próximas ocorrências"** (replica title+time para datas futuras do mesmo `templateId` até 31/dez do ano seguinte).

> ℹ️ `isThirdWeekdayOfMonth(date, dow)` / `thirdWeekdayOfMonth(y, m, dow)` generalizam o antigo `isThirdWednesday`. `dowPhrase(dow, monthly)` monta os textos ("toda terça" / "3ª terça do mês").

### Santa Ceia do domingo — cálculo (`getSantaCeiaDate` / `isSantaCeiaSunday`)
- 1º domingo do mês recebe Santa Ceia. Exceção: se o 1º domingo é dia 01, vai para o 2º domingo.

### Avisos do Mês — troca pela Santa Ceia (`getNoticeForWeek`)
- Trocam **no culto de Santa Ceia do 1º domingo**. A semana cujo domingo é a própria Santa Ceia (e anteriores) ainda exibe o mês anterior. `getNoticeForWeek(lastDayOfWeek)` compara com `getSantaCeiaDate(...)` usando `>` (estritamente depois).
- **Exemplo:** semana 01–07/06 (SC em 07/06) → avisos de **maio**; semana 08–14/06 → **junho**.

### Pessoas (Pregadores/Professores)
- Auto-pinados no topo: `Pr. Márcio Nascimento` e `Dc. Willian Borges` (`AUTO_PIN_NAMES`).
- Convidado(a) é atributo da PESSOA (`isPersonGuest()` consulta `state.people`).
- Visual: Local = chip navy texto branco; Convidado(a) = chip navy texto dourado italic + "· ✦ convidado(a)".
- 🆕 **Foto:** campo `photo` (base64). Avatar redondo na lista; botão de câmera (`setPersonPhoto`); clicar no avatar abre lightbox com ver/baixar/remover. Foto preservada no rename (troca de masterKey).

### Imagens (fotos e flyers) 🆕
- Guardadas como **base64 no próprio doc do Firestore** (sem Firebase Storage). Limite **1 MB/doc**.
- Resize/compressão no navegador: **fotos** lado máx. 512px (<250 KB); **flyers** lado máx. 1400px (<750 KB) — `fileToResizedDataUrl()`.
- Lightbox genérico `openImageLightbox(dataUrl, título, arquivo, onRemove?)` (z-index 60, acima dos modais).
- Flyer no mês: selo fixo `.pill-flyer-badge` (não some com título longo). Flyer no card: miniatura + ver/baixar. Flyer no modal: adicionar/ver/baixar/trocar/remover (`syncEventFormToState()` preserva o que foi digitado ao re-renderizar).

### Cabeçalho / identidade (`config/site`) 🆕
- `SITE_CONFIG_DEFAULTS` = { churchName: "ADVEC Sobradinho", year: "2026", yearScript: "Ano da", yearShout: "ORAÇÃO", verseTitle: "ORAI SEM CESSAR", verseRef: "1 TS. 5:17", leaderName: "Pr. Márcio Nascimento", viceName: "Dc. Willian Borges" }.
- `getSiteConfig()` = defaults + o que estiver salvo. `applyHeaderConfig()` aplica no topbar/banner/dirigentes/`<title>`. `saveSiteConfig(data)` grava (admin-only, merge).
- Listener `onSnapshot(doc(fbDb,"config","site"))` com erro **silencioso** (console only) — se as rules não estiverem publicadas, usa os defaults sem alarmar.
- IDs no HTML: `hdr-church-name`, `hdr-year`, `hdr-script`, `hdr-shout`, `hdr-verse-title`, `hdr-verse-ref`, `hdr-leader`, `hdr-vice`, `hdr-banner`.
- Também alimenta a **Programação Semanal (PDF)**: subtítulo, tema do ano e rodapé (igreja + dirigentes).

### Tipos de Eventos (`MINISTRIES` + `customMinistries`)
- **16 categorias fixas** com cores fixas (`MINISTRIES`) — só renomear o label (admin, via `ministryLabels`).
- **Custom (`customMinistries`)** — admin cria/renomeia/exclui (`CUSTOM_<SLUG>`, cores configuráveis). `getAllMinistries()` retorna fixos + custom.

```js
const MINISTRIES = {
  CULTOS:               { label: "Cultos (Celebração, Vitória, EDP, Palavra)", color: "#00BFFF", text: "#003E5C" },
  SEDE:                 { label: "Advec Sede (RJ)",                            color: "#FFD600", text: "#3D2F00" },
  SANTA_CEIA:           { label: "Santa Ceia / Batismo",                        color: "#1565C0", text: "#FFFFFF" },
  CONNECT:              { label: "Connect Jovens",                              color: "#0F0F0F", text: "#FFFFFF" },
  VITORIOSAS:           { label: "Vitoriosas",                                  color: "#E91E63", text: "#FFFFFF" },
  ADOLESCENTES:         { label: "Adolescentes",                                color: "#FF9800", text: "#3D1F00" },
  ADVEC_BSB:            { label: "Advec Regional (BSB)",                        color: "#E64A19", text: "#FFFFFF" },
  KIDS:                 { label: "Kids",                                        color: "#9FA8DA", text: "#1A237E" },
  HOMENS_VENCEDORES:    { label: "Homens Vencedores",                           color: "#9E9E9E", text: "#1A1A1A" },
  LOUVOR:               { label: "Louvor",                                      color: "#E53935", text: "#FFFFFF" },
  DIVERSOS:             { label: "Diversos (outros)",                           color: "#B71C1C", text: "#FFFFFF" },
  CONFERENCIA:          { label: "Conferências (Filiais)",                      color: "#43A047", text: "#FFFFFF" },
  FAMILIA:              { label: "Dep. Família",                                color: "#1B5E20", text: "#FFFFFF" },
  RECOMECO:             { label: "Recomeço",                                    color: "#FF7043", text: "#FFFFFF" },
  VENCEDORES_RENOVADOS: { label: "Vencedores Renovados",                        color: "#78909C", text: "#FFFFFF" },
  SOCIOAMBIENTAL:       { label: "Socioambiental",                              color: "#9575CD", text: "#FFFFFF" },
};
```

### Períodos Especiais (`specialPeriods`)
- `{ name, startDate, endDate, bgColor (default #C9A14A), textColor (default #0B3A5B) }`. Validação: início e fim, `endDate >= startDate`. `getSpecialPeriodsForDate(dateStr)` retorna os que contêm a data.

### Locais padrão (seed automático na 1ª execução)
- Advec Sobradinho, Advec Brasília, Advec Planaltina, Advec Samambaia, Advec Valparaíso, Advec Taguatinga, Outro local

---

## 🎨 FUNCIONALIDADES IMPLEMENTADAS

### Visualização
- Calendário mensal com cards arredondados; visão semanal (Seg→Dom, 7 cards)
- 🆕 Abre no **mês/semana atual** automaticamente
- Título do evento colorido com a cor do tipo de evento
- Pílula no mês: horário+título (trunca), pregador em chip navy, lição em italic (EDP), **selo de flyer** fixo no fim quando há flyer, 🆕 **observações** (só no computador)
- Header navy com logo ADVEC base64, dirigentes, banner anual — 🆕 **todos editáveis pelo admin** (config/site)
- Destaque visual de Períodos Especiais

### Filtros
- Tipos de evento (chips, vazio = todos), dias da semana, busca textual com ◀ ▶, painel expansível com badge

### Edição
- Modal respeita campos editáveis do recorrente; eventos manuais livres
- 🆕 **Flyer** no modal (adicionar/ver/baixar/trocar/remover); 🆕 **Foto** na aba Pregadores
- Cancelar/restaurar recorrentes (`/cancelled`); confirmação inline; auto-save com toast
- 2 checkboxes controlando o PDF: "Mostrar na Programação Semanal" (default on) e "Destacar como aviso" (default off)

### Configurações (⚙) — 10 abas
1. **Pregadores(as) e Professores(as)** — CRUD + Convidado(a) + fixar 📌 + 🆕 **foto** (editor+admin)
2. **Locais** — CRUD com rename propagado (editor+admin)
3. **Tipos de Eventos** — renomear fixos + criar/renomear/excluir custom + marcar "Sem pregador(a)" e 🆕 "Sem local" (admin)
4. **Períodos Especiais** — CRUD de faixas com cores (editor+admin)
5. **Avisos do Mês** — Leitura Bíblica, Motivos de Oração, Versículo por mês (editor+admin)
6. **Usuários** — role, remove acesso (admin)
7. **Cabeçalho** 🆕 — editar identidade (igreja, ano, tema, versículo, dirigentes) (admin)
8. **Cultos Fixos** — mudar o dia da semana / encerrar culto fixo, com data de vigência (admin)
9. **Eventos Recorrentes** 🆕 — criar evento que se repete (semanal, quinzenal, a cada 3/4 semanas, mensal) (admin)
10. **Backup** — exportar/restaurar (admin)

### Backup / Restauração
- **Exportar:** JSON com `BACKUP_COLLECTIONS` (events, overrides, cancelled, people, locations, ministryLabels, customMinistries, monthlyNotices, specialPeriods, **config** 🆕, users). Timestamps → ISO. 🆕 Tolerante: pula coleção sem permissão em vez de abortar.
- **Restaurar:** `BACKUP_RESTORABLE` (tudo acima **exceto `users`**, incluindo **config** 🆕).

### Programação Semanal (PDF)
- Botão "Programação" (modo Semana) abre janela com PDF (dourado/navy), logo do topbar
- 3 seções: Avisos para Todos os Cultos, Programação da Semana, Avisos para as Próximas Semanas
- 🆕 Subtítulo, tema do ano e rodapé (igreja + dirigentes) vêm da config do cabeçalho

### Auth
- Login (Google + Email/Senha + Cadastro), telas "Aguardando aprovação" / "Esqueci a senha", badge do usuário no topbar, restrições por role

---

## 🔄 WORKFLOWS

### Atualizar a agenda (deploy — GitHub Pages, pelo navegador)
1. Receber o `index.html` do Claude (já vem com o nome certo — não precisa renomear).
2. No repo `agendaoficial` → **Add file** → **Upload files** → arrastar por cima do antigo → **Commit changes**.
3. Aguardar ~1 min; **Ctrl+F5**.
4. **Se a versão mexer em Security Rules / coleções novas, publicar as rules no Firebase Console antes de testar.**

> 💡 Deploy no GitHub Pages **não consome créditos**. Alternativa: clicar no `index.html` → ✏️ Edit → colar → Commit (para arquivos grandes, o upload arrastando é mais seguro que colar).

### 🆕 Editar o cabeçalho (identidade da igreja) — pela tela, sem código
1. Logar como **admin** → ⚙ → aba **"Cabeçalho"**.
2. Editar nome da igreja, ano, subtítulo, tema, versículo+referência, dirigente e vice.
3. **Salvar cabeçalho** → reflete no topo, na aba do navegador e no PDF, para todos os usuários.
   - Pré-requisito: a Security Rule de `config` precisa estar publicada (ver Security Rules).

### 🆕 Mudar um culto fixo de dia da semana (ou encerrar)
1. Logar como **admin** → ⚙ → aba **"Cultos Fixos"**.
2. Conferir o bloco "Como está hoje" (dia atual de cada culto + mudanças já agendadas).
3. Em "Nova mudança": escolher o culto, "Passar para outro dia" ou "Encerrar", o novo dia e a data **a partir de** quando vale.
4. Deixar marcado **"Levar junto o que já está escalado"** para transportar pregador/lição/local das próximas ocorrências.
5. **Aplicar mudança** → confirma e reflete na hora para todos.
   - Semanas anteriores à data escolhida **não mudam**. Para voltar atrás, use o 🗑 na lista "Mudanças cadastradas".
   - Pré-requisito: a Security Rule de `config` precisa estar publicada (a mesma do cabeçalho — não há regra nova).

### 🆕 Criar um evento que se repete (ex.: quarta sim, quarta não)
1. Logar como **admin** → ⚙ → aba **"Eventos Recorrentes"**.
2. Preencher nome, tipo, **Repetir** (ex.: "A cada 2 semanas"), dia da semana, **Começa em** (a 1ª quarta do evento), horário e local.
3. Conferir a **prévia das próximas datas** → **Criar evento recorrente**.
4. Para parar: ✏️ → "Termina em" = último dia → Salvar.

### Aprovar novo usuário
1. Pessoa loga → "Aguardando aprovação". 2. Admin ⚙ → "Usuários" → muda role para Editor/Somente leitura. 3. Pessoa recarrega.

### Cadastrar avisos mensais
- ⚙ → "Avisos do Mês" → seleciona mês → preenche → Salvar. Lembrar da regra da Santa Ceia (cadastre no mês "real").

### Adicionar foto de pregador / flyer de evento 🆕
- **Foto:** ⚙ → Pregadores → botão de câmera na pessoa → escolher imagem (reduzida automaticamente). Clicar no avatar para ver/baixar/remover.
- **Flyer:** abrir o evento → campo "Flyer" → Adicionar. No card aparece miniatura; no mês, o selo.

### Cadastrar Período Especial / Tipo de Evento custom
- ⚙ → "Períodos Especiais" (nome, datas, cores) / "Tipos de Eventos" (nome + cores, admin).

### Fazer backup / restaurar
- ⚙ → "Backup" → "Exportar backup" / "Restaurar de um backup" (restaura tudo menos `users`).

### Imprimir Programação Semanal
- Modo Semana → ◀ ▶ → botão **🖨 Programação** → Imprimir / Salvar como PDF.

### Trocar o banner anual (ex: 2026 → 2027)
- 🆕 **Agora é pela tela:** ⚙ → "Cabeçalho" (admin) — editar ano, subtítulo, tema, versículo. Não precisa mexer no HTML nem pedir ao Claude.

### Mudar horário/descrição de um culto fixo (não o dia)
- Abrir o evento → editar descrição/horário → salvar. Só admin vê **"🔑 Aplicar nesta e nas próximas ocorrências"**.
- Para mudar o **dia da semana**, é a aba "Cultos Fixos" (acima).

### Usar a agenda em outra ADVEC (filial diferente)
1. 🆕 **Nome da igreja, ano/tema e dirigentes:** pela aba "Cabeçalho" (admin) — sem código.
2. Logo da igreja (base64 no topbar) — ainda no HTML.
3. Constantes JS: `ADMIN_EMAIL`, `AUTO_PIN_NAMES`.
4. Backend Firebase próprio (`firebaseConfig` + Security Rules, incluindo `config`).
5. Hosting: repositório GitHub separado + Pages.

---

## 🐛 BUGS / MELHORIAS PENDENTES

- [ ] **⚠️ Publicar Security Rule de `config`** no Console (necessária para salvar o cabeçalho)
- [ ] Confirmar `advecsobradinho.github.io` nos Authorized Domains do Firebase (login Google)
- [ ] Confirmar Security Rules de `customMinistries` e `specialPeriods`
- [ ] (Opcional) Cadastrar Pr. Silas Malafaia e Pr. Cesar Carreira na aba Pregadores
- [ ] (Opcional) Encurtar URL: renomear repo para `advecsobradinho.github.io`
- [ ] Testar fluxo completo em mobile (touch, scroll, modais, upload de imagem)
- [ ] Considerar PWA (instalável no celular)
- [ ] Domínio próprio (`agenda.advecsobradinho.com.br`)
- [ ] Email/Magic link como alternativa ao password
- [ ] Notificações push (eventos próximos)
- [ ] Histórico de auditoria (já gravamos `createdBy`/`updatedBy`; falta a tela)
- [ ] Exportar agenda **mensal** em PDF (a semanal já existe)
- [ ] Decidir se o Encontro de Crescimento acontece na quarta da Santa Ceia (hoje aparecem os dois; dá para cancelar o dia avulso)
- [x] ~~Criar evento fixo novo pela tela (semanal/quinzenal/mensal)~~ — **feito** via aba "Eventos Recorrentes" (config/series)
- [ ] (Opcional) Permitir que **editores** também criem eventos recorrentes (hoje só admin, pela regra de `config`)
- [ ] (Opcional) Evento recorrente em **vários dias da semana** de uma vez (hoje: um dia por evento)
- [x] ~~Mudar o dia da semana de um culto fixo sem mexer no HTML~~ — **feito** via aba "Cultos Fixos" (config/recurring)
- [x] ~~Refatorar identidade visual num bloco `CHURCH_CONFIG`~~ — **feito** via aba "Cabeçalho" (config/site)
- [x] ~~Abrir no mês/semana atual~~ — **feito**

---

## 📂 ARQUIVOS DO PROJETO

- `Agenda.html` / `index.html` — versão produção (entregue já como `index.html`). **No ar** em `https://advecsobradinho.github.io/agendaoficial/`. Inclui: fotos de pregador + flyers, selo de flyer no mês, cabeçalho editável, Santa Ceia Regional, pregadores padrão, mudança de dia dos cultos fixos, 🆕 eventos recorrentes criados pela tela. **NÃO contém Open Graph** (descartado).
- `CONTINUIDADE.md` — este documento
- `TUTORIAL_GitHub_Pages.md` — passo a passo de hospedagem (PT/EN)
- `QRCode_Agenda_cartao.png` / `QRCode_Agenda_com_logo.png` / `QRCode_Agenda_simples.png` — divulgação
- `preview.png` — imagem Open Graph (1200×630) criada mas **NÃO usada**

---

## 💡 COMO RETOMAR EM CONVERSA FUTURA

No início da próxima conversa, cole:

> "Estou continuando o projeto da agenda da igreja ADVEC Sobradinho. Anexei o CONTINUIDADE.md com todo o contexto e o Agenda.html atual. Quero [DESCREVER O QUE QUER FAZER]."

Anexe **os dois arquivos** juntos (o `CONTINUIDADE.md` e o `index.html`/`Agenda.html` atual).

---

## 🎯 COMANDOS ESPECIAIS COMBINADOS

- **"Volta o cabeçalho"** — desfazer `<header class="topbar hidden">` (tirar a classe `hidden`)
- **"Esconde o cabeçalho"** — colocar `<header class="topbar hidden">`
