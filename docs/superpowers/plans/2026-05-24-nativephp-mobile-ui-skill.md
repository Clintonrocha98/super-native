# Skill `nativephp-mobile-ui` — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Criar a skill de projeto `nativephp-mobile-ui` (`.claude/skills/`) que ensina o Claude a construir telas NativePHP Mobile do zero sem alucinar a API.

**Architecture:** TDD-for-skills (não TDD de código). O "teste" é um subagente cobaia: RED = cobaias SEM a skill produzem código e registramos as alucinações; GREEN = escrevemos `SKILL.md` + 3 arquivos de apoio a partir da fonte da verdade (`native-ui`/`mobile-air`) e das falhas do RED; REFACTOR = cobaias COM a skill repetem os cenários até acertarem. Uma skill umbrella + 3 arquivos de apoio.

**Tech Stack:** NativePHP Mobile (`nativephp/mobile` engine Edge + `nativephp/native-ui`), Laravel 13, Blade, Tailwind v4. Skill em Markdown.

---

## ⚠️ Regras de execução específicas deste plano

1. **As cobaias do RED NÃO podem ler arquivos deste repositório.** Este repo é um kitchen sink cheio de exemplos corretos; se a cobaia explorar, ela acerta copiando e o baseline fica falso. Os prompts instruem explicitamente: "projeto greenfield, não leia arquivos, use só seu conhecimento de NativePHP".
2. **As tarefas RED/REFACTOR despacham subagentes.** Se este plano for executado via subagent-driven-development, o subagente da tarefa precisa ter o tool `Agent`. Alternativamente (recomendado para RED/REFACTOR), execute essas tarefas **inline** pelo agente principal. Veja a seção "Execution Handoff".
3. **Fonte da verdade já instalada** em `vendor/nativephp/native-ui/` e `vendor/nativephp/mobile/` (symlinks para `../Plugins/nativephp/native-ui` e `../mobile-air`). Nunca inventar props/classes — sempre conferir nesses arquivos.
4. **Sem `Co-Authored-By` em commits** (regra do usuário).

---

## File Structure

```
.claude/skills/nativephp-mobile-ui/
├── SKILL.md                 # corpo enxuto (overview, core pattern, quick ref, common mistakes)
├── elements-reference.md    # catálogo completo: elementos base + componentes native-ui + props reais
├── styling-interaction.md   # classes Tailwind suportadas (de TailwindParser) + @press/estado
└── layouts-routing.md       # layouts + Route::native + ciclo de vida do NativeComponent

docs/superpowers/working/    # evidências do processo TDD (mantidas como prova)
├── nativephp-skill-baseline.md   # achados do RED
├── nativephp-skill-inventory.md  # extração da fonte da verdade
└── nativephp-skill-refactor.md   # achados do REFACTOR
```

---

## Task 1: Scaffold da estrutura

**Files:**
- Create: `.claude/skills/nativephp-mobile-ui/SKILL.md` (stub mínimo)
- Create: `docs/superpowers/working/.gitkeep`

- [ ] **Step 1: Criar diretórios**

Run:
```bash
mkdir -p .claude/skills/nativephp-mobile-ui docs/superpowers/working
touch docs/superpowers/working/.gitkeep
```

- [ ] **Step 2: Criar stub do SKILL.md (só frontmatter, corpo virá no GREEN)**

Escrever em `.claude/skills/nativephp-mobile-ui/SKILL.md`:

```markdown
---
name: nativephp-mobile-ui
description: Use when building mobile UI with NativePHP Mobile — writing native screens or components, blade elements like column/row/text/native:icon, Tailwind classes for native rendering, @press interactions, or Stack/Tabs layouts and Route::native screens.
---

# NativePHP Mobile UI

<!-- corpo preenchido na Task 4 (GREEN) -->
```

- [ ] **Step 3: Verificar frontmatter válido (≤1024 chars, name só com letras/números/hífens)**

