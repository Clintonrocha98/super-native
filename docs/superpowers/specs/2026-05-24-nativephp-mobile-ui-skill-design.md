# Design — Skill `nativephp-mobile-ui`

> Data: 2026-05-24
> Status: aprovado no brainstorming, aguardando review do spec

## 1. Objetivo

Criar uma skill de projeto (em `.claude/skills/`) que ensine o Claude a **construir telas NativePHP Mobile do zero sem alucinar a API**. O foco são os **fundamentos do sistema**: o catálogo de elementos/componentes, as classes Tailwind que de fato funcionam no render nativo, interação (`@press`/estado) e a estrutura de app (layouts + roteamento).

Este repositório (`super-native`) é um *kitchen sink* com ~60 telas reais e mini-apps (Twitter, IKEA, Spotify, etc.). Ele serve como acervo de exemplos. A skill destila esses fundamentos para que projetos NativePHP **futuros** sejam construídos corretamente.

### Por que uma skill (e não CLAUDE.md)

Padrões NativePHP **não são específicos deste projeto** — aplicam-se a qualquer app NativePHP. Logo, são reutilizáveis e merecem uma skill, conforme a `writing-skills`. O que é específico do repo continua no CLAUDE.md.

## 2. Decisões do brainstorming

| Tema | Decisão |
|---|---|
| Propósito #1 | Fundamentos do sistema (construir tela do zero sem alucinar) |
| Granularidade | **Uma** skill umbrella + arquivos de apoio |
| Profundidade | Curado no `SKILL.md` (80/20) + completo nos arquivos de apoio |
| Fonte da verdade | Pacote `native-ui` + exemplos do repo (instalados ✅) |
| Validação | Ciclo TDD completo (RED → GREEN → REFACTOR) |
| Nº de arquivos de apoio | 3 (elements / styling-interaction / layouts-routing) |
| Nome | `nativephp-mobile-ui` |

## 3. Pré-requisito resolvido: instalação das dependências

O `composer.json` referencia dois pacotes via *path repository* local que não vieram no fork:

- `nativephp/mobile` (`dev-element`) → `../mobile-air`
- `nativephp/native-ui` (`dev-main`) → `../Plugins/nativephp/native-ui`

Ambos foram clonados dos repositórios públicos (`NativePHP/mobile-air` branch `element`; `nativephp/native-ui` branch `main`) nos paths esperados, e `composer install` rodou com sucesso. A fonte da verdade está disponível em:

- `vendor/nativephp/native-ui/src/Elements/` e `src/Components/`
- `vendor/nativephp/mobile/src/Edge/` (engine que registra os tags base `<column>/<row>/<text>`)

## 4. Estrutura da skill

```
.claude/skills/nativephp-mobile-ui/
├── SKILL.md               # corpo enxuto e escaneável (80/20)
├── elements-reference.md  # catálogo completo de elementos + componentes
├── styling-interaction.md # classes Tailwind suportadas + @press/estado
└── layouts-routing.md     # layouts + Route::native + ciclo de vida
```

### 4.1 `SKILL.md` (corpo)

- **Overview + princípio central**: UI nativa renderizada de tags Blade próprias (não HTML, não WebView).
- **When to use / when NOT**: gatilhos e quando não se aplica.
- **Core Pattern**: comparação antes/depois (HTML comum ❌ vs `<column>`/`<text>` ✅).
- **Quick Reference**: tabela dos elementos mais usados (layout + conteúdo + componentes campeões).
- **Common Mistakes**: preenchido a partir do que o baseline RED revelar.
- **Ponteiros** para os 3 arquivos de apoio (sem `@`, só nome — para não forçar load).

### 4.2 `elements-reference.md`

Catálogo completo, com props **reais** extraídas de `vendor/nativephp/native-ui/src/{Elements,Components}` e 1 exemplo ótimo por item:

