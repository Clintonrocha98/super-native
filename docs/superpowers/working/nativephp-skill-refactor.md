# REFACTOR — Re-teste com a skill

Data: 2026-05-24. 3 cobaias `general-purpose` frescas, instruídas a ler SOMENTE os 4 arquivos da skill (não os exemplos do repo) e refazer os cenários do RED.

## Resultado por cenário (vs. erros do baseline)

### Cenário 1 — Perfil → **PASS**
- ✅ `use Native\Mobile\Edge\NativeComponent`
- ✅ `render(): \Illuminate\View\View` + `view()`
- ✅ `<native:column>`/`<native:image>`/`<native:text>`/`<native:button>` (sem HTML)
- ✅ estado via `mount()`, navegação `navigate('/path')`
- ✅ classes Tailwind suportadas, `StackLayout` na rota

### Cenário 2 — Settings → **PASS**
- ✅ namespace/render corretos
- ✅ `<native:card>` + `<native:divider/>` (não inventou `<view>`)
- ✅ `<native:toggle native:model="darkMode">` (componente real + binding two-way)
- ✅ ícones SF (`bell.fill`, `lock.fill`, `moon.fill`, `chevron.right`) — não Lucide
- ✅ `dark:` usado (suportado)

### Cenário 3 — Navegação → **PASS (com 1 lacuna menor)**
- ✅ namespace/render corretos
- ✅ `{id}` via `mount()` lendo `$this->params['id']`
- ✅ `Route::native('/products', Class)` COM path + `->layout(StackLayout::class)`
- ✅ `navigate('/products/{{ $id }}')` (string de path)
- ✅ criou o `StackLayout` corretamente; `showsNavBack()` na tela raiz
- ⚠️ **Lacuna:** ficou inseguro sobre QUAL facade fornece `Route::native` e deixou um import órfão `use Native\Mobile\Facades\Route as NativeRoute;` com comentário de incerteza.

## Veredito

Todos os 7 padrões de erro do baseline foram eliminados. **Skill aprovada.**

## Lacuna a fechar (Task 9)

`layouts-routing.md` deve deixar explícito: `Route::native` / `Route::nativeGroup` são **macros na facade padrão** `Illuminate\Support\Facades\Route` (confirmado em `routes/web.php:67`). Não há facade própria — use o `Route` normal.

**Fechada (Task 9):** adicionada a nota na seção Roteamento + `use Illuminate\Support\Facades\Route;` nos exemplos. Re-teste do Cenário 3 confirmou: a cobaia passou a usar a facade padrão sem import órfão nem hesitação.

## Smoke test (Task 10) — tarefa NOVA: tela de chat → **PASS**

Cobaia fresca, tarefa não testada (lista de bolhas + input + enviar). Resultado: namespace Edge, `render(): View`, tags nativas, `native:model="draft"`, estado estilo Livewire (`messages`/`sendMessage()`), `justify-end/start` (sabe que não há `grid`), valor arbitrário `max-w-[280]`, SF Symbol `paperplane.fill`, `<native:divider/>`, `Route::native` + `StackLayout` na facade padrão. Reconheceu honestamente um limite não documentado (scroll-to-bottom). A skill generaliza para cenários inéditos.