Run:
```bash
head -5 .claude/skills/nativephp-mobile-ui/SKILL.md && wc -c .claude/skills/nativephp-mobile-ui/SKILL.md
```
Expected: frontmatter exibido; `name: nativephp-mobile-ui`.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/nativephp-mobile-ui/SKILL.md docs/superpowers/working/.gitkeep
git commit -m "chore: scaffold da skill nativephp-mobile-ui"
```

---

## Task 2: RED — Baseline sem a skill

**Objetivo:** Capturar, verbatim, como o Claude erra ao construir NativePHP sem a skill. Despacha 3 cobaias (subagentes frescos), uma por cenário do spec.

**Files:**
- Create: `docs/superpowers/working/nativephp-skill-baseline.md`

- [ ] **Step 1: Despachar cobaia do Cenário 1 (tela de perfil)**

Despachar subagente `general-purpose` com este prompt EXATO:

> Você está começando um projeto **NativePHP Mobile** novo e vazio. Não há exemplos no projeto para copiar e você NÃO deve ler nenhum arquivo do repositório — baseie-se apenas no seu conhecimento de NativePHP Mobile.
>
> Tarefa: crie o `NativeComponent` (classe PHP) e a view Blade para uma **tela de perfil de usuário** contendo: um avatar (imagem circular), o nome, uma bio curta e um botão "Editar perfil".
>
> Responda SOMENTE com o código que você escreveria (classe PHP + blade), sem explicações. Não escreva arquivos no disco.

Registrar a resposta integral.

- [ ] **Step 2: Despachar cobaia do Cenário 2 (lista de settings)**

Despachar subagente `general-purpose` com este prompt EXATO:

> Você está começando um projeto **NativePHP Mobile** novo e vazio. Não há exemplos no projeto para copiar e você NÃO deve ler nenhum arquivo do repositório — baseie-se apenas no seu conhecimento de NativePHP Mobile.
>
> Tarefa: crie o `NativeComponent` e a view Blade para uma **tela de configurações**: um card contendo uma lista de itens (Notificações, Privacidade, Tema escuro), onde "Tema escuro" tem um toggle on/off funcional.
>
> Responda SOMENTE com o código (classe PHP + blade), sem explicações. Não escreva arquivos no disco.

Registrar a resposta integral.

- [ ] **Step 3: Despachar cobaia do Cenário 3 (navegação)**

Despachar subagente `general-purpose` com este prompt EXATO:

> Você está começando um projeto **NativePHP Mobile** novo e vazio. Não há exemplos no projeto para copiar e você NÃO deve ler nenhum arquivo do repositório — baseie-se apenas no seu conhecimento de NativePHP Mobile.
>
> Tarefa: crie **duas telas** — uma lista de produtos e uma tela de detalhe que abre ao tocar num item, com botão de voltar — e configure as **rotas**. Inclua as classes `NativeComponent`, as views Blade e o trecho de `routes/web.php`.
>
> Responda SOMENTE com o código, sem explicações. Não escreva arquivos no disco.

Registrar a resposta integral.

- [ ] **Step 4: Analisar e consolidar achados**

Para cada cenário, comparar a resposta da cobaia contra a fonte da verdade e listar erros em categorias. Conferir contra:
- Tags built-in: `vendor/nativephp/mobile/src/Edge/NativeElementCollector.php` (linha ~48)
- Componentes e tags: `vendor/nativephp/native-ui/src/Components/*.php` (cada `elementType()`)
- Classe base/ciclo de vida: `vendor/nativephp/mobile/src/Edge/NativeComponent.php`
- Roteamento: `routes/web.php` deste repo (referência interna; usar só para conferir, a cobaia não viu)

Escrever `docs/superpowers/working/nativephp-skill-baseline.md` com esta estrutura:

```markdown
# RED Baseline — nativephp-mobile-ui

Data: 2026-05-24. Cobaias: 3 subagentes general-purpose, sem acesso ao repo.

## Cenário 1 — Tela de perfil
### Código produzido (resumo)
<colar trechos relevantes>
### Erros observados
- [ ] Usou tags HTML (`<div>`/`<img>`/`<button>`) em vez de `<column>`/`<image>`/`<button>` nativo? (sim/não + evidência)
- [ ] Classes Tailwind não suportadas pelo TailwindParser?
- [ ] Estrutura do NativeComponent incorreta (extends errado, sem render(), etc.)?
- [ ] Props inexistentes?

## Cenário 2 — Lista de settings
(mesma estrutura)

## Cenário 3 — Navegação
### Erros observados
- [ ] Usou `Route::get()`/`view()` em vez de `Route::native()`?
- [ ] Faltou layout (`StackLayout`) para back/push?
- [ ] Outros.

## Padrões recorrentes (alimentam "Common Mistakes" da skill)
1. ...
2. ...
```

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/working/nativephp-skill-baseline.md
git commit -m "test(skill): baseline RED da nativephp-mobile-ui"
```

---

## Task 3: Extração da fonte da verdade (inventário)

**Objetivo:** Catalogar elementos, componentes, props e classes Tailwind reais — a matéria-prima dos arquivos da skill. Nada de inventar; tudo lido do vendor.

**Files:**
- Create: `docs/superpowers/working/nativephp-skill-inventory.md`

- [ ] **Step 1: Listar elementos built-in e seus tipos**

Run:
```bash
sed -n '40,60p;520,560p' vendor/nativephp/mobile/src/Edge/NativeElementCollector.php
```
Registrar a lista de tags built-in (`column`, `row`, `stack`, `scroll_view`, `pressable`, `canvas`) e o mapeamento para classes.

- [ ] **Step 2: Catalogar os componentes do native-ui e seus tags**

Run:
```bash
for f in vendor/nativephp/native-ui/src/Components/*.php; do
  name=$(basename "$f" .php)
  type=$(grep -oE "return '[a-z_]+';" "$f" | head -1)
  echo "$name -> $type"
done
```
Registrar nome do componente → `elementType` (o tag usado no blade).

- [ ] **Step 3: Extrair os props/atributos de cada componente principal**

Ler os arquivos dos componentes campeões e anotar props públicos (construtor/propriedades):
```bash
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/Button.php
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/Card.php
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/ListItem.php
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/Toggle.php
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/Select.php
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/Modal.php
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/BottomSheet.php
sed -n '1,80p' vendor/nativephp/native-ui/src/Components/Icon.php
```
Para cada um, registrar: tag, props/atributos aceitos, e se usa slot.

- [ ] **Step 4: Extrair as categorias de classes Tailwind suportadas**

Run:
```bash
grep -nE "case '|str_starts_with|match\(|'bg-|'text-|'p-|'m-|'rounded|'shadow|'flex|'justify|'items|'w-|'h-|'gap" vendor/nativephp/mobile/src/Edge/TailwindParser.php | head -120
```
Registrar quais prefixos/utilitários o parser reconhece (spacing, cor, tipografia, radius, shadow, flex/justify/items, w/h, gap, opacity, border) e exemplos. Anotar utilitários comuns que **não** aparecem (= não suportados).

- [ ] **Step 5: Extrair API de layout/rota e ciclo de vida**

Run:
```bash
grep -nE "function native\(|function nativeGroup\(|function layout\(" vendor/nativephp/mobile/src/**/*.php 2>/dev/null | head
grep -nE "function navTitle|function navigationOptions|function showsNavBack|abstract|function render" vendor/nativephp/mobile/src/Edge/NativeComponent.php | head -20
ls app/NativeComponents/Layouts/
```
Registrar assinaturas de `Route::native`, `nativeGroup`, `->layout()`, e os métodos override do `NativeComponent` (`navTitle`, `navigationOptions`, `render`, etc.), e os 4 layouts disponíveis.

- [ ] **Step 6: Escrever o inventário**

Consolidar tudo em `docs/superpowers/working/nativephp-skill-inventory.md` com seções: Elementos base, Componentes (tabela tag→props→slot), Classes Tailwind suportadas/não-suportadas, API de layout/rota, Ciclo de vida do NativeComponent. Cada dado com o arquivo:linha de origem.

- [ ] **Step 7: Commit**

```bash
git add docs/superpowers/working/nativephp-skill-inventory.md
git commit -m "docs(skill): inventário da fonte da verdade nativephp-mobile-ui"
```

---

## Task 4: GREEN — Escrever `SKILL.md` (corpo)

**Files:**
- Modify: `.claude/skills/nativephp-mobile-ui/SKILL.md`

- [ ] **Step 1: Escrever o corpo completo**

Substituir o comentário-stub pelo corpo, usando dados do inventário (Task 3) e priorizando as falhas do baseline (Task 2). Estrutura obrigatória:

```markdown
## Overview
NativePHP Mobile renderiza UI **nativa** a partir de tags Blade próprias + classes Tailwind (layout via Yoga). Não é HTML, não é WebView. Princípio central: você escreve `<column>`/`<text>`/`<button>`, não `<div>`/`<p>`/`<button>` HTML.

