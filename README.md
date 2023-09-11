# Atualização do GLPI 9.5.X para o GLPI 10.X

O GLPI 9.5 foi descontinuado em 30 de junho de 2023.

## Sumário

- [Comparação](#comparação)
- [Atualização](#atualização)
  - [Verificação dos requisitos](#verificação-dos-requisitos)
  - [Backup](#backup)
  - [Etapas para atualização do GLPI](#etapas-para-atualização-do-glpi)
- [Fuso Horários](#fuso-horários)
  - [Utilizadores não Windows](#utilizadores-não-windows) 
- [Utilização do plugin GLPI Agent e GLPI Agent Monitor](#utilização-do-plugin-glpi-agent-e-glpi-agent-monitor)
  - [Download](#download)
  - [Instalação](#instalação)
  - [Forçando inventário](#forçando-inventário)
 
## Comparação

| | GLPI 9.5.5 | GLPI 10.0.7 |
| :--- | :---: | :---: |
| Criação de chamado à partir da tela inicial | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/dc7a5b37-b2fd-4b33-9f15-d1074e500442) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/2561f98e-c139-4b34-8c6c-ba4d9d55b5ee) |
| Formulário de criação de chamado (Technician) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/ec9e01d1-7135-4792-b657-37114149d8f1) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/cafc71a7-a2cb-4f5b-a30d-2a1dcace430e) |
| Chamados | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/086a048b-5ae3-4114-8f27-eac10152f687) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/177e97a6-a771-41fd-8d4a-75c6329f5260) |
| Ativos | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/2a433c27-805c-4fe0-a1b0-f518ffea1054) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/bec77450-7812-4c75-865b-d2bd729d2171) |
| Base de Conhecimento | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/d80a8606-e5d3-4121-9a65-96bd6a6619d4) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/80ebe7d3-e25d-4d7c-a89c-6177b9799f02) |
| Acesso aos Menus | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/5d7fc77c-599d-4c51-965d-cd25396d83b2) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/9fa1c2f5-8e5b-44b4-9cb0-3c0571b4fa05) |
| Visão global | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/429a9cca-940a-4649-b2b8-b0f03016c780) | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/48e0cf9e-4cb3-4a06-a433-759268ccadee) |

## Atualização

### Verificação dos requisitos

- Antes de instalar ou [atualizar](https://glpi-install.readthedocs.io/pt/latest/update.html), os requisitos são verificados automaticamente; mas você pode executá-los separadamente e ver o estado de todos eles usando o comando `php bin/console glpi:system:check_requirements`.

### Backup
- Backup do **Banco de Dados**;
- Backup do diretório `config`, especialmente para o seu arquivo de chave GLPI (`config/glpi.key` ou `config/glpicrypt.key`) que é gerado aleatoriamente;
- Backup do diretório `files`, que contém arquivos gerados por **usuários** e _plugins_, como documentos "upados";
- Backup dos diretórios `marketplace` e `plugins`.
  
### Etapas para atualização do GLPI
-  Baixe a [última versão](https://github.com/glpi-project/glpi/releases) do GLPI
-  Certifique-se de que o diretório de destino esteja vazio e extraia os arquivos lá.
-  Restaure os diretórios `config`, `files`, `marketplace` e `plugins`.
-  Em seguida, abra a URI da instância GLPI em seu navegador ou use a ferramenta de linha de comando `php bin/console db:update` (recomendado).
 
> [!WARNING]
>  Você não deve tentar restaurar um backup de banco de dados em um banco de dados não vazio (por exemplo, um banco de dados parcialmente migrado por qualquer motivo).
>
> Verifique se o banco de dados está vazio antes de restaurar o backup e tente atualizar, e repita o processo caso ocorra alguma falha.

> [!NOTE]
> O processo de atualização desativará automaticamente seus plugins.

> [!NOTE]
> Desde o GLPI 10.0.1, você pode usar a ferramenta de linha de comando `php bin/console db:check` antes de executar o comando **update**. Isso permitirá que você verifique a integridade do seu banco de dados e identifique alterações que possam comprometer a atualização.

## Fuso Horários

Para que os fusos horários funcionem em uma instância do MariaDB/MySQL, você precisará inicializar os dados dos fusos horários e permitir que o usuário do banco de dados GLPI leia a ACL em sua tabela.

> [!WARNING]
> Atualmente, o MySQL e o MariaDB têm uma data máxima limitada a 2038-01-19 em campos que dependem do tipo `timestamp`!

### Utilizadores não Windows

Na maioria dos sistemas, você precisará inicializar os dados dos fusos horários a partir dos fusos horários do sistema:

```console
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
```
Você pode verificar a documentação do MariaDB sobre mysql_tzinfo_to_sql e a documentação do sistema para saber onde os dados estão armazenados (se não estiver em `/usr/share/zoneinfo`).

Não esqueça de reiniciar o servidor de banco de dados quando o comando for bem-sucedido.

## Utilização do plugin GLPI Agent e GLPI Agent Monitor

### Download

- [GLPI Agent v1.5](https://github.com/glpi-project/glpi-agent/releases/tag/1.5)
- [Outros releases](https://github.com/glpi-project/glpi-agent/releases)

### Instalação

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/e31b3618-552c-4612-8bc7-72db3f8dc09e)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/49b054e3-8e62-401e-a63a-e48154dfa7c0)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/d398f4d3-2ef3-408c-9203-e8724c643a0c)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/aeb04afd-97c2-48d2-972b-67e7d3bc2ea4)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/18c6f56d-c1a6-4acd-9ae1-dd3be55c75a8)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/ee117fdd-d220-4dcc-9482-1185c11938fe)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/a68c56d0-0590-4919-bfc6-6049217b5391)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/39846b96-aae6-43a8-a553-2d10d13f6594)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/edef3c80-bf41-43b0-8a26-933e24f262e8)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/d1236bce-5c7f-40e8-9c64-69a405185221)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/8c9581ba-8a23-4d94-91bb-4f37fd0c2721)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/887fcf60-7b2c-4766-a715-6acb5fc5c2dd)

### Forçando inventário

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/e92c1f01-2355-475a-b98b-0008b305fe3e)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/38c3ec76-1d19-469a-88f7-42b884a1127a)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/aeab45db-8b5a-4a09-962c-80c19a12f8d7)

![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/073214f6-7cb9-43ea-9775-ae75b0741798)
