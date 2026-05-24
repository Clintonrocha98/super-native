# RED Baseline — nativephp-mobile-ui

Data: 2026-05-24. Cobaias: 3 subagentes `general-purpose` frescos, instruídos a NÃO ler o repo nem usar skills (simulando projeto greenfield). Cada resposta foi comparada com a fonte da verdade em `vendor/nativephp/{mobile,native-ui}` e nos exemplos do repo.

## Fonte da verdade (referência das correções)

| Tema | Correto (confirmado no vendor/exemplos) |
|---|---|
| Classe base | `Native\Mobile\Edge\NativeComponent` |
| render() | `public function render(): \Illuminate\View\View { return view('...'); }` |
| Param de rota | `public function mount(): void { $this->id = (int) ($this->params['id'] ?? 0); }` |
| Tags built-in | `column`, `row`, `stack`, `scroll_view`, `pressable`, `canvas` (`NativeElementCollector.php:48`) |
| Imagem | `<native:image ...>` (15 usos nos exemplos) |
| Ícone | `<native:icon :sf="App\Icons\SF::X" :material="App\Icons\Material::Y" :size="24"/>` ou `name="<sf-symbol>"` (ex. `minus.circle.fill`) |
| Componentes native-ui | tags reais: `toggle`, `card`, `list_item`, `select`, `button`, etc. |
| Rota | `Route::native('/path', Component::class)->name('...')` |
| Navegação | `@press="navigate('/path/{{ $id }}')"` (string de path) |
| Back/push | via layout `StackLayout` (`->layout(StackLayout::class)` ou `nativeGroup`) |

## Cenário 1 — Tela de perfil

Código (resumo): `extends Native\Mobile\Components\NativeComponent`; construtor com props default; `render(): string` via `$this->view()`; blade com `<column>/<row>/<text>` (ok), mas `<image src="...">` e `<button><text>...</text></button>`.

### Erros observados
- [x] **Namespace errado**: `Native\Mobile\Components\NativeComponent` → correto `...\Edge\NativeComponent`.
- [x] **render() errado**: tipo de retorno `string` + `$this->view()` → correto `: \Illuminate\View\View` + `view()`.
- [x] **Imagem**: `<image src=>` → convenção é `<native:image>`.
- [x] **Estado via construtor**: dados injetados no `__construct` → o padrão é propriedade pública + `mount()`.
- [~] **Botão**: `<button><text>Editar perfil</text></button>` funciona por slot→label, mas idiomático é `<native:button>` / prop `label`.
- [x] **Classes tailwind**: usou `text-gray-900/500` — paleta ok, mas convém confirmar tokens.

## Cenário 2 — Lista de settings

Código (resumo): `extends Native\Mobile\Components\NativeComponent`; `mount()` lendo `config()`; toggle/divisória reimplementados na mão; `<native:icon name="bell|lock|moon|chevron-right">`; classes `dark:*` por todo lado.

### Erros observados
- [x] **Namespace errado** (igual cenário 1).
- [x] **render() errado** (igual cenário 1).
- [x] **Tag `<view>` inexistente**: usada para divisória e para o toggle → não é built-in nem componente. Usar `<column>`/`<row>` ou componente real.
- [x] **Toggle reimplementado**: existe `toggle` (componente native-ui) → não construir na mão com `<view>`.
- [x] **Card/lista reimplementados**: existem `card` e `list_item` → usar os componentes.
- [x] **Ícones estilo Lucide**: `name="bell|moon|lock|chevron-right"` → nomes reais são SF Symbols (`minus.circle.fill`) ou via `:sf=`/`:material=`. A prop `name` existe, mas esses valores não.
- [~] **`dark:` variantes**: uso extensivo; o parser tem suporte parcial (4 refs em `TailwindParser.php`) — confirmar cobertura na escrita (Task 6).

## Cenário 3 — Navegação (lista + detalhe)

Código (resumo): `use Native\Mobile\Component as NativeComponent`; `render(): string` via `view()`; detalhe com `mount(int $id)`; navegação `@press="$navigate('product.show', { id: ... })"` e `@press="$back()"`; rotas `Route::native(ProductList::class)` sem path; import confuso de facade.

### Erros observados
- [x] **Namespace errado**: `Native\Mobile\Component` → correto `...\Edge\NativeComponent`.
- [x] **render() errado** (igual aos demais).
- [x] **Param de rota via método**: `mount(int $id)` → correto `mount(): void` lendo `$this->params['id']`.
- [x] **`Route::native(Class)` sem path**: → `Route::native('/products', ProductList::class)`.
- [x] **Navegação inventada**: `$navigate('rota', { id })` (nome+objeto JS) → correto `navigate('/products/{{ $id }}')` (string de path).
- [x] **`$back()` inventado**: → back-chevron vem do `StackLayout` (não há helper `$back()`).
- [x] **Sem layout**: nenhuma rota dentro de `nativeGroup`/`->layout()` → sem chrome de navegação/back.
- [x] **Import de facade redundante**: `Native\Mobile\Facades\Route as NativeRoute` não usado.

## Padrões recorrentes → alimentam "Common Mistakes" da skill

1. **Namespace da classe base**: sempre erram. É `Native\Mobile\Edge\NativeComponent` (engine "Edge").
2. **render()**: retornam `string`; o correto retorna `\Illuminate\View\View` via `view()`.
3. **Parâmetro de rota**: tentam construtor ou `mount($arg)`; o correto é `mount(): void` + `$this->params['id']`.
4. **Inventar tag `<view>`** para divisórias/containers; usar `<column>`/`<row>`. Imagem é `<native:image>`.
5. **Ícones**: usam nomes Lucide; o correto são SF Symbols (`name="..."`) ou `:sf=`/`:material=` com `App\Icons\*` e `:size`.
6. **Reimplementar componentes na mão** (toggle/card/list); existem `toggle`, `card`, `list_item`, `select`, etc.
7. **Navegação**: inventam `$navigate(rota, {obj})`/`$back()` e `Route::native(Class)`; o correto é `Route::native('/path', Class)`, `navigate('/path')` e back via `StackLayout`.
