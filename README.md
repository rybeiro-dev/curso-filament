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

O _Filament utiliza a tabela _User_

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


> Sábado, 09 de Março de 2024, Developer by Fabio Ribeiro