## When to Use
- Criar telas/`NativeComponent`, escrever blade com tags nativas, estilizar com Tailwind no render nativo, navegação Stack/Tabs.
- NOT: lógica de backend pura, web Blade comum, plugins de device (câmera/GPS).

## Core Pattern (antes/depois)
<bloco ❌ HTML comum  vs  ✅ tags nativas — derivado do erro #1 do baseline>

## Quick Reference
| Preciso de... | Use | Notas |
|---|---|---|
| coluna/linha | `<column>` / `<row>` | flex; classes gap/justify/items |
| texto | `<text>` | nada de `<p>`/`<span>` |
| imagem | `<image>` / `<native:image>` | |
| ícone | `<native:icon>` | SF + Material |
| botão | `<button>` (native-ui) | label via slot ou prop |
| ... | ... | (linhas dos componentes campeões) |

## Common Mistakes
<lista derivada dos "Padrões recorrentes" do baseline — 1 linha cada: erro → correção>

## Reference files
- Catálogo completo de elementos e componentes: ver `elements-reference.md`
- Classes Tailwind suportadas e interação: ver `styling-interaction.md`
- Layouts, rotas e ciclo de vida: ver `layouts-routing.md`
```

Regras: corpo escaneável; props nas tabelas conferidos no inventário; ponteiros por nome de arquivo (sem `@`).

- [ ] **Step 2: Verificar tamanho e ausência de placeholders**

Run:
```bash
wc -w .claude/skills/nativephp-mobile-ui/SKILL.md
grep -niE "TODO|TBD|<colar|<bloco|<lista|<linhas|<derivado" .claude/skills/nativephp-mobile-ui/SKILL.md || echo "sem placeholders"
```
Expected: alvo <500 palavras; "sem placeholders".

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/nativephp-mobile-ui/SKILL.md
git commit -m "feat(skill): corpo do SKILL.md nativephp-mobile-ui"
```

