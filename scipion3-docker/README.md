# Scipion Docker

Docker image with **Scipion + CUDA + noVNC** for browser-based graphical access.

Imagem Docker com **Scipion + CUDA + noVNC** para acesso gráfico pelo navegador.

---

# Português

## Build

Na pasta onde está o `Dockerfile`:

```bash
docker build -t scipion .
```

## Executar

```bash
docker run -d \
    --name scipion \
    --gpus all \
    -p 6080:6080 \
    -e VNC_PASSWORD='sua_senha' \
    -v /data:/data \
    scipion
```

## Acessar pelo navegador

Abra:

```text
http://IP_DO_SERVIDOR:6080/vnc.html
```

Exemplo:

```text
http://192.168.0.82:6080/vnc.html
```

Use a senha definida em:

```bash
-e VNC_PASSWORD='sua_senha'
```

## Abrir o Scipion

No terminal do desktop remoto:

```bash
/opt/scipion3/scipion3
```

## Verificar GPU

```bash
docker exec -it scipion nvidia-smi
```

## Entrar no container

```bash
docker exec -it scipion bash
```

## Logs

```bash
docker logs -f scipion
```

## Parar / iniciar

```bash
docker stop scipion
docker start scipion
```

## Remover

```bash
docker stop scipion
docker rm scipion
```

Os dados importantes devem ficar em volumes montados, como:

```bash
-v /data:/data
```

---

# English

## Build

From the directory containing the `Dockerfile`:

```bash
docker build -t scipion .
```

## Run

```bash
docker run -d \
    --name scipion \
    --gpus all \
    -p 6080:6080 \
    -e VNC_PASSWORD='your_password' \
    -v /data:/data \
    scipion
```

## Open in the browser

Open:

```text
http://SERVER_IP:6080/vnc.html
```

Example:

```text
http://192.168.0.82:6080/vnc.html
```

Use the password defined with:

```bash
-e VNC_PASSWORD='your_password'
```

## Start Scipion

From a terminal inside the remote desktop:

```bash
/opt/scipion3/scipion3
```

## Check GPU

```bash
docker exec -it scipion nvidia-smi
```

## Open a shell

```bash
docker exec -it scipion bash
```

## Logs

```bash
docker logs -f scipion
```

## Stop / start

```bash
docker stop scipion
docker start scipion
```

## Remove

```bash
docker stop scipion
docker rm scipion
```

Important data should be stored in mounted volumes, for example:

```bash
-v /data:/data
```