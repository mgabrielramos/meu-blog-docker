# 🛰️ WordPress + Docker + Nginx + Cloudflare Tunnel (Homelab no Windows 11)

Este projeto é uma stack completa para rodar um blog WordPress **em casa**, no Windows 11, usando:

- Docker + Docker Compose
- Nginx como reverse proxy
- MySQL como banco de dados
- Cloudflare Tunnel para expor o site publicamente, mesmo com IP dinâmico e sem acesso ao roteador

Ideal para estudos de DevOps, redes, containers e auto‑hospedagem.

---

## 🔧 Arquitetura da Stack

### 1. Banco de Dados (`db` – MySQL 8)

- Contêiner baseado na imagem oficial `mysql:8.0`.
- Armazena todas as informações do WordPress: posts, usuários, configurações.
- Dados persistidos no volume Docker `db_data`, para não perder nada ao reiniciar containers.
- Credenciais e senhas são definidas via variáveis de ambiente no `.env` (não versionado).

### 2. Aplicação (`wordpress` – WordPress + PHP‑FPM)

- Contêiner baseado na imagem oficial `wordpress:php8.3-fpm` (WordPress rodando via PHP‑FPM).
- Conecta no banco MySQL usando as variáveis definidas no `docker-compose.yml` e `.env`.
- Código e arquivos do WordPress (plugins, temas, uploads) ficam no volume `wp_data`, garantindo persistência entre recriações de containers.

### 3. Servidor Web (`nginx` – Nginx como reverse proxy)

- Contêiner leve baseado em `nginx:alpine`.
- Serve arquivos estáticos (imagens, CSS, JS) e encaminha requisições PHP para o contêiner `wordpress` (PHP‑FPM).
- Configuração em `nginx.conf`, incluindo:
  - GZIP para compressão de respostas.
  - Cabeçalhos de segurança básicos (X-Frame-Options, X-Content-Type-Options, etc.).
  - Cache de página inteira com `proxy_cache` para reduzir carga em PHP/MySQL.
  - Proteção de arquivos sensíveis como `wp-config.php`.

### 4. Túnel Seguro (`tunnel` – Cloudflare Tunnel / `cloudflared`)

- Contêiner baseado na imagem oficial `cloudflare/cloudflared`.
- Cria um túnel seguro entre a sua máquina e a rede da Cloudflare usando um `TUNNEL_TOKEN`.
- Permite expor o site em um domínio público (ex: `https://seu-dominio.com`) **sem abrir portas no roteador** e mesmo atrás de CGNAT.
- O tráfego entra pela Cloudflare, passa pelo túnel até o Nginx e chega ao WordPress.

---

## 🗂 Estrutura de Arquivos

Principais arquivos deste repositório:

- `docker-compose.yml`  
  Define todos os serviços (db, wordpress, nginx, tunnel), volumes e rede interna Docker.

- `nginx.conf`  
  Configuração do Nginx com:
  - Reverse proxy para PHP‑FPM
  - Cache de página
  - GZIP
  - Cabeçalhos de segurança
  - Regras específicas para WordPress

- `.gitignore`  
  Garante que arquivos sensíveis e dados não sejam enviados ao GitHub, como:
  - `.env` (senhas, tokens)
  - Volumes locais (`db_data/`, `wp_data/`)
  - Logs, arquivos temporários etc.

> Observação: o arquivo `.env` **não** é versionado por segurança. Ele deve ser criado localmente.

---

## 🔐 Arquivo `.env` (exemplo)

Crie um arquivo chamado `.env` na raiz do projeto com variáveis como:

```env
MYSQL_ROOT_PASSWORD=troque_essa_senha_root_2026!
MYSQL_PASSWORD=senha_wp_usuario_2026!
WORDPRESS_DB_PASSWORD=senha_wp_usuario_2026!
TUNNEL_TOKEN=SEU_TOKEN_REAL_DA_CLOUDFLARE_AQUI
```

> Nunca commitar este arquivo. Ele contém senhas e o token do Cloudflare Tunnel.

---

## 🚀 Como subir a stack

1. Clonar o repositório:

```bash
git clone https://github.com/SEU_USUARIO/SEU_REPO.git
cd SEU_REPO
```

2. Criar o arquivo `.env` com suas senhas/tokens.

3. Subir os serviços:

```bash
docker compose up -d
```

4. Acessar o WordPress pelo domínio configurado no Cloudflare Tunnel (ex: `https://seu-dominio.com`) e seguir o instalador padrão.

---

## 🌐 Ajustes para Cloudflare / HTTPS no WordPress

No `wp-config.php`, podem ser adicionados ajustes para:

- Usar o IP real do visitante vindo dos cabeçalhos do Cloudflare.
- Forçar HTTPS atrás do túnel.
- Definir `WP_HOME` e `WP_SITEURL` com o domínio público.

Esses ajustes ajudam a evitar problemas de URL mista e garantem que o painel admin use sempre HTTPS.

---

## 🎯 Objetivo do Projeto

Este repositório foi criado para:

- Estudo prático de Docker, Nginx, WordPress e Cloudflare Tunnel.
- Servir como base de homelab para auto‑hospedagem de blog.
- Demonstrar arquitetura moderna de aplicação web containerizada, com foco em:
  - Separação de responsabilidades (app, banco, proxy, túnel)
  - Segurança básica
  - Reprodutibilidade (subir tudo com um único comando `docker compose up -d`).

Sinta‑se à vontade para forkar, adaptar e evoluir essa stack (cache mais agressivo, Redis, monitoramento, etc.).
