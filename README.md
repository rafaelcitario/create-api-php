Neste projeto vamos criar uma API Simple utilizando somente PHP em nenhum framework. Tudo o que vamos precisar será:

- PHP - Essencial
- Composer - Essencial
- Editor/IDE como VScode ou PHPStorm
- Docker - Preferencialmente, mas não essencial
- Postman - Preferencialmente, mas não essencial

Vamos começar definindo nosso arquivo `docker-compose.yml` para a configuração de nosso banco de dados. Caso você não queira utilizar o docker para criação de banco de dados em containers uma solução é instalar o banco de dados em sua máquina. Para este tutorial vamos utilizar o `MySQL`.

## Configuração

Após a criação da pasta onde nossa aplicação irá ficar, começamos configurando o `docker-compose.yaml`:

```yaml
services:
  mysql:
    image: mysql:9.1.0
    ports:
      - '3306:3306'
    environment:
      MYSQL_ROOT_PASSWORD: useroot
      MYSQL_USER: api_user
      MYSQL_PASSWORD: api_password
      MYSQL_DATABASE: api_example
```

Vamos quebrar este arquivo em partes para explicação:
```yaml
services:
  mysql:
```

Neste arquivo estamos definindo os serviços que serão utilizados. 
Estou dando o nome de **mysql** para este serviço. **Atenção o nome deste serviço será utilizado para a conexão com o banco dedados passando como host**

```yaml
image: mysql:9.1.0
```