- Layout: `column`, `row`, `stack`, scroll, `spacer`.
- Conteúdo: `text`, `image`, `native:icon`.
- Componentes: `Button`, `Card`, `ListItem`, `Modal`, `BottomSheet`, `Select`, `Toggle`, `Checkbox`, `Radio`, `Slider`, `Badge`, `Chip`, `Carousel`, `ProgressBar`, `ActivityIndicator`, text inputs (`Filled`/`Outlined`/`Bare`).

### 4.3 `styling-interaction.md`

- Classes Tailwind que funcionam no render nativo: spacing, cores, tipografia (size/weight/align), radius, shadow, flex/justify/items, opacity, borders.
- O que **não** funciona (e por quê) — armadilha comum.
- Interação: `@press` e binding de estado estilo Livewire (estado público + métodos de ação).

### 4.4 `layouts-routing.md`

- Layouts: `StackLayout`, `TabsLayout`, `NativeStackLayout`, `NativeTabsLayout` — quando usar cada.
- Roteamento: `Route::native`, `nativeGroup()`, `->layout()`, `navTitle()`, `NavBarOptions`.
- Ciclo de vida do `NativeComponent`: estado, métodos de ação, `render()`.

## 5. Frontmatter

```yaml
---
name: nativephp-mobile-ui
description: Use when building mobile UI with NativePHP Mobile — writing
  native screens or components, blade elements like column/row/text/native:icon,
  Tailwind classes for native rendering, @press interactions, or Stack/Tabs
  layouts and Route::native screens.
---
```

A `description` descreve **apenas o quando usar** (gatilhos + keywords), nunca resume o workflow — regra crítica da `writing-skills` para o Claude não pular o corpo da skill.

## 6. Processo de criação (TDD para skills)

A `writing-skills` exige: **nenhuma skill sem um teste que falha primeiro.** Sendo skill de referência, o teste é *gap testing* com subagente.

```
[RED] baseline SEM skill   →   [GREEN] escrever skill   →   [REFACTOR] re-testar
 subagente executa             mirando as falhas             com a skill até
 cenários e registramos        reais observadas             passar; falhas
 onde alucina                                               viram Common Mistakes
```

### Comportamento esperado (BDD dos cenários de teste)

**Cenário 1 — Tela de perfil (layout + conteúdo + botão)**
- Given um subagente sem a skill recebe "crie uma tela NativePHP de perfil com avatar, nome, e um botão de editar"
- Then registramos os erros (ex.: uso de `<div>`/`<img>`/`<button>` HTML, classes não suportadas)
- Given a skill aplicada
- Then o subagente usa `<column>/<image>/<text>` e `Button` com props corretas

**Cenário 2 — Lista de settings (componentes)**
- Given "crie uma lista de configurações com itens e toggles, dentro de um card"
- Then sem a skill: props inventadas em `ListItem`/`Toggle`; com a skill: props reais do `native-ui`

**Cenário 3 — Navegação (estrutura de app)**
- Given "crie duas telas: uma lista e um detalhe que abre ao tocar, com voltar"
- Then sem a skill: estrutura de layout/rota incorreta; com a skill: `Route::native` + `StackLayout` + push/back corretos

### Critério de sucesso

O subagente, **com a skill**, produz blade NativePHP válido (tags nativas corretas, props existentes, classes suportadas, navegação correta) nos 3 cenários, sem alucinar API.

## 7. Fora de escopo (YAGNI)

- Receitas dos mini-apps (Twitter/IKEA/etc.) — fica para uma possível 2ª skill futura, se surgir gatilho distinto.
- Plugins de device/dialog (câmera, geolocalização, etc.).
- Build/deploy nativo (`native:run`, Android Studio).
- Catálogo exaustivo de ícones SF/Material (referenciar `app/Icons`, não duplicar).

## 8. Verificação / qualidade

- Props e nomes de componentes conferidos contra `vendor/nativephp/native-ui`.
- `SKILL.md` enxuto e escaneável; referência pesada nos arquivos de apoio.
- Rationalization/Common Mistakes derivados do baseline real, não imaginados.
