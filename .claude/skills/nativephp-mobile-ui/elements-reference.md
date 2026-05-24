# Elements Reference

Catálogo dos elementos e componentes do NativePHP Mobile. Tudo usa o prefixo `native:`. Props extraídos de usos reais; componentes definidos em `vendor/nativephp/native-ui/src/Components/`.

## Elementos base (engine Edge)

Containers e primitivos resolvidos pela engine. Aceitam classes Tailwind (ver `styling-interaction.md`).

### `<native:column>` / `<native:row>`
Containers flex (vertical / horizontal). A base de todo layout.

```blade
<native:row class="items-center justify-between gap-3 p-4">
    <native:text class="text-base font-semibold">Título</native:text>
    <native:icon name="chevron.right" :size="18"/>
</native:row>
```

### `<native:stack>`
Sobreposição em camadas (z-index). Para badges, overlays sobre imagem.

```blade
<native:stack>
    <native:image src="{{ $url }}" class="w-20 h-20 rounded-lg"/>
    <native:badge :count="3" class="self-end"/>
</native:stack>
```

### `<native:scroll_view>`
Área rolável. Envolva listas longas.

```blade
<native:scroll_view class="flex-1">
    <native:column class="gap-2 p-4"> {{-- conteúdo --}} </native:column>
</native:scroll_view>
```

### `<native:spacer>` / `<native:divider>`
Espaço flexível que empurra o conteúdo / linha divisória. **Nunca** crie divisória com um `<view>` ou `<native:column class="h-px">` na mão.

```blade
<native:row class="items-center">
    <native:text>Esquerda</native:text>
    <native:spacer/>
    <native:text>Direita</native:text>
</native:row>
<native:divider/>
```

### `<native:pressable>`
Área tocável genérica (quando não é um botão).

```blade
<native:pressable @press="open">
    <native:card class="p-4"><native:text>Toque aqui</native:text></native:card>
</native:pressable>
```

### `<native:canvas>`
Desenho de formas (rect, circle, line). Para gráficos/shapes simples.

### `<native:text>`
Todo texto. Nunca `<p>`/`<span>`/`<h1>`. Estilo via classes (`text-`, `font-`).

```blade
<native:text class="text-2xl font-bold text-slate-900">Olá</native:text>
```

### `<native:image>`
Imagem remota/local. Nunca `<img>`. `src` + classes de tamanho/raio.

```blade
<native:image src="{{ $avatar }}" class="w-32 h-32 rounded-full"/>
```

### `<native:icon>`
Ícone. Use **SF Symbols** via `name` ou os helpers `:sf=`/`:material=` com `App\Icons\*`. Nomes estilo Lucide (`bell`, `moon`) **não existem**.

```blade
{{-- forma idiomática: helpers tipados (cross-platform) --}}
<native:icon :sf="App\Icons\SF::House" :material="App\Icons\Material::Home" :size="24" color="#475569"/>

{{-- forma direta com SF Symbol --}}
<native:icon name="minus.circle.fill" :size="24" color="#FFFFFF"/>
```

## Componentes (native-ui)

Tags em kebab-case com prefixo `native:`. Use estes em vez de reimplementar UI na mão.

### `<native:button>`
`@press` (ação), `label` (texto) ou texto via slot, `class`.

```blade
<native:button @press="addToCart" class="w-full rounded bg-blue-600 text-white">Adicionar</native:button>
<native:button @press="save" label="Salvar" class="w-full"/>
```

### `<native:list-item>`
Linha de lista rica. `:headline`, `:overline`, `:supporting`, `:leadingMonogram`, `:leadingMonogramColor`, `:trailing-badges`, `@press`.

```blade
<native:list-item
    @press="open('{{ $email['id'] }}')"
    :overline="$email['from']"
    :headline="$email['subject']"
    :supporting="$email['preview']"
    :leadingMonogram="strtoupper(substr($email['from'], 0, 1))"
/>
```

### `<native:card>`
Container com elevação/cantos. Aceita slot.

```blade
<native:card class="p-4 gap-2">
    <native:text class="text-lg font-semibold">Card</native:text>
    <native:text class="text-sm text-slate-500">Conteúdo</native:text>
</native:card>
```

### `<native:toggle>` / `<native:checkbox>` / `<native:radio>` / `<native:radio-group>`
Controles booleanos/seleção. Toggle usa `native:model` (binding two-way) + `label` + `disabled`.

```blade
<native:toggle native:model="notificationsOn" label="Notificações" class="w-full"/>
<native:checkbox :value="$agreeTerms" label="Aceito os termos" @change="toggleTerms"/>
```

### `<native:select>`
Dropdown. `:value`, `:options="[...]"`, `placeholder`, `@change`.

```blade
<native:select class="w-full" :value="$cor" placeholder="Escolha..."
    :options="['Vermelho', 'Verde', 'Azul']" @change="onCorChange"/>
```

### `<native:slider>`
`native:model` (com `.live` para atualizar enquanto arrasta), `:min`, `:max`.

```blade
<native:slider native:model.live="volume" :min="0" :max="100" class="w-full"/>
```

### `<native:modal>` / `<native:bottom-sheet>`
Sobreposições. `:visible` (estado), `@dismiss` (fechar), slot com o conteúdo. Bottom-sheet aceita `detents="small"`.

```blade
<native:bottom-sheet :visible="$sheetVisible" @dismiss="hideSheet" detents="small">
    <native:column class="p-6 gap-3">
        <native:text class="text-lg font-semibold">Opções</native:text>
        <native:button @press="hideSheet" label="Fechar"/>
    </native:column>
</native:bottom-sheet>
```

### `<native:badge>` / `<native:chip>`
`badge`: `:count`, `color`. `chip`: `label`, `:value`, `class`.

```blade
<native:badge :count="3"/>
<native:chip label="Filtro" :value="false"/>
```

### Campos de texto
`<native:filled-text-input>`, `<native:outlined-text-input>`, `<native:bare-text-input>` — use `native:model` para o valor e `placeholder`.

```blade
<native:outlined-text-input native:model="search" placeholder="Buscar..." class="w-full"/>
```

### Outros disponíveis
`<native:activity-indicator>` (loading), `<native:progress-bar>`, `<native:carousel>`, `<native:button-group>`, `<native:tab>` / `<native:tab-row>`, `<native:list>` / `<native:virtual-list>` (listas otimizadas).
