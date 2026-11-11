# Nossas Lojas V1

API de estudo para cadastrar e consultar lojas físicas de um e-commerce. Implementa operações de criação, edição, remoção e busca por identificador, estado e cidades com Express e MySQL. O projeto nasceu da [trilha de microsserviços Node.js](https://github.com/myreli/c_trilhas/blob/master/microservices_node/desafio1/README.md).

[![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

## 📋 Table of contents

- [✨ Features](#features)
- [🧪 Technologies](#technologies)
- [🛠️ Environment setup](#environment-setup)
- [🚀 Getting started](#getting-started)
- [🐋 Getting started with Docker](#docker)
- [🔌 API usage](#api)
- [🗂️ Project structure](#structure)
- [✅ Testing and quality](#testing)
- [🔎 Troubleshooting](#troubleshooting)
- [🤝 Contributing](#contributing)
- [📜 License](#license)

<a id="features"></a>

## ✨ Features

- Cadastro e alteração de nome, endereço, celular, CNPJ, horário, cidade e estado.
- Remoção por identificador e listagem de todas as lojas.
- Filtros por estado e por uma, duas ou três cidades.
- Validação dos campos no cadastro; persistência pela tabela `lojas`.
- Especificação histórica [Swagger](swagger-Paulo.yaml). O código dos controllers é a referência para as rotas atuais.

<a id="technologies"></a>

## 🧪 Technologies

Versões declaradas em [package.json](package.json); não há lockfile ou `engines.node`.

| Tecnologia | Declaração | Finalidade |
| --- | --- | --- |
| [Node.js](https://nodejs.org/) | Não fixada | Runtime JavaScript |
| [Express](https://expressjs.com/) | `^4.16.4` | Servidor HTTP |
| [mysql](https://github.com/mysqljs/mysql) | `^2.16.0` | Acesso ao MySQL |
| [express-validator](https://express-validator.github.io/) | `^5.3.1` | Validação de entrada |
| [body-parser](https://github.com/expressjs/body-parser) | `^1.18.3` | Leitura de JSON/formulários |
| [consign](https://github.com/jarradseers/consign) | `^0.1.6` | Carregamento dos módulos |
| [nodemon](https://nodemon.io/) | `1.18.4` | Reinício em desenvolvimento |

<a id="environment-setup"></a>

## 🛠️ Environment setup

Tenha Git, Node.js e acesso a um MySQL próprio. A versão do servidor MySQL não é definida no repositório. Para reproduzir a tentativa de validação, foi escolhido Node.js 14.21.3, compatível com a geração de dependências Express 4/nodemon 1; essa escolha histórica não representa atualização para um runtime atualmente suportado.

```bash
nvm install 14.21.3
nvm use 14.21.3
git clone https://github.com/paulodaluz/NossasLojasV1.git
cd NossasLojasV1
npm install
```

O fluxo usa npm, como o script e o Dockerfile. Sem lockfile, use `npm install`, não `npm ci`. Não há hooks ou script `prepare` configurados e o nodemon já é dependência local.

Configure `host`, `user`, `password` e `database` diretamente em [persistencia/connectionFactory.js](persistencia/connectionFactory.js), usando seu ambiente. O código não carrega `.env`; criar variáveis `DB_*` não altera essa conexão. Não publique os dados de acesso.

Não existem migrações. Em uma base de desenvolvimento, crie o banco e a tabela usada por `LojaDao.js`, por exemplo:

```sql
CREATE DATABASE lojas_exemplo;
USE lojas_exemplo;
CREATE TABLE lojas (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome_loja VARCHAR(100) NOT NULL,
  endereco TEXT NOT NULL,
  celular VARCHAR(12) NOT NULL,
  cnpj VARCHAR(18) NOT NULL,
  horarioDeTrabalho TEXT NOT NULL,
  cidade TEXT NOT NULL,
  estado VARCHAR(2) NOT NULL
);
```

O cadastro exige CNPJ com no mínimo 18 caracteres (`len(18)`), sem conferir dígitos verificadores. O SQL antigo do README usava 14, uma divergência que pode impedir gravar o formato aceito pelo controller. O exemplo acima reserva 18; avalie bases existentes separadamente.

<a id="getting-started"></a>

## 🚀 Getting started

```bash
npm start
```

O script executa `nodemon index.js`. A porta definida em `index.js` é **3001**, sem leitura de `PORT`; a URL base é `http://localhost:3001` e não há prefixo global.

Uma verificação sem acesso ao banco consiste em enviar um cadastro vazio e conferir HTTP 400:

```bash
curl -i -X POST http://localhost:3001/criaLoja   -H 'Content-Type: application/json' -d '{}'
```

Para verificar a integração com um banco já configurado:

```bash
curl http://localhost:3001/listaLojas
```

Não há compilação: a entrada é JavaScript. `node index.js` executa sem o observador do nodemon. Encerre com `Ctrl+C`.

<a id="docker"></a>

## 🐋 Getting started with Docker

O arquivo se chama `node.dockerfile`; a raiz é o contexto de build:

```bash
docker build -f node.dockerfile -t nossas-lojas-v1 .
docker run --name nossas-lojas-v1 -p 3001:3001 nossas-lojas-v1
docker stop nossas-lojas-v1
docker rm nossas-lojas-v1
```

Esses comandos foram elaborados por inspeção e **não foram executados nesta tarefa**. A imagem parte de `node:latest`, instala as dependências e usa `npm start`. O `EXPOSE 3000` diverge da porta 3001 do código; o mapeamento acima segue o servidor. Não há Compose, serviço MySQL ou volume configurado.

Antes de utilizar o container, a conexão deve apontar para um MySQL alcançável pelo container: `localhost` nele não é o host. O Dockerfile copia todo o contexto e desabilita a validação TLS do npm durante o build; essas configurações legadas precisam de revisão separada, inclusive para evitar incorporar credenciais.

<a id="api"></a>

## 🔌 API usage

Não há autenticação implementada nos controllers. Use JSON para os corpos.

| Método | Caminho | Uso |
| --- | --- | --- |
| POST | `/criaLoja` | Criar loja |
| PUT | `/editaLoja/:id` | Alterar loja |
| DELETE | `/deletaLoja/:id` | Remover loja |
| GET | `/buscaId/:id` | Buscar por ID |
| GET | `/buscaEstado/:estado` | Buscar por estado |
| GET | `/buscaEstadoeCidade/:estado/:cidadeA` | Buscar por estado/cidade |
| GET | `/buscaEstadoeCidade/:estado/:cidadeA/:cidadeB` | Filtrar duas cidades |
| GET | `/buscaEstadoeCidade/:estado/:cidadeA/:cidadeB/:cidadeC` | Filtrar três cidades |
| GET | `/listaLojas` | Listar todas |

Exemplo de corpo fictício para criar/editar uma loja:

```json
{
  "nome_loja": "Loja Exemplo",
  "endereco": "Rua Exemplo, 100",
  "celular": "11999999999",
  "cnpj": "00.000.000/0000-00",
  "horarioDeTrabalho": "Segunda a sexta, 9h às 18h",
  "cidade": "Cidade Exemplo",
  "estado": "SP"
}
```

A validação de CNPJ verifica o comprimento, não os dígitos verificadores. A [coleção histórica do Postman](https://www.getpostman.com/collections/821e689fc365585952a9) pode conter URLs antigas; use a porta e os caminhos desta tabela.

<a id="structure"></a>

## 🗂️ Project structure

| Caminho | Responsabilidade |
| --- | --- |
| `index.js` | Inicialização e porta |
| `config/custom-express.js` | Middlewares e carregamento dos módulos |
| `controllers/CriaLojas.js` | Rotas e validação |
| `persistencia/LojaDao.js` | Consultas SQL |
| `persistencia/connectionFactory.js` | Conexão MySQL |
| `swagger-Paulo.yaml` | Contrato histórico da API |

<a id="testing"></a>

## ✅ Testing and quality

O `npm test` atual é somente um placeholder que encerra com erro; não existe suíte automatizada, lint ou build configurado.

Instalação concluída com Node.js 14.21.3/npm 6.14.18. O aplicativo Express foi iniciado diretamente pela fábrica `config/custom-express.js` na porta temporária 4400, sem alterar arquivos; `POST /criaLoja` com `{}` retornou HTTP 400 com erros de validação. O processo foi encerrado. A entrada padrão continua usando 3001. Persistência e operações SQL não foram verificadas por falta de MySQL. Captura de backend pendente: VS Code e aparência atual do usuário não estavam disponíveis para confirmação no ambiente de validação. Docker não foi executado.

<a id="troubleshooting"></a>

## 🔎 Troubleshooting

| Situação | Verificação |
| --- | --- |
| Falha ao acessar a porta 3000 | O servidor desta versão usa 3001. |
| Falha em consultas | Confira MySQL, conexão e tabela `lojas`; iniciar o HTTP não comprova acesso ao banco. |
| CNPJ não cabe na coluna | Compare o mínimo de 18 caracteres exigido pelo controller com o schema existente. |
| `404` em filtros | Use `buscaEstadoeCidade`; o nome de rota antigo do README não corresponde ao controller. |

<a id="contributing"></a>

## 🤝 Contributing

Crie uma branch a partir de `master`, valide a instalação e os cenários afetados em um banco de desenvolvimento próprio e abra um pull request descrevendo os resultados. Correções de código e atualizações de dependências são entregas separadas. Não publique credenciais nem dados reais.

<a id="license"></a>

## 📜 License

O arquivo [LICENSE](LICENSE) contém a licença MIT, com copyright de 2021 de Paulo da Luz. O campo `license` de `package.json` declara `ISC`, divergência preservada e registrada para revisão; esta documentação usa o arquivo de licença como referência.

---

<p align="center">Made with 💜 by <a href="https://github.com/paulodaluz">Paulo da Luz</a></p>
