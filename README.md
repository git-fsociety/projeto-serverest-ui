# ServerRest UI

Projeto de estudo de DevOps: uma página estática (`app/index.html`) que consome a API pública de treino [ServeRest](https://serverest.dev), servida por um container **nginx** e validada por uma pipeline do **GitHub Actions**.

## Estrutura

```
.
├── app/
│   └── index.html            # página estática
├── Dockerfile                # imagem nginx com a página
├── compose.yaml              # sobe o container com Docker Compose
└── .github/workflows/ci.yml  # pipeline de CI
```

## Pré-requisitos

- Docker
- Docker Compose (plugin `docker compose` ou `docker-compose`)

## Rodando com Docker

Build da imagem:

```bash
docker build -t serverest-ui:1.0 .
```

Subir o container:

```bash
docker run -d --name serverest-ui -p 8080:80 serverest-ui:1.0
```

> As opções (`-d`, `--name`, `-p`) vêm **antes** do nome da imagem. Tudo que vem depois da imagem é passado como comando para o container.

Acesse: http://localhost:8080

Parar e remover:

```bash
docker rm -f serverest-ui
```

## Rodando com Docker Compose

```bash
docker compose up -d --build
```

```bash
docker compose ps
```

```bash
docker compose down
```

> Não misture `docker run` e `docker compose` para o mesmo serviço: os dois tentam usar a porta 8080 e o segundo falha com `port is already allocated`.

## Testando

```bash
curl -I http://localhost:8080
```

A resposta esperada começa com `HTTP/1.1 200 OK`.

## Pipeline (GitHub Actions)

O arquivo `.github/workflows/ci.yml` roda a cada push na `main`, a cada Pull Request ou manualmente pela aba **Actions**.

| Etapa | O que faz |
|---|---|
| Checkout | Baixa o código do repositório |
| Build da imagem | `docker build` a partir do Dockerfile |
| Subir container | `docker run` expondo a porta 8080 |
| Smoke test | Faz `curl` na página até 5 vezes e falha se não receber HTTP 200 |
| Logs do container | Só roda se algo falhar, para ajudar no diagnóstico |

## Comandos úteis

| Comando | Para quê |
|---|---|
| `docker ps` | Containers rodando |
| `docker ps -a` | Todos os containers, inclusive parados |
| `docker logs <nome>` | Logs do container |
| `docker ps --filter publish=8080` | Qual container está usando a porta 8080 |
| `docker images` | Imagens locais |
| `docker rmi <imagem>` | Remove uma imagem |

## Próximos passos de estudo

- Adicionar um `nginx.conf` com cabeçalhos de segurança (`X-Frame-Options`, `Content-Security-Policy`) e `server_tokens off`
- Fixar a versão da imagem base (ex.: `nginx:1.31-alpine`) em vez de `nginx:alpine`
- Adicionar lint do Dockerfile com [hadolint](https://github.com/hadolint/hadolint) na pipeline
- Adicionar scan de vulnerabilidades da imagem com [Trivy](https://github.com/aquasecurity/trivy)
- Publicar a imagem no GitHub Container Registry (GHCR)

> A ServeRest é uma API pública compartilhada: não use nela senhas reais.
