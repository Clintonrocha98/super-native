---
name: nativephp-mobile-ui
description: Use when building mobile UI with NativePHP Mobile — writing native screens or components, blade elements like column/row/text/native:icon, Tailwind classes for native rendering, @press interactions, or Stack/Tabs layouts and Route::native screens.
---

# NativePHP Mobile UI

## Overview

NativePHP Mobile renderiza UI **nativa** (SwiftUI/Compose, layout via Yoga) a partir de tags Blade próprias + classes Tailwind. **Não é HTML, não é WebView.**

**Princípio central:** prefixe tudo com `native:`. Você escreve `<native:column>` / `<native:text>` / `<native:button>`, nunca `<div>` / `<p>` / `<button>` HTML. Não existe `<view>`.

## When to Use

- Criar telas (`NativeComponent`), escrever blade com tags nativas, estilizar com Tailwind no render nativo, ligar toque a ações (`@press`), navegação Stack/Tabs.
- **NOT:** lógica de backend pura, Blade web comum, plugins de device (câmera/GPS), build/deploy nativo.

## Core Pattern (antes/depois)

```blade
{{-- ❌ HTML — não renderiza nada de nativo --}}
<div class="flex flex-col">
    <img src="{{ $avatar }}" class="rounded-full">
    <p>{{ $name }}</p>
    <button onclick="edit()">Editar</button>
</div>

{{-- ✅ Tags nativas com prefixo native: --}}
<native:column class="items-center gap-4 p-6">
    <native:image src="{{ $avatar }}" class="w-32 h-32 rounded-full"/>
    <native:text class="text-2xl font-bold">{{ $name }}</native:text>
    <native:button @press="edit" label="Editar"/>
</native:column>
```

```php
// ❌ namespace e retorno errados
use Native\Mobile\Components\NativeComponent;        // não existe
public function render(): string { return $this->view('x'); }

// ✅ engine "Edge" + retorna View
use Native\Mobile\Edge\NativeComponent;
public function render(): \Illuminate\View\View { return view('x'); }
```

## Quick Reference

| Preciso de... | Use | Notas |
|---|---|---|
| coluna / linha | `<native:column>` / `<native:row>` | flex; classes `gap- justify- items-` |
| sobreposição | `<native:stack>` | badges, overlays |
| rolagem | `<native:scroll_view>` | |
| texto | `<native:text>` | nunca `<p>`/`<span>` |
| imagem | `<native:image src="">` | nunca `<img>` |
| ícone | `<native:icon :sf=... :material=... :size>` | SF Symbols; **não** nomes Lucide |
| botão | `<native:button @press="m" label="">` | ou texto via slot |
| toggle / slider | `<native:toggle native:model="p">` | binding two-way `native:model` |
| card / chip / badge | `<native:card>` / `<native:chip>` / `<native:badge>` | |
| lista | `<native:list-item :headline :supporting>` | |
| modal / sheet | `<native:modal :visible>` / `<native:bottom-sheet :visible>` | `@dismiss` |
| navegar | `@press="navigate('/rota/{{ $id }}')"` | string de path |
| receber `{id}` da rota | `mount(): void { $this->id = $this->params['id']; }` | não via construtor |

Catálogo completo de elementos/componentes e props: **elements-reference.md**.
Classes Tailwind suportadas, variantes e interação: **styling-interaction.md**.
Layouts, rotas, navegação e ciclo de vida: **layouts-routing.md**.

## Common Mistakes

| ❌ Erro | ✅ Correto |
|---|---|
| `use Native\Mobile\Components\NativeComponent` (ou `Native\Mobile\Component`) | `use Native\Mobile\Edge\NativeComponent` |
| `render(): string` / `$this->view()` | `render(): \Illuminate\View\View` / `view()` |
| `<div>` `<img>` `<p>` `<button>` HTML | `<native:column>` `<native:image>` `<native:text>` `<native:button>` |
| inventar `<view>` para divisória/container | `<native:divider>` ou `<native:column>`/`<native:row>` |
| `<native:icon name="bell">` (Lucide) | `:sf="App\Icons\SF::Bell"` ou `name="bell.fill"` (SF Symbol) |
| reimplementar toggle/card/lista na mão | usar `<native:toggle>` / `<native:card>` / `<native:list-item>` |
| receber rota via `__construct`/`mount(int $id)` | `mount(): void` lendo `$this->params['id']` |
| `Route::native(Class::class)` sem path | `Route::native('/path', Class::class)` |
| `$navigate('rota', {obj})` / `$back()` | `navigate('/path')`; back vem do `StackLayout` |
| `wire:model` | `native:model` |
