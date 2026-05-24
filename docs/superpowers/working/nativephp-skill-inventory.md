# Inventário da fonte da verdade — nativephp-mobile-ui

Extraído de `vendor/nativephp/mobile` (engine Edge), `vendor/nativephp/native-ui` (componentes) e dos blades de `resources/views/native/`. Cada item indica a origem.

## 1. Convenção de tags (CRÍTICO)

Convenção dominante: **prefixar tudo com `native:`** — `<native:column>` (1072 usos) vs `<column>` (3). Os built-in funcionam sem prefixo, mas a regra segura/consistente é **sempre `<native:...>`**.

## 2. Elementos built-in (engine Edge)

`NativeElementCollector.php:48,530` — resolvidos diretamente:
`column`, `row`, `stack`, `scroll_view`, `spacer`, `divider`, `pressable`, `canvas`.

Conteúdo básico (usados como tag): `text`, `image`, `icon` (estes via prefixo `native:`).

| Tag | Uso |
|---|---|
| `<native:column>` | container vertical (flex column) |
| `<native:row>` | container horizontal (flex row) |
| `<native:stack>` | sobreposição (z-layers; badges, overlays) |
| `<native:scroll_view>` | área rolável |
| `<native:spacer>` | espaço flexível |
| `<native:divider>` | linha divisória (NÃO usar `<view>` na mão) |
| `<native:pressable>` | área tocável genérica |
| `<native:canvas>` | desenho (rect, circle, line) |
| `<native:text>` | texto (NÃO `<p>`/`<span>`) |
| `<native:image src="...">` | imagem (NÃO `<img>`) |
| `<native:icon>` | ícone (ver §3) |

## 3. Componentes native-ui

Tags em kebab-case com prefixo `native:`. Origem: `vendor/nativephp/native-ui/src/Components/*.php` (`elementType()`); props reais observados nos blades.

| Tag | elementType | Props/atributos reais (dos exemplos) | Slot |
|---|---|---|---|
| `<native:button>` | button | `@press`, `label`, `class`; texto via slot | sim |
| `<native:list-item>` | list_item | `@press`, `:headline`, `:overline`, `:supporting`, `:leadingMonogram`, `:leadingMonogramColor`, `:trailing-badges` | não |
| `<native:card>` | card | `class` | sim |
| `<native:toggle>` | toggle | `native:model="prop"`, `label`, `:value`, `disabled` | não |
| `<native:checkbox>` | checkbox | `:value`, `label`, `:labelColor`, `@change`, `disabled` | não |
| `<native:select>` | select | `:value`, `:options="[...]"`, `placeholder`, `@change` | não |
| `<native:slider>` | slider | `native:model.live="prop"`, `:min`, `:max` | não |
| `<native:modal>` | modal | `:visible`, `:dismissible`, `@dismiss` | sim |
| `<native:bottom-sheet>` | bottom_sheet | `:visible`, `@dismiss`, `detents="small"` | sim |
| `<native:badge>` | badge | `:count`, `color` | não |
| `<native:chip>` | chip | `label`, `:value`, `class="glass"` | não |
| `<native:icon>` | icon | `:sf="App\Icons\SF::X"`, `:material="App\Icons\Material::Y"`, `:size`, `color`, ou `name="<sf-symbol>"` | não |

Outros disponíveis (mesmo padrão): `activity_indicator`, `progress_bar`, `carousel`, `radio`/`radio_group`, `button_group`, `tab`/`tab_row`, `list`/`virtual_list`, `filled_text_input`/`outlined_text_input`/`bare_text_input`, `screen`.

### Ícones
Prop `name` aceita **SF Symbols** (ex.: `minus.circle.fill`, `add`). Idiomático: `:sf=`/`:material=` com `App\Icons\SF` e `App\Icons\Material` + `:size`. ❌ Nomes estilo Lucide (`bell`, `moon`) não existem.

## 4. Bindings e eventos

