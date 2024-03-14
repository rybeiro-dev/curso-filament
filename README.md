# Fundamentos Filament v2 - Material de apoio

O comando a seguir cria o projeto Laravel com a infra em _docker_, utilizando _sail_.

```shell
curl -s "https://laravel.build/curso-filament?with=mysql,redis" | bash
```

Efetuar os ajustes necessários no _.env_

Ajustar o composer.json para versão do php instalado no PC.

Iniciar o projeto com _sail_

```shell
# dentro da raiz do projeto
vendor/bin/sail up -d
```

O comando a seguir instala o _Filament_ no projeto.

```shell
vendor/bin/sail composer require filament/filament:"^2.0"
```

O comando abaixo instalar o _Doctrine/dbal_ o _Filament_ vai utlizar esse pacote para gerar o CRUD com todos os campos da tabela.

```shell
vendor/bin/sail composer require doctrine/dbal --dev
```

Incluir o filamente no _composer.json_ para atualizações automáticas.

```shell
# composer.json
# adicionar dentro do array post-autoload-dump
"@php artisan filament:upgrade"
```

O comando a seguir publica o arquivo de configuração do _Filament_

```shell
vendor/bin/sail artisan vendor:publish --tag=filament-conf
```

Para validar se esta tudo certo, acesse http://localhost/admin

**IMPORTANTE:** Trabalhar com tipagem forte melhora a seguraça do projeto. Incluir em todas as classes a seguinte função:

```php
declare(strict_types=1);
```

### Configuração do Banco de dados

O _Filament_ utiliza a tabela _User_

Ajuste na _migration User_

```php
# incluir a coluna is_admin do tipo boolean e valor default=false
$table->boolean('is_admin')->default(false);
```

Ajuste na _Factory: UserFactory.php_

```php
# incluir no array
'is_admin' => fake()->boolean(),
```

Gerar dados através de _Seeder: DatabaseSeeder.php_

```php
# incluir
\App\Model|User::factory()->create([
    'name' => 'Fabio Ribeiro',
    'email' => 'fabio@fabioribeiro.dev.br',
    'password' => Hash::make('12345678'),
    'is_admin' => true,
])
# descomentar a linha
\App\Model\User::factory(35)->create();
```

Executar a _migrate_ para criar as tabelas com usuários

```shell
# o parâmetro --seed é responsável por criar os usuários
vendor/bin/sail artisan migrate --seed
```
Para testar acesso o localhost/admin e utilizar o usuário fabio@...

Em caso de FALHA: Nome de usuário não encontrado

Solução: editar a Model/User.php

```php
# ajustes
class User extends Authenticatable implements FilamentUser, HasName
{
  # trecho de código omitido

  # fazendo o cast para is_admin o laravel conventer 0 para false e 1 para true

  protected $casts = [
    'email_verified_at' => 'datetime',
    'password' => 'hashed',
    'is_admin' => 'boolean',
  ];

  public function canAccessFilament(): bool
  {
    return $this->is_admin === true;
  }

  public function getFilamentName(): string
  {
    return $this->name;
  }

}
```

## Estrutura de diretótios do Filament

App/Filament/Resources

+-- CustomerResource.php

+-- CustomerResource

|   +-- Pages

|   |   +-- CreateCustomer.php

|   |   +-- EditCustomer.php

|   |   +-- ListCustomer.php


## CRUD de Usuários

Gerado CRUD com _Filament_

```shell
vendor/bin/sail artisan make:filament-resource User --generate --simple
```

**IMPORTANTE:** A _flag --simple_ indica para o _Filament_ o uso de _Modal_. Sem essa _flag_ será gerado as páginas especifica do CRUD, conforme estrutura de diretórios.

Entendo o que foi gerado:

App/Filament/Resources

+-- UserResource.php # O principal onde todas as alterações são efetuadas.

+-- UserResource

|   +-- Pages

|   |   +-- ManagerUsers.php # só gerou esse porque é Modal

O método form() é construir os formulários atráves de classes e métodos PHP

```php
public static function form(Form $form): Form
{...}
```

O método table() é para construir o datatables através de classes e métodos PHP

```php
public static function table(Table $table): Table
{...}
```
**IMPORTANTE:** Toda alteração no método table() já altera no método form().

### Personalização

Personalizar campo _boolean_

```php
# padrão
\Tables\Columns\IconColumn::make('is_admin')->boolean();

# customizado
\Tables\Columns\ToggleColumn::make('is_admin');
```

Personalizar campo _datetime_

```php
# padrão
Tables\Columns\TextColumn::make('created_at')->dateTime();

# customizado
Tables\Columns\TextColumn::make('created_at')->dateTime('d/m/Y H:i');
```
#### Ordenação e Pesquisa

```php
Tables\Columns\TextColumn::make('name')->sortable()->searchable(),
```

#### Filtro ternário

```php
# table()
->filters([
  Tables\Filters\TernaryFilter::make('is_admin'),
])
```

## CRUD de TaskGroup

Gerar a _Model_ com _Migration_ e _Factory_

```shell
vendor/bin/sail artisan make:model TaskGroup -mf
```

Configurar a _Migration_

```php
# adicionar colunas
$table->string('title');
$table->text('description')->nullable();
```

Configurar a _Factory_

```php
# adicionar no array da função definition()
'title' => fake()->sentence(),
'description' => fake()->text(), 
```

Configurar o _Model_

```php
protected $guarded = ['id'];
```

Configurar o _Seeder_

```php
# DatabaseSeeder.php
# adicionar
TaskGroup::factory(5)->create();
```

Criar a tabela e popular com dados

```shell
# a flag --seed chama as seeder para popular as tabelas
vendor/bin/sail artisan migrate --seed
```