---

## Task 5: GREEN — Escrever `elements-reference.md`

**Files:**
- Create: `.claude/skills/nativephp-mobile-ui/elements-reference.md`

- [ ] **Step 1: Escrever o catálogo completo**

Usar o inventário (Task 3). Estrutura:

```markdown
# Elements Reference

## Elementos base (engine Edge)
### column / row
<o que faz, atributos de classe aceitos, 1 exemplo ótimo>
### stack / scroll_view / pressable / canvas
<idem, 1 exemplo cada>
### text / image / native:icon
<idem>

## Componentes (native-ui)
Para cada: tag, props (do inventário), slot (sim/não), 1 exemplo.
- Button, Card, ListItem, Modal, BottomSheet, Select, Toggle, Checkbox,
  Radio, Slider, Badge, Chip, Carousel, ProgressBar, ActivityIndicator,
  FilledTextInput / OutlinedTextInput / BareTextInput
```

Cada exemplo deve ser blade real e válido (tags/props conferidos no vendor). Um exemplo excelente por item, não vários medíocres.

- [ ] **Step 2: Verificar props contra a fonte (amostragem)**

Run:
```bash
grep -oE "return '[a-z_]+';" vendor/nativephp/native-ui/src/Components/Button.php
grep -niE "TODO|TBD|<o que faz|<idem|<props" .claude/skills/nativephp-mobile-ui/elements-reference.md || echo "sem placeholders"
```
Expected: tag de Button confere com o doc; "sem placeholders".

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/nativephp-mobile-ui/elements-reference.md
git commit -m "feat(skill): elements-reference da nativephp-mobile-ui"
```

---

## Task 6: GREEN — Escrever `styling-interaction.md`

**Files:**
- Create: `.claude/skills/nativephp-mobile-ui/styling-interaction.md`

- [ ] **Step 1: Escrever estilo + interação**

Usar a extração do `TailwindParser` (Task 3, Step 4). Estrutura:

```markdown
# Styling & Interaction