- `:prop="$valorPhp"` — passa valor PHP (estilo Livewire/Vue).
- `native:model="propPublica"` — binding bidirecional. Modificadores: `.live`, `.blur` (`TailwindParser`/blades). NÃO é `wire:model`.
- Eventos: `@press` (339), `@change` (33), `@dismiss` (12), `@submit` (2). NÃO há `$navigate`/`$back` no front.

## 5. Classes Tailwind suportadas

`TailwindParser.php` reconhece (prefixos via `str_starts_with`/`match`):

| Categoria | Prefixos |
|---|---|
| Spacing | `p- px- py- pt- pb- pl- pr- m- mx- my- mt- mb- ml- mr- gap-` |
| Cor | `bg- text- border-` (+ tema: `bg-theme- text-theme- border-theme-`) |
| Tipografia | `text-` (size+cor), `font-` |
| Radius | `rounded rounded-` |
| Shadow | `shadow shadow-` |
| Border | `border border-` |
| Opacidade | `opacity opacity-` |
| Tamanho | `w- h- w h` |
| Flex/alinhamento | `justify- items- self-` (+ `flex-1` via layout) |
| Posição | `top- bottom- left- right-` |

- **Variantes**: `dark:`, `glass:` (Liquid Glass), `ios:`, `android:` (por plataforma). → o `dark:` da cobaia 2 **é** suportado.
- **Valores arbitrários**: `w-[100]`, `h-[100]`, `text-[100]`, `bg-[#003399]`.
- **Não suportado** (ausente do parser): utilitários de `grid` (use `row`/`column`), e qualquer prefixo fora da lista acima.

## 6. Roteamento (de `routes/web.php`)

```php
Route::native('/path', Component::class)->name('nome');
Route::native('/item/{id}', Detail::class)->layout(StackLayout::class)->name('item.detail');
Route::nativeGroup(TabsLayout::class, function () {
    Route::native('/tabs', Home::class)->name('home');
});
```

## 7. Navegação

`NativeComponent::navigate(string $uri, array $data = []): static` (`NativeComponent.php:1107`). No blade: `@press="navigate('/item/{{ $id }}')"`. ❌ Não existe `$navigate('rota', {obj})` nem `$back()` (o back vem do layout).

## 8. Ciclo de vida do NativeComponent

Base: `Native\Mobile\Edge\NativeComponent` (`NativeComponent.php`).

| Membro | Assinatura | Uso |
|---|---|---|
| `$params` | `protected array $params = []` (l.37) | params de rota |
| `mount()` | `public function mount(): void` (l.792) | inicialização; lê `$this->params['id']` |
| `render()` | `public function render(): Element\|\Illuminate\View\View` (l.50) | `return view('...')` |
| `navTitle()` | `public function navTitle(): string` (l.477) | título da barra |
| `navigationOptions()` | `: ?NavBarOptions` (l.562) | opções da nav bar (displayMode, subtitle) |
| `showsNavBack()` | `bool` (opcional) | esconder chevron em telas raiz |

Estado: propriedades públicas; ações: métodos públicos (ex.: `increment()`); o re-render é automático após a ação.

## 9. Layouts

Criados no projeto estendendo `Native\Mobile\Edge\Layouts\NativeLayout`. Métodos overridable (base `NativeLayout.php:27,35,50`):

| Método | Retorno | Para quê |
|---|---|---|
| `navBar(NativeComponent $screen)` | `?NavBar` | barra superior |
| `tabBar(NativeComponent $screen)` | `?TabBar` | barra de abas inferior |
| `tabBarAccessory(NativeComponent $screen)` | `?Element` | acessório sobre as tabs |
| `usesNativeChrome()` | `bool` | `true` = chrome SwiftUI nativo (NavigationStack/TabView, Liquid Glass iOS 26+) |

Builders: `NavBar::make()->title($screen->navTitle())->back()`; `TabBar::make()->add(Tab::link('Home', '/tabs', sf: SF::House, material: Material::Home))`; `NavBarOptions::make()->displayMode('large')->subtitle('...')`.

Exemplos do projeto: `StackLayout` (navBar+back), `TabsLayout` (navBar+tabBar), `NativeStackLayout`/`NativeTabsLayout` (`usesNativeChrome()=true`).
