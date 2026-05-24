# Styling & Interaction

Estilo via classes Tailwind interpretadas pela engine (`vendor/nativephp/mobile/src/Edge/TailwindParser.php`) — **não** é o Tailwind do navegador. Só os utilitários abaixo são reconhecidos.

## Classes Tailwind suportadas no render nativo

| Categoria | Classes | Exemplo |
|---|---|---|
| Spacing | `p- px- py- pt- pb- pl- pr- m- mx- my- mt- mb- ml- mr- gap-` | `<native:column class="p-4 gap-2">` |
| Cor de fundo | `bg-<cor>` | `class="bg-blue-600"` |
| Cor de texto | `text-<cor>` | `class="text-slate-900"` |
| Tipografia | `text-xs..text-3xl`, `font-thin..font-extrabold`, `text-center/left/right` | `class="text-lg font-bold text-center"` |
| Radius | `rounded`, `rounded-<sz>`, `rounded-full` | `class="rounded-2xl"` |
| Shadow | `shadow`, `shadow-<sz>` | `class="shadow-lg"` |
| Border | `border`, `border-<w>`, `border-<cor>` | `class="border border-slate-200"` |
| Opacidade | `opacity-<n>` | `class="opacity-50"` |
| Tamanho | `w- h- w-full h-full` | `class="w-32 h-32 w-full"` |
| Flex/alinhamento | `flex-1`, `justify-<x>`, `items-<x>`, `self-<x>` | `class="flex-1 justify-between items-center"` |
| Posição | `absolute`, `relative`, `top- bottom- left- right-` | `class="absolute top-2 right-2"` |

### Valores arbitrários
Use colchetes para valores fora da escala: `w-[100]`, `h-[100]`, `text-[100]` (tamanho de fonte), `bg-[#003399]`, `text-[#FFFFFF]`.

### Variantes (prefixos)
- `dark:` — modo escuro: `class="bg-white dark:bg-slate-900"`.
- `ios:` / `android:` — por plataforma: `class="ios:rounded-2xl android:rounded-lg"`.
- `glass:` — material Liquid Glass (iOS 26+): `class="glass:clear:interactive"`.

### Tokens de tema
`bg-theme-*`, `text-theme-*`, `border-theme-*` — cores que respeitam o tema do app (ex.: `text-theme-on-surface`, `bg-theme-surface`).

## O que NÃO funciona

| ❌ Não suportado | ✅ Alternativa |
|---|---|
| `grid`, `grid-cols-*` | use `<native:row>` / `<native:column>` aninhados |
| `fixed`, `sticky` | use `absolute`/`relative` + `<native:stack>` para sobreposição |
| `hover:`/`focus:` (estados de mouse) | eventos nativos (`@press`) |
| qualquer prefixo fora da lista acima | reescreva com utilitário suportado ou valor arbitrário |

## Interação

### `@press` — toque em ação
Liga um toque a um método público do componente. Funciona em qualquer elemento (`pressable`, `button`, `row`, `column`...).

```blade
{{-- blade --}}
<native:button @press="increment" label="+1"/>
<native:row @press="open('{{ $id }}')" class="p-4"> ... </native:row>
```

```php
// componente
public int $count = 0;
public function increment(): void { $this->count++; }   // re-render automático
```

Outros eventos: `@change` (select/checkbox), `@dismiss` (modal/bottom-sheet), `@submit` (formulário).

### `native:model` — binding bidirecional
Sincroniza um valor do controle com uma propriedade pública. **Não é `wire:model`.**

```blade
<native:toggle native:model="darkMode" label="Tema escuro"/>
<native:slider native:model.live="volume" :min="0" :max="100"/>
<native:outlined-text-input native:model.blur="email" placeholder="E-mail"/>
```

- `.live` — atualiza durante a interação (ex.: arrastar o slider).
- `.blur` — atualiza ao sair do campo.

```php
public bool $darkMode = false;
public int $volume = 30;
public string $email = '';
```

### `:prop` — passar valores PHP
Prefixe o atributo com `:` para passar uma expressão PHP em vez de string literal.

```blade
<native:badge :count="$unread"/>                 {{-- valor PHP --}}
<native:text :color="$muted">Texto</native:text>
<native:button label="Salvar"/>                  {{-- string literal, sem ':' --}}
```

## Estado (estilo Livewire)

Propriedades públicas guardam estado; métodos públicos são ações; o re-render acontece sozinho após a ação. Exemplo completo (contador):

```php
use Native\Mobile\Edge\NativeComponent;

class Counter extends NativeComponent
{
    public int $count = 0;

    public function increment(): void { $this->count++; }
    public function decrement(): void { $this->count--; }

    public function render(): \Illuminate\View\View
    {
        return view('native.counter');
    }
}
```

```blade
<native:column class="items-center justify-center gap-4">
    <native:text class="text-3xl font-bold">{{ $count }}</native:text>
    <native:row class="gap-8">
        <native:button @press="decrement" label="−"/>
        <native:button @press="increment" label="+"/>
    </native:row>
</native:column>
```