## Classes Tailwind suportadas no render nativo
| Categoria | Classes suportadas | Exemplo |
|---|---|---|
| Spacing | p-*, px-*, py-*, m-*, gap-* | `<column class="p-4 gap-2">` |
| Cor | bg-*, text-<cor> | ... |
| Tipografia | text-xs..3xl, font-thin..extrabold, text-center/left/right | ... |
| Radius | rounded, rounded-* | ... |
| Shadow | shadow, shadow-* | ... |
| Flex | flex-1, justify-*, items-* | ... |
| Tamanho | w-*, h-*, w-full, h-full | ... |
| Opacidade/Border | opacity-*, border, border-* | ... |

## O que NÃO funciona
<utilitários comuns que o parser não reconhece + alternativa>

## Interação: @press
<como ligar toque a método do componente — 1 exemplo>

## Estado (estilo Livewire)
<propriedade pública + método de ação + re-render — 1 exemplo (ex.: counter)>
```

Conferir cada categoria no `TailwindParser.php`. Listar em "NÃO funciona" apenas o que confirmou ausente.

- [ ] **Step 2: Verificar ausência de placeholders**

Run:
```bash
grep -niE "TODO|TBD|<utilitários|<como|<propriedade|\.\.\.$" .claude/skills/nativephp-mobile-ui/styling-interaction.md || echo "sem placeholders"
```
Expected: "sem placeholders".

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/nativephp-mobile-ui/styling-interaction.md
git commit -m "feat(skill): styling-interaction da nativephp-mobile-ui"
```

---

## Task 7: GREEN — Escrever `layouts-routing.md`

**Files:**
- Create: `.claude/skills/nativephp-mobile-ui/layouts-routing.md`

- [ ] **Step 1: Escrever layouts + rotas + ciclo de vida**

Usar Task 3 Step 5 + `app/NativeComponents/Layouts/` + `routes/web.php`. Estrutura:

```markdown
# Layouts, Routing & Lifecycle

## NativeComponent (ciclo de vida)
<extends Native\Mobile\Edge\NativeComponent; estado público; métodos de ação;
 render() → view(); navTitle()/navigationOptions()/showsNavBack() — 1 exemplo mínimo>

## Layouts disponíveis
| Layout | Quando usar |
|---|---|
| StackLayout | telas com back-chevron (push/detail) |
| TabsLayout | abas custom |
| NativeStackLayout | NavigationStack nativo (top bar, Liquid Glass iOS 26+) |
| NativeTabsLayout | TabView nativo (bottom bar) |

## Roteamento
<Route::native('/x', Component::class)->name(...);
 Route::nativeGroup(Layout::class, fn() => ...);
 ->layout(StackLayout::class);  — exemplos reais>

## Receita: lista + detalhe com push/back
<exemplo completo de 2 rotas + 2 components, derivado do erro do Cenário 3 do baseline>
```

Conferir assinaturas no vendor; o exemplo de navegação deve corrigir exatamente o que a cobaia do Cenário 3 errou.

- [ ] **Step 2: Verificar ausência de placeholders**