Em seguida estou definindo qual a imagem será utilizada para criação do nosso banco de dados, para este projeto estou utilizando a versão **9.1.0** do **mysql**.
Você pode encontrar esta e outras versões em [Docker Hub](https://hub.docker.com/search?q=mysql).

```yaml
ports:
      - '3306:3306'
```

A porta está sendo setada como **3306**. Está é a porta padrão do mysql! 

Você pode notar que a porta está como **3306:3306**, estes **:** significam que queremos escutar esta porta em nossa maquina local e não apenas no docker container, assim podendo acessa-la diretamente em nossa maquina.

```yaml
environment:
      MYSQL_ROOT_PASSWORD: useroot
      MYSQL_USER: api_user
      MYSQL_PASSWORD: api_password
      MYSQL_DATABASE: api_example
```

Como ambiente devemos definir as credenciais para nosso serviço mysql.
Sendo assim estamos definindo usuário, senha e nome do banco de dados utilizando as variáveis de ambiente:
```yaml
MYSQL_USER: api_user // <--- Este é nosso usuário
```

```yaml
MYSQL_PASSWORD: api_password // <--- Este é nosso password
```

```yaml
MYSQL_DATABASE: api_example // <--- Este é nosso banco de dados
```

```yaml
MYSQL_ROOT_PASSWORD: useroot // <--- Está é a senha para o usuário root
```

Para iniciarmos nosso container basta estar dentro da pasta onde se encontra o arquivo `docker-compose.yaml` e digitar em nosso terminal o seguinte comando:
```bash
docker compose up -d
```

Isto inicializará o serviço mysql em nosso container.
Caso você queira acessar o mysql dentro do terminal, pode utilizar este comando:
```bash
docker exec -it <nome do container> bash
```

Após digitar este comando e apertar o enter você entrara no container onde a imagem mysql esta rodando.

> O nome do container é formado por nomedapasta-nomedohost-número
Neste caso o nome do nosso container ficaria: **create-api-php-mysql-1** se nossa aplicação fosse criada dentro do diretório "create-api-php".

Criaremos também um arquivo `composer.json`, este arquivo servira como base para instalação de bibliotecas externas que serão utilizadas no projeto. Neste projeto utilizaremos apenas a `Dotenv`.
```json
{
  "require": {
      "vlucas/phpdotenv": "^2.4"
  },
  "autoload": {
      "psr-4": {
          "Src\\": "src/"
      }
  }
}
```

Nesta linha estamos adicionando a biblioteca mais utilizada para dotenv no php. 
Você pode encontrar o repositório para esta lib em: [Github Repositório Vlucas](https://github.com/vlucas/phpdotenv)
```json
"require": {
      "vlucas/phpdotenv": "^2.4"
},
```

Na linha abaixo estamos basicamente dizendo que utilizaremos o autoload com a configuração padrão da PSR-4. As PSR's mais utilizadas atualmente são PSR-12 e PSR-4, sendo a 12 mais utilizada até o momento. Más por hora vamos seguir utilizando a PSR-4.
```json
"autoload": {
      "psr-4": {
          "Src\\": "src/"
      }
  }
```
Com estes dois arquivos criados podemos dar o comando 
```bash
composer install
```

Ele instalará a biblioteca do Dotenv e definira as configurações para a PSR desejada.
Após este comando será criado em nosso ambiente o arquivo `composer.lock`.

Para quem vem do mundo JavaScript estes arquivos podem ser comparados com o **package.json** e **package-lock.json**.

Você verá também que foi adiciona uma pasta em seu diretório com o nome `vendor` nela está localizada nossa lib Dotenv e também um arquivo muito importante: `autoload.php`.
Neste arquivo não precisamos mexer ou alterar qualquer coisa, pois ele será o responsável por transacionar as informações do Dotenv entre nossos demais arquivos.

Criaremos também um arquivo chamado `bootstrap.php` este arquivo é responsável por inicializar nossa aplicação e conectar alguns parâmetros importantes para que tudo funcione de forma esperada:
```php
<?php
require 'vendor/autoload.php';

use Dotenv\Dotenv;

$dotenv = new DotEnv(__DIR__);
$dotenv->load();

```

Podemos criar então o arquivo `.env` para adicionarmos as variáveis de ambiente que serão responsáveis por conectar com nosso banco de dados mysql. 
Adicionamos então:
```text
DB_HOST='127.0.0.1'
DB_PORT='3306'
DB_DATABASE='api_example '
DB_USERNAME='api_user'
DB_PASSWORD='api_password'
```
Criaremos também um arquivo `.env.example` onde será guardada uma cópia dessas informações para caso alguém queira clonar nosso repositório ou até mesmo para nós do futuro caso queiramos dar continuidade ao nosso projeto, assim teremos as informações necessárias para saber o que precisar definir e o que não precisamos.

```text
DB_HOST='127.0.0.1'
DB_PORT='3306'
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
```
O motivo para qual criaremos estes dois arquivos, um contendo toda a informação e outro contendo somente uma parte da informação é por que o arquivo `.env` não deve subir ao repositório, pois contem informações confidenciais. Digamos que futuramente queiramos utilizar uma API de terceiros onde precisamos adicionar um **token** para o acesso então guardaremos esta informação dentro do arquivo `.env`.

Para impedir que o arquivo `.env` suba para nosso repositório criaremos um arquivo chamado `.gitignore` e adicionaremos as seguintes informações:
```text
vendor/
.env
```
Assim definimos que o arquivo `.env` e todo o conteúdo da pasta `vendor` não serão comitados.

Com isso finalizamos a configuração do nosso projeto e estamos livres para seguir com a codificação.

## Codificação

Criaremos os seguintes diretorios src/System e dentro de System o arquivo `DatabaseConnector.php`
```php
<?php

namespace Src\System;

class DatabaseConnector {

  private $dbConnection = null;

  public function __construct() {
    $host = getenv('DB_HOST');
    $port = getenv('DB_PORT');
    $db   = getenv('DB_DATABASE');
    $user = getenv('DB_USERNAME');
    $pass = getenv('DB_PASSWORD');

    try {
      $this->dbConnection = new \PDO(
        "mysql:host=$host;port=$port;charset=utf8mb4;dbname=$db",
        $user,
        $pass
      );
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }

  public function getConnection() {
    return $this->dbConnection;
  }
}
```

Aqui estamos definindo um **namespace** para este arquivo, para que possamos utilizá-lo futuramente dentro de outros arquivos.
```php
namespace Src\System;
``` 

Criaremos nossa classe com o mesmo nome do arquivo e criaremos uma variável privada como nome `$dbConnection` passando o valor null.
Esta variável será responsável por uma nova instancia desta classe e nos conectar com o banco de dados.
Veremos mais a seguir quando implementarmos o **try-catch**.

```php
class DatabaseConnector {

  private $dbConnection = null;

  public function __construct() {
    $host = getenv('DB_HOST');
    $port = getenv('DB_PORT');
    $db   = getenv('DB_DATABASE');
    $user = getenv('DB_USERNAME');
    $pass = getenv('DB_PASSWORD');

    try {
      $this->dbConnection = new \PDO(
        "mysql:host=$host;port=$port;charset=utf8mb4;dbname=$db",
        $user,
        $pass
      );
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }
}
```

Dentro do constructor criaremos as seguintes variáveis e aferindo os valores capturados do arquivo `.env` com `Dotenv`.
```php
$host = getenv('DB_HOST');
$port = getenv('DB_PORT');
$db   = getenv('DB_DATABASE');
$user = getenv('DB_USERNAME');
$pass = getenv('DB_PASSWORD');
```

Ainda dentro do constructor faremos um try-catch para validação da ação que queremos realizar:

```php
try {
      $this->dbConnection = new \PDO(
        "mysql:host=$host;port=$port;charset=utf8mb4;dbname=$db",
        $user,
        $pass
      );
    }
```

Dentro deste try estamos tentando criar uma nova instancia da nossa classe e passando para dentro da variável $dbConnection. Estamos utilizando um módulo PDO para isso onde recebe os parâmetros 

- **DSN** - Data Source Name ou URI
- - mysql: **Sendo o serviço/banco que estamos utilizando**.
- - host=$host; **Nosso Host**
- - port=$port; **Nossa porta**
- - charset=utf8mb4; **Definição do charset utf8 para o banco de dados**
- - dbname=$db **O Nome de nosso banco de dados**
- **USER** - Usuário para login no banco de dados
- **PASS** - Senha para login no banco de dados

Caso de erro:

```php
 catch (\PDOException $e) {
      exit($e->getMessage());
    }
```

Acionaremos uma exception propria do PDO e retornamos a mensagem de erro.
Claramente este é somente um exemplo de como devemos apresentar os erros em ambiente de desenvolvimento. Para ambiente de produção é uma boa prática apresentarmos erros mais concisos que nos ajudem a entender de forma mais clara o problema ocorrido.

Já fora do constructor mas dentro de nossa classe, criaremos a seguinte função:

```php
  public function getConnection() {
    return $this->dbConnection;
  }
```

Sendo responsável por chamar nossa variável contendo a instancia de nossa conexão.

Lembra do nosso arquivo `bootstrap.php`? Vamos adicionar as seguintes linhas de código nele:

```php
use Src\System\DatabaseConnector;
$dbConnection = (new DatabaseConnector())->getConnection();
```

Ficando desta forma:

```php
<?php
require 'vendor/autoload.php';

use Dotenv\Dotenv;

use Src\System\DatabaseConnector;

$dotenv = new DotEnv(__DIR__);
$dotenv->load();

$dbConnection = (new DatabaseConnector())->getConnection();
```
Dentro da pasta **src** criaremos mais um diretório com o nome de **Database** e dentro dele o arquivo `database_seed.php`.
Este arquivo será responsável por popular nossa base de dados pela primeira vez, assim caso queiramos compartilhar este projeto com algum ele não vira com uma base de dados vazia.

Dentro deste arquivo adicionaremos os seguintes códigos:
```php
<?php
require 'bootstrap.php';

$statement = <<<EOS
  DROP TABLE IF EXISTS person;
  CREATE TABLE IF NOT EXISTS person (
      id INT NOT NULL AUTO_INCREMENT,
      firstname VARCHAR(100) NOT NULL,
      lastname VARCHAR(100) NOT NULL,
      firstparent_id INT DEFAULT NULL,
      secondparent_id INT DEFAULT NULL,
      PRIMARY KEY (id),
      FOREIGN KEY (firstparent_id)
          REFERENCES person(id)
          ON DELETE SET NULL,
      FOREIGN KEY (secondparent_id)
          REFERENCES person(id)
          ON DELETE SET NULL
  ) ENGINE=INNODB;

  INSERT INTO person
      (id, firstname, lastname, firstparent_id, secondparent_id)
  VALUES
      (1, 'Krasimir', 'Hristozov', null, null),
      (2, 'Maria', 'Hristozova', null, null),
      (3, 'Masha', 'Hristozova', 1, 2),
      (4, 'Jane', 'Smith', null, null),
      (5, 'John', 'Smith', null, null),
      (6, 'Richard', 'Smith', 4, 5),
      (7, 'Donna', 'Smith', 4, 5),
      (8, 'Josh', 'Harrelson', null, null),
      (9, 'Anna', 'Harrelson', 7, 8);
EOS;

try {
  $createTable = $dbConnection->exec($statement);
  echo "Success!\n";
} catch (\PDOException $e) {
  exit($e->getMessage());
}
```
Importamos `require 'bootstrap.php';` pois dentro de nosso arquivo bootstrap já realizamos a importação da variável que é responsável por instanciar nosso banco de dados.
```php
<?php
require 'bootstrap.php';
```
Criamos uma variável com o nome de `$statement` que tem como valor um [**Heredoc**](https://www.php.net/manual/pt_BR/language.types.string.php#language.types.string.syntax.heredoc)
```php
$statement = <<<EOS
EOL;
```

Dentro deste **Heredoc** adicionaremos algumas **querys**:
```sql
DROP TABLE IF EXISTS person;
  CREATE TABLE IF NOT EXISTS person (
      id INT NOT NULL AUTO_INCREMENT,
      firstname VARCHAR(100) NOT NULL,
      lastname VARCHAR(100) NOT NULL,
      firstparent_id INT DEFAULT NULL,
      secondparent_id INT DEFAULT NULL,
      PRIMARY KEY (id),
      FOREIGN KEY (firstparent_id)
          REFERENCES person(id)
          ON DELETE SET NULL,
      FOREIGN KEY (secondparent_id)
          REFERENCES person(id)
          ON DELETE SET NULL
  ) ENGINE=INNODB;
```
Aqui estou optando por **drop table** para derrubar toda a base e logo em seguida iniciar uma nova, entretanto se você quiser pode retirar esta linha de código.

A seguinte linha de código especifica que esta tabela será utilizada para fazer transações e tera conexção entre as tabelas. Caso queira aprender mais sobre esta declaração mysql: [documentação innoDb](https://dev.mysql.com/doc/refman/8.4/en/innodb-storage-engine.html)
```sql
ENGINE=INNODB;
```

Dentro do mesmo **Heredoc** adicionaremos outra **query**:
```sql
INSERT INTO person
      (id, firstname, lastname, firstparent_id, secondparent_id)
  VALUES
      (1, 'Krasimir', 'Hristozov', null, null),
      (2, 'Maria', 'Hristozova', null, null),
      (3, 'Masha', 'Hristozova', 1, 2),
      (4, 'Jane', 'Smith', null, null),
      (5, 'John', 'Smith', null, null),
      (6, 'Richard', 'Smith', 4, 5),
      (7, 'Donna', 'Smith', 4, 5),
      (8, 'Josh', 'Harrelson', null, null),
      (9, 'Anna', 'Harrelson', 7, 8);
```
Aqui estamos inserindo alguns dados dentro da tabela `person`.

Criamos um try-catch ao final do arquivo onde tentamos inicializar as **querys** e caso de erro retornamos uma mensagem de erro assim como fizemos nos tratamentos de dados nos códigos acima.
```php
try {
  $createTable = $dbConnection->exec($statement);
  echo "Success!\n";
} catch (\PDOException $e) {
  exit($e->getMessage());
}
```

Dentro de **src** criaremos outro diretório com o nome de `TableGateways` e dentro dela criaremos o arquivo: `PersonGateway.php`.
```php
<?php

namespace Src\TableGateways;

class PersonGateway {
  private $db = null;
  public function __construct($db) {
    $this->db = $db;
  }

  public function findAll() {
    $statement = "
      SELECT 
        id, firstname, lastname, firstparent_id, secondparent_id
      FROM
        person;
    ";

    try {
      $statement = $this->db->query($statement);
      $result    = $statement->fetchAll(\PDO::FETCH_ASSOC);
      return $result;
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }

  public function find($id) {
    $statement = "
      SELECT
        id, firstname, lastname, firstparent_id, secondparent_id
      FROM
        person
      WHERE
        id LIKE ?;
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array($id));
      $result = $statement->fetchAll(\PDO::FETCH_ASSOC);
      return $result;
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }

  public function insert(array $input) {
    $statement = "
      INSERT INTO person 
        (firstname, lastname, firstparent_id, secondparent_id)
      VALUES
        (:firstname, :lastname, :firstparent_id, :secondparent_id);
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array(
        'firstname' => $input['firstname'],
        'lastname' => $input['lastname'],
        'firstparent_id' => $input['fristparent_id'] ?? null,
        'secondparent_id' => $input['secondparent_id'] ?? null,
      ));

      return $statement->rowCount();
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }

  public function update($id, array $input) {
    $statement = "
      UPDATE person
      SET
        firstname = :firstname,
        lastname = :lastname,
        firstparent_id = :firstparent_id,
        secondparent_id = :secondparent_id
      WHERE id = :id;
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array(
        'id' => (int) $id,
        'firstname' => $input['firstname'],
        'lastname' => $input['lastname'],
        'firstparent_id' => $input['firstparent_id'] ?? null,
        'secondparent_id' => $input['secondparent_id'] ?? null,
      ));

      return $statement->rowCount();
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }

  public function delete($id) {
    $statement = "
      DELETE FROM person
      WHERE id = :id
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array('id' => $id));
      return $statement->rowCount();
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }
}
```

Os arquivos dentro desta pasta seram responsáveis por interagir com nosso banco de dados, quase como um [`Repository`](https://renicius-pagotto.medium.com/entendendo-o-repository-pattern-fcdd0c36b63b).

Em nossa classe `PersonGateway` adicionaremos o seguinte contructor:
```php
 private $db = null;
  public function __construct($db) {
    $this->db = $db;
  }
```

Adicionaremos este constructor pois nossa classe será chamada em outros arquivos para que possamos acionar alguns methodos de nossa classe.

Veja os metodos a seguir:


**Metodo responsável por listar todos os usuários de nossa tabela**
```php
public function findAll() {
    $statement = "
      SELECT 
        id, firstname, lastname, firstparent_id, secondparent_id
      FROM
        person;
    ";

    try {
      $statement = $this->db->query($statement);
      $result    = $statement->fetchAll(\PDO::FETCH_ASSOC);
      return $result;
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }
```


**Metodo responsável por listar um único usuário de nossa tabela**
```php
  public function find($id) {
    $statement = "
      SELECT
        id, firstname, lastname, firstparent_id, secondparent_id
      FROM
        person
      WHERE
        id LIKE ?;
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array($id));
      $result = $statement->fetchAll(\PDO::FETCH_ASSOC);
      return $result;
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }
```


**Metodo responsável inserir um usuário em nossa tabela**
```php
  public function insert(array $input) {
    $statement = "
      INSERT INTO person 
        (firstname, lastname, firstparent_id, secondparent_id)
      VALUES
        (:firstname, :lastname, :firstparent_id, :secondparent_id);
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array(
        'firstname' => $input['firstname'],
        'lastname' => $input['lastname'],
        'firstparent_id' => $input['fristparent_id'] ?? null,
        'secondparent_id' => $input['secondparent_id'] ?? null,
      ));

      return $statement->rowCount();
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }
```


**Metodo responsável por atualizar as informações de um usuário em nossa tabela**
```php
  public function update($id, array $input) {
    $statement = "
      UPDATE person
      SET
        firstname = :firstname,
        lastname = :lastname,
        firstparent_id = :firstparent_id,
        secondparent_id = :secondparent_id
      WHERE id = :id;
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array(
        'id' => (int) $id,
        'firstname' => $input['firstname'],
        'lastname' => $input['lastname'],
        'firstparent_id' => $input['firstparent_id'] ?? null,
        'secondparent_id' => $input['secondparent_id'] ?? null,
      ));

      return $statement->rowCount();
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }
```


**Metodo responsável por deletar um usuário de nossa tabela**
```php
  public function delete($id) {
    $statement = "
      DELETE FROM person
      WHERE id = :id
    ";

    try {
      $statement = $this->db->prepare($statement);
      $statement->execute(array('id' => $id));
      return $statement->rowCount();
    } catch (\PDOException $e) {
      exit($e->getMessage());
    }
  }
```

Criaremos dentro de **src** um diretório com o nome de `Controller` e dentro ele o arquivo: `PersonController.php`.
Os arquivos dentro deste diretório são responsáveis por interagir com a rota de nossa aplicação. Aqui interagirmos diretamente com o banco, mas poderíamos utilizar uma camada de serviços e limitar toda logica e regra de negócios para esta camada.
Caso deseje criar a camada de serviços seria desta forma:

```bash
src/
├── Controller/
├── Database/
├── System/
├── TableGateways/
└── Services/
```

Entretanto nosso intuito não é se aprofundar neste tipo de arquitetura, por hora vamos seguir com o arquivo controller:

```php
<?php

namespace Src\Controller;

use Src\TableGateways\PersonGateway;

class PersonController {
  private $db;
  private $requestMethod;
  private $userId;
  private $personGateway;
  public function __construct($db, $requestMethod, $userId) {
    $this->db            = $db;
    $this->requestMethod = $requestMethod;
    $this->userId        = $userId;

    $this->personGateway = new PersonGateway($this->db);
  }

  public function processRequest() {
    switch ($this->requestMethod) {
      case 'GET':
        if ($this->userId) {
          $response = $this->getUser($this->userId);
        } else {
          $response = $this->getAllUsers();
        }
        break;

      case 'POST':
        $response = $this->createUserFromRequest();
        break;
      case 'PUT':
        $response = $this->updateUserFromRequest($this->userId);
        break;
      case 'DELETE':
        $response = $this->deleteUser($this->userId);
        break;
      default:
        $response = $this->notFoundResponse();
        break;
    }

    header($response['status_code_header']);
    if ($response['body']) {
      echo $response['body'];
    }
  }

  private function getUser($id): mixed {
    $result = $this->personGateway->find($id);
    if (!$result) {
      return $this->notFoundResponse();
    }
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']               = json_encode($result);
    return $response;
  }

  private function getAllUsers(): mixed {
    $result = $this->personGateway->findAll();
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']                 = json_encode($result);
    return $response;
  }

  private function createUserFromRequest(): mixed {
    $input = (array) json_decode(file_get_contents('php://input'), TRUE);
    if (!$this->validatePerson($input)) {
      return $this->unprocessableEntityResponse();
    }
    $this->personGateway->insert($input);
    $response['status_code_header'] = 'HTTP/1.1 201 Created';
    $response['body']               = null;
    return $response;
  }

  private function updateUserFromRequest($id): mixed {
    $result = $this->personGateway->find($id);
    if (!$result) {
      return $this->notFoundResponse();
    }
    $input = (array) json_decode(file_get_contents('php://input'), TRUE);
    if (! $this->validatePerson($input)) {
      return $this->unprocessableEntityResponse();
    }
    $this->personGateway->update($id, $input);
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']               = null;
    return $response;
  }

  private function deleteUser($id): mixed {
    $result = $this->personGateway->find($id);
    if (! $result) {
      $this->notFoundResponse();
    }
    $this->personGateway->delete($id);
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']                = null;
    return $response;
  }

  private function validatePerson(array $input): bool {
    if (!isset($input['firstname'])) {
      return false;
    }

    if (! isset($input['lastname'])) {
      return false;
    }
    return true;
  }

  private function unprocessableEntityResponse(): array {
    $response['status_code_header'] = 'HTTP/1.1 402 Unprocessable Entity';
    $response['body']               = json_encode([
      'error' => 'Invalid Input',
    ]);

    return $response;
  }

  private function notFoundResponse(): array {
    $response['status_code_header'] = 'HTTP/1.1 404 Not Found';
    $response['body']                 = null;
    return $response;
  }
}
```

Dentro de nossa classe `PersonController` adicionaremos:
```php
private $db;
  private $requestMethod;
  private $userId;
  private $personGateway;
  public function __construct($db, $requestMethod, $userId) {
    $this->db            = $db;
    $this->requestMethod = $requestMethod;
    $this->userId        = $userId;

    $this->personGateway = new PersonGateway($this->db);
  }
```
Assim garantimos que estamos interagindo com uma nova instancia de nosso banco de dados.

Criamos também um método para processar nossas requisições:
```php
public function processRequest() {
    switch ($this->requestMethod) {
      case 'GET':
        if ($this->userId) {
          $response = $this->getUser($this->userId);
        } else {
          $response = $this->getAllUsers();
        }
        break;

      case 'POST':
        $response = $this->createUserFromRequest();
        break;
      case 'PUT':
        $response = $this->updateUserFromRequest($this->userId);
        break;
      case 'DELETE':
        $response = $this->deleteUser($this->userId);
        break;
      default:
        $response = $this->notFoundResponse();
        break;
    }

header($response['status_code_header']);
    if ($response['body']) {
      echo $response['body'];
    }
}
```

Este **header** é responsável por transmitir o `status code` e caso um `body` seja criado ele retorna este mesmo body para que seja visualizado.

```php
header($response['status_code_header']);
    if ($response['body']) {
      echo $response['body'];
    }
```

Criamos também os métodos que iram interagir com as rotas:

**Metodo responsável por interagir com a rota de listagem do usuário**
```php
private function getUser($id): mixed {
    $result = $this->personGateway->find($id);
    if (!$result) {
      return $this->notFoundResponse();
    }
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']               = json_encode($result);
    return $response;
  }

  private function getAllUsers(): mixed {
    $result = $this->personGateway->findAll();
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']                 = json_encode($result);
    return $response;
  }
```

**Metodo responsável por interagir com a rota de criação do usuário**
```php
  private function createUserFromRequest(): mixed {
    $input = (array) json_decode(file_get_contents('php://input'), TRUE);
    if (!$this->validatePerson($input)) {
      return $this->unprocessableEntityResponse();
    }
    $this->personGateway->insert($input);
    $response['status_code_header'] = 'HTTP/1.1 201 Created';
    $response['body']               = null;
    return $response;
  }
```

**Metodo responsável por interagir com a rota de update do usuário**
```php
  private function updateUserFromRequest($id): mixed {
    $result = $this->personGateway->find($id);
    if (!$result) {
      return $this->notFoundResponse();
    }
    $input = (array) json_decode(file_get_contents('php://input'), TRUE);
    if (! $this->validatePerson($input)) {
      return $this->unprocessableEntityResponse();
    }
    $this->personGateway->update($id, $input);
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']               = null;
    return $response;
  }
```

**Metodo responsável por interagir com a rota de deleção do usuário**
```php
  private function deleteUser($id): mixed {
    $result = $this->personGateway->find($id);
    if (! $result) {
      $this->notFoundResponse();
    }
    $this->personGateway->delete($id);
    $response['status_code_header'] = 'HTTP/1.1 200 OK';
    $response['body']                = null;
    return $response;
  }
```

**Métodos responsáveis por validação**
```php
  private function validatePerson(array $input): bool {
    if (!isset($input['firstname'])) {
      return false;
    }

    if (! isset($input['lastname'])) {
      return false;
    }
    return true;
  }
```

```php
private function unprocessableEntityResponse(): array {
    $response['status_code_header'] = 'HTTP/1.1 402 Unprocessable Entity';
    $response['body']               = json_encode([
      'error' => 'Invalid Input',
    ]);

    return $response;
  }
```

```php
  private function notFoundResponse(): array {
    $response['status_code_header'] = 'HTTP/1.1 404 Not Found';
    $response['body']                 = null;
    return $response;
  }
```

Por final criaremos um diretório fora de nossa pasta **src** com o nome de `Public`.
Esta pasta é responsável por conter o arquivo de exibição do php.
Criaremos dentro dela o arquivo: `index.php`
Adicionaremos o seguinte código:
```php
<?php
require "../bootstrap.php";

use Src\Controller\PersonController;

header("Access-Control-Allow-Origin: *");
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Methods: OPTIONS,GET,POST,PUT,DELETE");
header("Access-Control-Max-Age: 3600");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
$uri = explode('/', $uri);

if ($uri[1] !== 'person') {
  header("HTTP/1.1 404 Not Found");
  exit();
}

$userId = null;
if (isset($uri[2])) {
  $userId = (int) $uri[2];
}

$requestMethod = $_SERVER['REQUEST_METHOD'];

$controller = new PersonController($dbConnection, $requestMethod, $userId);
$controller->processRequest();
```

Este arquivo está sendo responsável por setar os headeres e verificar o acesso à url. Caso o acesso seja bem sucedido ele retornar os conteúdos, caso não seja ele retorna um erro.

Para acessar sua aplicação basta subir um servidor utilizando o servidor interno do PHP:
```bash
php -S localhost:8000 -t public
```
se não tiver inicializado o container digite o seguinte comando no terminal:
```bash
docker compose up -d
```
Agora basta utilizar o `postman` ou qualquer outra aplicação que te ajude a interagir com url.



> Minhas redes sociais:
> [Github](https://github.com/rafaelcitario) [Linkedin](https://linkedin.com/in/rafaelcitario)
