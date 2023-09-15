# Atualização do GLPI 9.5.X para o GLPI 10.X

O GLPI 9.5 foi descontinuado em 30 de junho de 2023.

## Sumário

- [Comparação](#comparação)
- [Atualização com Docker](#atualização-com-docker)
  - [Instalação do Docker no Debian](#instalação-do-docker-no-debian)
  - [Instalação do Portainer](#instalação-do-portainer)
  - [Instalação do GLPI 10 em Docker](#instalação-do-glpi-10-em-docker)
  - [Atualização do GLPI 9.5.5 para o 10.0.9 em Docker](#atualização-do-glpi-955-para-o-1009-em-docker)
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
- [Utilização do plugin Form Creator](#utilização-do-plugin-form-creator)
  - [Criação](#criação)
  - [Edição](#edição)
  - [Utilização](#utilização)
- [Fontes](#fontes)
 
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

## Atualização com Docker

### Instalação do Docker no Debian

Como root:
```bash
apt-get update
```
```bash
apt-get install \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```
```bash
mkdir -p /etc/apt/keyrings
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Repositório:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Instalação do pacote:
```bash
apt-get update
```
```bash
apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Inicialização do serviço:
```bash
systemctl start docker
```
```bash
systemctl enable docker
```

### Instalação do Portainer

```bash
docker volume create portainer_data
```

Edição Community:
```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

http://`IP`:9443

- Escolher o nome do usuário ou admin
- Inserir a senha
- Selecionar "Get Started"

### Instalação do GLPI 10 em Docker

GLPI 10.0.9

Imagem: [sdbrasil/glpi](https://hub.docker.com/r/sdbrasil/glpi)

Instalação do GLPI totalmente funcional, com dados persistentes em rede específica para comunicação entre os containers, em menos de 5 minutos.

Um ponto importante de atenção são os campos descritos como `senha_`, necessário alterar a senha do Banco de Dados (Percona) do GLPI.

Caso tenha interesse em alterar o nome dos containers `glpi_`, não esqueca de alterar dentro do comando do GLPI Console na instalação / nome do host do Banco de Dados.

- Necessário analisar a porta "-p 8080:80".
- Estamos expondo o GLPI para a porta 8080 do navegador
- Caso queira alterar a porta (Ex: 80) você precisará alterar no comando "-p 80:80"

**1. Criar os volumes de dados persistentes para que o GLPI rode de maneira segura**

```bash
mkdir -p /data/glpi-10-dcs/glpi/documents
mkdir -p /data/glpi-10-dcs/glpi/marketplace
mkdir -p /data/glpi-10-dcs/glpi/plugins
mkdir -p /data/glpi-10-dcs/glpi/files/_pictures
mkdir -p /data/glpi-10-dcs/glpi/files/_plugins
mkdir -p /data/glpi-10-dcs/glpi/etc
mkdir -p /data/glpi-10-dcs/backup
mkdir -p /data/glpi-10-dcs/percona/lib
mkdir -p /data/glpi-10-dcs/percona/log
```

**2. Ajuste permissões dos volumes**

```bash
chown 70:70 -R /data/glpi-10-dcs/glpi
```
```bash
chown 1001:0 -R /data/glpi-10-dcs/percona
```

**3. Criar uma rede no Docker para uso exclusivo dos containers do GLPI**

```bash
docker network create glpi-dcs
```

**4. Container do Percona Server para o Banco de Dados em MySQL**

```bash
docker run -d --name glpi_db-dcs \
--restart always \
-v /data/glpi-10-dcs/percona/lib:/var/lib/mysql \
-v /data/glpi-10-dcs/percona/log:/var/log/mysql \
-e MYSQL_ROOT_PASSWORD=rootglpipassword \
-e MYSQL_DATABASE=glpi \
-e MYSQL_USER=glpi \
-e MYSQL_PASSWORD=glpipassword \
--network=glpi-dcs \
percona/percona-server:5.7 \
--character-set-server=utf8 \
--collation-server=utf8_bin
```

**5. Container da Aplicação do GLPI com a imagem da Servicedesk Brasil**

```bash
docker run -d --name glpi_app-dcs --privileged  \
-p 8080:80 \
 -v /data/glpi-10-dcs/glpi/documents/:/var/lib/glpi/files/data-documents \
-v /data/glpi-10-dcs/glpi/marketplace/:/var/lib/glpi/marketplace \
-v /data/glpi-10-dcs/glpi/marketplace/:/usr/share/glpi/marketplace \
-v /data/glpi-10-dcs/glpi/plugins/:/usr/share/glpi/plugins \
-v /data/glpi-10-dcs/glpi/files/_pictures/:/var/lib/glpi/files/_pictures \
-v /data/glpi-10-dcs/glpi/files/_plugins/:/var/lib/glpi/files/_plugins \
-v /data/glpi-10-dcs/glpi/etc/:/etc/glpi \
-v /data/glpi-10-dcs/backup/:/backup \
--network=glpi-dcs \
sdbrasil/glpi:10.0.9 /usr/sbin/init
```

**6. Instalação do GLPI dentro do Container via GLPI Console, caso queira fazer a instalação via navegador a aplicação está disponivel no endereço http://`IP`:8080**

Acesso ao container via console do Sistema Operacional
```bash
docker exec -it glpi_app-dcs /bin/bash
```

Instalação do GLPI via Console
```bash
glpi-console glpi:database:install -L pt_BR -Hglpi_db-dcs -dglpi -uglpi -pglpipassword --no-telemetry --force -n && mv /usr/share/glpi/install /usr/share/glpi/install_ori && chown -R apache:apache /usr/share/glpi/marketplace/ && chown -R apache:apache /var/lib/glpi/files && chown -R apache:apache /var/log/glpi && chown -R apache:apache /var/lib/glpi/files/data-documents && rm -rf /var/log/glpi/*
```

### Atualização do GLPI 9.5.5 para o 10.0.9 em Docker

- Imagem: [sdbrasil/glpi](https://hub.docker.com/r/sdbrasil/glpi)
- Realizar Backup do Banco de Dados (do [Docker Percona Server MySQL](https://docs.percona.com/percona-server/8.0/installation/docker.html)) e dos arquivos persistentes.

**1. Validar os volumes de dados persistentes para que o GLPI atualize de maneira segura**

```bash
mkdir -p /data/glpi-10-dcs/glpi/documents
mkdir -p /data/glpi-10-dcs/glpi/marketplace
mkdir -p /data/glpi-10-dcs/glpi/plugins
mkdir -p /data/glpi-10-dcs/glpi/files/_pictures
mkdir -p /data/glpi-10-dcs/glpi/files/_plugins
mkdir -p /data/glpi-10-dcs/glpi/etc
mkdir -p /data/glpi-10-dcs/backup
mkdir -p /data/glpi-10-dcs/percona/lib
mkdir -p /data/glpi-10-dcs/percona/log
```

**2. Desligar os containers**

```bash
docker stop glpi_app-dcs
```
```bash
docker stop glpi_db-dcs
```

Cópia segura de todos os arquivos `/data/glpi-10-dcs`
```bash
mkdir -p /data/glpi-9-dcs-Backup-9.5.5
```
```bash
cp -rp /data/glpi-10-dcs/ /data/glpi-9-dcs-Backup-9.5.5
```

Você pode executar o Backup do MySQL via console com `mysqldump` no container `glpi_db-dcs` para `/data/glpi-10-dcs/backup`.

Backup:
```bash
mysqldump -u root -p nome_banco > backup.sql
```

Se o banco de dados apresentar algum problema e precisar ser restaurado a partir do backup, siga o procedimento a seguir:

Restauração
```bash
mysql -u root -p nome_banco < backup.sql
```

Reiniciar o container Banco de Dados
```bash
docker start glpi_db-dcs
```

- Necessário analisar a porta "-p 8080:80".
- Estamos expondo o GLPI para a porta 8080 do navegador
- Caso queira alterar a porta (Ex: 80) você precisará alterar no comando "-p 80:80"

**3. Parar o container da aplicação do GLPI e depois removê-lo**
```bash
docker stop glpi_app-dcs
```
```bash
docker rm glpi_app-dcs
```

**4. Com base na TAG da nova versão, sdbrasil/glpi:10.0.9 vamos executar o comando com a nova imagem**
```bash
   docker run -d --name glpi_app-dcs --privileged  \
   -p 8080:80 \
   -v /data/glpi-10-dcs/glpi/documents/:/var/lib/glpi/files/data-documents \
   -v /data/glpi-10-dcs/glpi/marketplace/:/var/lib/glpi/marketplace \
   -v /data/glpi-10-dcs/glpi/marketplace/:/usr/share/glpi/marketplace \
   -v /data/glpi-10-dcs/glpi/plugins/:/usr/share/glpi/plugins \
   -v /data/glpi-10-dcs/glpi/files/_pictures/:/var/lib/glpi/files/_pictures \
   -v /data/glpi-10-dcs/glpi/files/_plugins/:/var/lib/glpi/files/_plugins \
   -v /data/glpi-10-dcs/glpi/etc/:/etc/glpi \
   -v /data/glpi-10-dcs/backup/:/backup \
   --network=glpi-dcs \
   sdbrasil/glpi:10.0.9 /usr/sbin/init
   ```

**5. Atualização do GLPI dentro do container via GLPI console (caso queira fazer pelo navegador, a aplicação estará disponível no endereço _http://`IP`:8080_)**

Acesso ao container via console do Sistema Operacional:
   ```bash
   docker exec -it glpi_app-dcs /bin/bash
   ```
Atualização do GLPI via console:
   ```bash
   glpi-console glpi:database:update --force -n && mv /usr/share/glpi/install /usr/share/glpi/install_ori && chown -R apache:apache /usr/share/glpi/marketplace/ && chown -R apache:apache /var/lib/glpi/files && chown -R apache:apache /var/log/glpi && chown -R apache:apache /var/lib/glpi/files/data-documents && cd /usr/share/glpi/plugins/ && glpi-console glpi:plugin:activate * && cd /usr/share/glpi/marketplace/ && glpi-console glpi:plugin:activate * && rm -rf /var/log/glpi/*
   ```

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

## Utilização do plugin Form Creator

### Criação
> Super-Admin

- ➡Menu Principal
  » 🛡Administração
    » 📋Formulários
      » ➕Adicionar

### Edição
> Super-Admin

- 📝Formulário - `Nome do formulário`
  - Formulário
    - Nome, Categoria, Ícone, Descrição, ...
  - Questões
    - ➕Adicionar uma questão
      - Nome, Tipo, Descrição, ...
  - Alvo
    - ➕Adicionar em alvo
      - Nome, Tipo
    - Seleciona um alvo para editar
      - Chamado alvo
        - Título do chamado: "`##ansewer_1##` | `##ansewer_2`"
          - Ex: _"BP153348 | Backup e Formatação"_
        - Descrição: "`##FULLFORM##`"
    - Pré-visualização
    - Propriedades de resposta do formulário
      - Lista de 'tags' disponíveis
        - | Questão | Título | Resposta | Seção |
          | --- | --- | --- | --- |
          | Formulário completo | - | `##FULLFORM##` | - |
          | Questão 1 | `##question_1##` | `##answer_1##` | Seção 1 |
          | Questão 2 | `##question_2##` | `##answer_2##` | Seção 1 |
    - Respostas do formulário: _Todas as respostas ao formulário_

### Utilização
> Self-Service

- ➡Menu Principal
  » 📋Formulários

> Super-Admin ou Technician

- ➡Menu Principal
  » 🎧Assistência
    » 📋Formulários

  - Painel Principal (`/formlist.php`)
    - Formulários
      - Ícone, Nome, Descrição
        - Visualização:

          ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/31884395-0521-4827-b4a5-da88c004b0be)
    - Visualização por Categoria
    - Organização por popularidade
    - Organização por ordem alfabética
 
- 🏠Home

  |  | GLPI 10.0.9 |
  | --- | :---: |
  | Self-Service | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/9104ad68-6860-4f4d-afaa-633e90f83993) |
  | Technician | ![image](https://github.com/michelrubens/glpi-9to10/assets/61568495/b11c663b-441c-4519-be4f-ac3d094d8c3f) |

---
## Fontes:
- [GLPI Documentation](https://glpi-project.org/documentation/)
- [Service Desk Brasil](https://www.servicedeskbrasil.com.br/)
- [Verdanatech](https://verdanatech.com/) 