Run:
```bash
grep -niE "TODO|TBD|<extends|<Route|<exemplo" .claude/skills/nativephp-mobile-ui/layouts-routing.md || echo "sem placeholders"
```
Expected: "sem placeholders".

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/nativephp-mobile-ui/layouts-routing.md
git commit -m "feat(skill): layouts-routing da nativephp-mobile-ui"
```

---

## Task 8: REFACTOR — Re-teste com a skill

**Objetivo:** Provar que a skill resolve as falhas do RED. Despacha 3 cobaias frescas COM o conteúdo da skill, mesmos cenários.

**Files:**
- Create: `docs/superpowers/working/nativephp-skill-refactor.md`

- [ ] **Step 1: Montar o pacote da skill para injeção**

Run:
```bash
cat .claude/skills/nativephp-mobile-ui/SKILL.md .claude/skills/nativephp-mobile-ui/*.md
```
Guardar o conteúdo concatenado para colar nos prompts das cobaias (simula a skill carregada).

- [ ] **Step 2: Re-despachar Cenário 1 COM a skill**

Despachar subagente `general-purpose` com: o conteúdo da skill (do Step 1) + "Projeto NativePHP novo, não leia arquivos do repo. Usando APENAS a referência acima, " + a tarefa do Cenário 1 (perfil). Registrar resposta.

- [ ] **Step 3: Re-despachar Cenário 2 COM a skill**

Igual ao Step 2, com a tarefa do Cenário 2 (settings list).

- [ ] **Step 4: Re-despachar Cenário 3 COM a skill**

Igual ao Step 2, com a tarefa do Cenário 3 (navegação).

- [ ] **Step 5: Avaliar pass/fail e registrar**

Escrever `docs/superpowers/working/nativephp-skill-refactor.md`: para cada cenário, marcar se os erros do baseline sumiram (tags nativas corretas, props reais, navegação correta) e listar quaisquer NOVAS lacunas/alucinações.

Critério de PASS: nos 3 cenários, tags nativas corretas + props existentes (conferidas no vendor) + classes suportadas + navegação via `Route::native`/layout correta.

- [ ] **Step 6: Commit**

```bash
git add docs/superpowers/working/nativephp-skill-refactor.md
git commit -m "test(skill): refactor re-teste da nativephp-mobile-ui"
```

---

## Task 9: REFACTOR — Fechar lacunas

**Files:**
- Modify: arquivos da skill conforme as lacunas da Task 8

- [ ] **Step 1: Aplicar correções**

Para cada nova lacuna registrada na Task 8: ajustar o arquivo da skill correspondente (esclarecer, adicionar exemplo, ou acrescentar linha em "Common Mistakes" do `SKILL.md`). Se nenhuma lacuna: registrar "nenhuma correção necessária" e pular para Step 3.

- [ ] **Step 2: Re-despachar o(s) cenário(s) que falhou(aram)**

Repetir o dispatch da Task 8 apenas para os cenários com lacuna, com a skill atualizada. Confirmar PASS. (Loop até passar.)

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/nativephp-mobile-ui/
git commit -m "fix(skill): fechar lacunas do refactor nativephp-mobile-ui"
```

---

## Task 10: Verificação final

- [ ] **Step 1: Checklist de qualidade da writing-skills**

Conferir manualmente:
- `name` só letras/números/hífens; `description` começa com "Use when", terceira pessoa, sem resumo de workflow.
- `SKILL.md` escaneável (<500 palavras no corpo); referência pesada nos 3 arquivos.
- Cross-references por nome (sem `@`).
- Nenhuma prop/classe/tag inventada (amostragem final contra o vendor).

Run:
```bash
wc -w .claude/skills/nativephp-mobile-ui/*.md
grep -rniE "TODO|TBD" .claude/skills/nativephp-mobile-ui/ || echo "sem TODOs"
grep -nE "@skills/|@\.claude" .claude/skills/nativephp-mobile-ui/*.md || echo "sem @-links"
```

- [ ] **Step 2: Smoke test de descoberta (opcional, recomendado)**

Despachar uma cobaia fresca com uma tarefa NOVA não testada (ex.: "tela de chat com bolhas e input") + a skill, e confirmar que ela aplica corretamente. Registrar resultado no fim do `nativephp-skill-refactor.md`.

- [ ] **Step 3: Commit final**

```bash
git add -A .claude/skills/nativephp-mobile-ui/ docs/superpowers/
git commit -m "chore(skill): verificação final nativephp-mobile-ui"
```

---

## Self-Review (preenchido pelo autor do plano)

- **Cobertura do spec:** Propósito (fundamentos) → Tasks 4-7. Granularidade 1 skill + 3 apoios → File Structure + Tasks 4-7. Profundidade curado+completo → Task 4 (curado) + Tasks 5-7 (completo). Fonte da verdade native-ui → Task 3. Validação TDD → Tasks 2 (RED), 8-9 (REFACTOR). Cenários BDD do spec → prompts das Tasks 2 e 8. Fora de escopo (receitas/plugins/build/ícones) → não há tarefas para eles. ✅
- **Placeholders:** os `<...>` nos blocos de estrutura são *gabaritos de seção*, não placeholders de execução — cada um vem acompanhado da fonte exata (inventário/baseline) de onde o conteúdo é derivado, e os steps de verificação fazem `grep` justamente por esses marcadores para garantir que foram substituídos. ✅
- **Consistência de nomes:** `nativephp-mobile-ui`, 3 arquivos (`elements-reference.md`, `styling-interaction.md`, `layouts-routing.md`), working docs com nomes estáveis em todas as tasks. ✅
