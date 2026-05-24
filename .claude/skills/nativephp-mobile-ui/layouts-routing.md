# Layouts, Routing & Lifecycle

Como telas viram rotas, ganham barras de navegação e recebem parâmetros.

## NativeComponent (ciclo de vida)

Toda tela estende `Native\Mobile\Edge\NativeComponent` (engine "Edge" — não `Native\Mobile\Components\*`).

```php
use Native\Mobile\Edge\NativeComponent;

class ItemDetail extends NativeComponent
{
    public int $id;
    public string $title = '';

    // mount(): inicialização. Parâmetros de rota chegam em $this->params,
    // NÃO como argumentos do método nem via __construct.
    public function mount(): void
    {
        $this->id = (int) ($this->params['id'] ?? 0);
        $this->title = "Item #{$this->id}";
    }

    public function navTitle(): string { return $this->title; }

    public function render(): \Illuminate\View\View
    {
        return view('item-detail');
    }
}
```

| Membro | Para quê |
|---|---|
| propriedades públicas | estado da tela (re-render automático ao mudar) |
| métodos públicos | ações (`@press="metodo"`) |
| `$this->params` | parâmetros de rota (`{id}` → `$this->params['id']`) |
| `mount(): void` | inicialização (lê `$this->params`) |
| `render(): \Illuminate\View\View` | retorna a view (`view('...')`) |
| `navTitle(): string` | título da barra |
| `navigationOptions(): ?NavBarOptions` | opções da barra (ver abaixo) |
| `showsNavBack(): bool` | retorne `false` em telas raiz para esconder o chevron |

## Roteamento

`Route::native` e `Route::nativeGroup` são **macros na facade padrão** `Illuminate\Support\Facades\Route` — use o `Route` normal do Laravel (não existe facade própria do NativePHP para rotas). Sempre passe o **path** como primeiro argumento.

```php
use Illuminate\Support\Facades\Route;
use App\NativeComponents\{ProductList, ProductDetail};
use App\NativeComponents\Layouts\StackLayout;

// rota simples
Route::native('/products', ProductList::class)->name('products');

// rota com parâmetro + layout que dá barra com back
Route::native('/products/{id}', ProductDetail::class)
    ->layout(StackLayout::class)
    ->name('products.show');

// grupo: todas as rotas compartilham o mesmo layout
Route::nativeGroup(TabsLayout::class, function () {
    Route::native('/tabs', Home::class)->name('home');
    Route::native('/tabs/browse', Browse::class)->name('browse');
});
```

## Navegação

`navigate('/path')` é um método do `NativeComponent` — chame no `@press` com a **string do path** (não nome de rota, não objeto). Não existe `$navigate(...)` nem `$back()` no front; o "voltar" vem do layout.

```blade
{{-- empurra a tela de detalhe na pilha --}}
<native:row @press="navigate('/products/{{ $product['id'] }}')" class="p-4">
    <native:text>{{ $product['name'] }}</native:text>
</native:row>
```

## Layouts

Layouts dão a "moldura" (barra superior, abas). Você os **cria no projeto** estendendo `Native\Mobile\Edge\Layouts\NativeLayout` e sobrescrevendo os métodos abaixo.

| Método | Retorno | Para quê |
|---|---|---|
| `navBar(NativeComponent $screen)` | `?NavBar` | barra superior |
| `tabBar(NativeComponent $screen)` | `?TabBar` | barra de abas inferior |
| `tabBarAccessory(NativeComponent $screen)` | `?Element` | acessório sobre as abas |
| `usesNativeChrome()` | `bool` | `true` = chrome SwiftUI nativo (NavigationStack/TabView, Liquid Glass iOS 26+) |

### StackLayout (barra com back) — para telas de detalhe

```php
use Native\Mobile\Edge\Layouts\Builders\NavBar;
use Native\Mobile\Edge\Layouts\NativeLayout;
use Native\Mobile\Edge\NativeComponent;

class StackLayout extends NativeLayout
{
    public function navBar(NativeComponent $screen): ?NavBar
    {
        return NavBar::make()
            ->title($screen->navTitle())
            ->back();
    }
}
```

### TabsLayout (abas inferiores)

```php
use App\Icons\{Material, SF};
use Native\Mobile\Edge\Layouts\Builders\{NavBar, Tab, TabBar};
use Native\Mobile\Edge\Layouts\NativeLayout;
use Native\Mobile\Edge\NativeComponent;

class TabsLayout extends NativeLayout
{
    public function navBar(NativeComponent $screen): ?NavBar
    {
        return NavBar::make()->title($screen->navTitle());
    }

    public function tabBar(NativeComponent $screen): ?TabBar
    {
        return TabBar::make()
            ->add(Tab::link('Home',    '/tabs',        sf: SF::House,           material: Material::Home))
            ->add(Tab::link('Browse',  '/tabs/browse', sf: SF::Magnifyingglass, material: Material::Search));
    }
}
```

Para chrome nativo (SwiftUI `NavigationStack`/`TabView` com Liquid Glass), adicione `public function usesNativeChrome(): bool { return true; }` ao layout.

### NavBarOptions (título grande, subtítulo)

```php
public function navigationOptions(): ?\Native\Mobile\Edge\Layouts\Builders\NavBarOptions
{
    return NavBarOptions::make()->displayMode('large')->subtitle('Toque para abrir');
}
```

## Receita completa: lista + detalhe com push/back

`routes/web.php`:
```php
use Illuminate\Support\Facades\Route;
use App\NativeComponents\{ProductList, ProductDetail};
use App\NativeComponents\Layouts\StackLayout;

Route::native('/products', ProductList::class)
    ->layout(StackLayout::class)->name('products');
Route::native('/products/{id}', ProductDetail::class)
    ->layout(StackLayout::class)->name('products.show');
```

`ProductList` (blade): cada item navega para o detalhe.
```blade
<native:scroll_view class="flex-1">
    <native:column class="gap-2 p-4">
        @foreach ($products as $product)
            <native:row @press="navigate('/products/{{ $product['id'] }}')"
                        class="items-center justify-between rounded-xl bg-white p-4 shadow">
                <native:text class="font-semibold">{{ $product['name'] }}</native:text>
                <native:icon name="chevron.right" :size="18" color="#94A3B8"/>
            </native:row>
        @endforeach
    </native:column>
</native:scroll_view>
```

`ProductDetail`: recebe o `{id}` via `mount()` (ver ciclo de vida). O botão de voltar aparece sozinho porque a rota usa `StackLayout`.