Gerar o _Filament_ do _TaskGroup_

```shell
vendor/bin/sail artisan make:filament-resource TaskGroup --generate --simple
```

### Personalização

##### Tooltip

```php
Tables\Columns\TextColumn::make('description')
  ->sortable()
  ->searchable()
  ->tooltip(fn (Model $record) => $record->description),
```

##### RichEditor

Ferramenta para formatação de texto

```php
Forms\Components\RichEditor::make('description')
  ->maxLength(65535),

```

##### Limit
Para limitar a quantidade de caracteres na exibição

```php
->limit(30)
```

##### Html
Para corrigir problemas de exibição de tags html
```php
->html()
```

```shell
```

```shell
```

```shell
```

```shell
```

# CRUD de Task

Gerar o crud organizado por pasta

```shell
# -m ( migration ) -f ( factory )
php artisan make:model Admin/Task -mf
```

Editar a migration task e incluir

```php
$table->foreignId('user_id')->index()->cascadeOnUpload()->cascadeOnDelete();
$table->foreignId('task_group_id')->index()->cascadeOnUpdate()->cascadeOnDelete();
$table->string('title');
$table->text('desctiption')->nullable();
```

Editar a factory

```php
# TaskFactory.php

return = [
	'user_id' => User::all()->random()->id,
	'task_group_id' => TaskGroup::all()->random()->id,
	'title' => fake()->sentence(3),
	'description' => fake()->text()
];

```

Adicionar Seeds

```php
# DatabaseSeeder.php
# adicionar
Task::factory(200)->create();
````

Executar a migration
```shell
# --seed ( para popular a tabela )
php artisan migrate:fresh --seed
```

#### Gerar o Filament do crud Task
```shell
php artisan make:filament-resource Admin/Task --generate
```

Importante fazer os relacionamentos de Task com User e TaskGroup.

### Personalizando a Task

EConfigurações paa exibir o relacionamento
```php
# App\Filament\Admin\TaskResource.php
# Na função table()
# alterar de
Table\Columns\TextColumn::make('user_id');

# para
Table\Columns\TextColumn::make('user.name');


# alterar de
Table\Columns\TextColumn::make('task_group_id');

# para
Table\Columns\TextColumn::make('taskGroup.title');
```

Ordenação e Busca no relacionamento
```php
Table\Columns\TextColumn::make('user.name')->sortable()->searchable();
Table\Columns\TextColumn::make('taskGroup.title')->sortable()->searchable();
```

#### Adicionando filtros no crud Task

Criando filtros

```php
# table()
->filters([
  # filtro seleção simples
  Tables\Filters\SelectFilter::make('user.name')->relationship('user', 'name')->searchable()->label('Usuário'),
  # filtro de seleção multipla
  Tables\Filters\SelectFilter::make('taskGroup.title')->relationship('taskGroup', 'title')->searchable()->label('Grupo de tarefas')->multiple(),
])

```

#### Criando BadgeColumns

```php
# table()
Tables\Columns\BadgeColumn::make('taskGroup.title')
  ->colors([
    'secondary',
    'primary' => 'Backlog',
    'warning' => 'In Progress',
    'success' => 'Done',
    'danger' => 'To Do'
  ])
```

#### Adicionando relacionamento no Edit e campo select

```php
# form()
Forms\Components\Select::make('user_id')->relationship('user', 'name')->searchable()->required(),
Forms\Components\Select::make('task_group_id')->relationship('taskGroup', 'title')->required(),
```

# Personalizando o Tema

As configurações de personalização do Filament podem ser encontrados em config/filament.php

#### Habilitando a opção dark mode

```php
'dark_mode' => true,
```

#### Alterando a logo 

criar o seguinte arquivo e a árvore de diretórios em view:

vendor/filament/components/brand.blade.php

```php
# brand.blade.php
<div class="flex justify-start">
  <div>
    <img src="https://placehold.co/40x40/png" alt"Logo">
  </div>
  <div class="pt-1 pl-2 text2xl font-bold tracking-tight filament-brand dark:text-white">
    {{ config('app.name') }}
  </div>
</div>

```




### Instalar as dependências Tailwindcss
```shell
npm install tailwindcss @tailwindcss/forms @tailwindcss/typography auto-prefixer typpy.js --save-dev

# criar o arquivo tailwind
npx tailwindcss init

```

Configuração e editação do tailwind.config.js
```javascript
const colors = require("tailwindcss/colors");

module.exports = {
  content: ["./resources/**/*.blade.php", "./vendor/filament/**/*.blade.php",
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        danger: colors.rose,
        primary: colors.purple,
        success: colors.green,
        warning: colors.yellow,
        secondary: colors.blue,
      },
    },
  },
  plugins: [
    require("@tailwindcss/forms"),
    require("@tailwindcss/typography"),
  ],
};
```

Criar o aquivo postcss.config.js na raiz do projeto e configurar

```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {}
  }
}
```

Customizar o css alterando o arquivo resources/css/app.css o conteúdo está disponível na documentação fo filament no blog

Sobreescrever o import default para
```css
@import "../../vendor/filament/filament/resources/css/app.css"
```

Em Provider na classe AppServiceProvider
```php

public function boot(): void
{
  Filament::serving(function(){
    Filament::registerTheme(
      app(Vite::class)('resources/css/app.css')
    )
  })
}
```

Executar o build do NPM

```shell
npm run dev

```

Solução de problema se o build quebrar: Remover do package.json a linha:  "type":"module"

IMPORTANTE: Toda alteração no css e javascript do Tailwind é necessário efetuar o build novamente.
```
```






> Sábado, 09 de Março de 2024, Developer by Fabio Ribeiro
