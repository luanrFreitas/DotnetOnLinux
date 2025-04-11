
# 🌐 Configurando uma Aplicação .NET 6 com NGINX no Ubuntu 24.04

Este guia mostra como configurar uma aplicação ASP.NET 6 rodando no Ubuntu 24.04 com NGINX como proxy reverso.

---

## ✅ Pré-requisitos

- Ubuntu 24.04 instalado e atualizado
- Aplicação ASP.NET 6 publicada
- Acesso root ou `sudo`
- Domínio configurado (opcional para HTTPS)
- Porta 5000 liberada localmente (ou a que você configurar)

---

## 🛠️ 1. Instalar o .NET 6

Atualize os pacotes e instale o runtime do .NET 6:

```bash
sudo apt update
sudo apt install -y dotnet-sdk-6.0
sudo apt install -y aspnetcore-runtime-6.0
```


## 🌐 2. Publicar a aplicação

Crie a pasta se necessário:

```bash
sudo mkdir -p /var/www/minhaapp
sudo chown -R www-data:www-data /var/www/minhaapp
```

No ambiente de desenvolvimento, execute:

```bash
dotnet publish --configuration Release --output /var/www/minhaapp
```

---

## ⚙️ 3. Criar um serviço systemd

Crie um serviço para executar sua aplicação como um daemon:

```bash
sudo vim /etc/systemd/system/minhaapp.service
```

Cole o conteúdo abaixo:

```ini
[Unit]
Description=Aplicação .NET 6 - MinhaApp
After=network.target

[Service]
WorkingDirectory=/var/www/minhaapp
ExecStart=/usr/bin/dotnet /var/www/minhaapp/MinhaApp.dll
Restart=always
RestartSec=10
SyslogIdentifier=minhaapp
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_CLI_HOME=/tmp

[Install]
WantedBy=multi-user.target
```

Salve e feche o arquivo.

Agora recarregue os serviços e inicie:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable minhaapp
sudo systemctl start minhaapp
sudo systemctl status minhaapp
```

---

## 🌍 4. Instalar e configurar o NGINX

Instale o NGINX (se ainda não tiver):

```bash
sudo apt install nginx
```

Crie um arquivo de configuração para o site:

```bash
sudo vim /etc/nginx/sites-available/minhaapp
```

Conteúdo de exemplo:

```nginx
server {
    listen 80;
    server_name seu-dominio.com;

    location / {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

Ative o site e teste a configuração:

```bash
sudo ln -s /etc/nginx/sites-available/minhaapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🔒 (Opcional) Configurar HTTPS com Let's Encrypt

Certifique-se que o domínio está apontando corretamente.

Instale o Certbot e o plugin do NGINX:

```bash
sudo apt install certbot python3-certbot-nginx
```

Execute o comando para gerar e configurar automaticamente o certificado SSL:

```bash
sudo certbot --nginx -d seu-dominio.com
```

Para testar a renovação automática:

```bash
sudo certbot renew --dry-run
```

---

## ✅ Verificando

- Acesse: `http://seu-dominio.com` (ou `https://` se usar o Let's Encrypt)
- Verifique o status do serviço:  
  ```bash
  sudo systemctl status minhaapp
  ```

---

## 🧹 Dicas finais

- Log da aplicação:
  ```bash
  journalctl -fu minhaapp
  ```
- Se alterar o `.service`, recarregue:
  ```bash
  sudo systemctl daemon-reload
  sudo systemctl restart minhaapp
  ```

---

## 📦 Estrutura final esperada

```
/var/www/minhaapp/
    MinhaApp.dll
    ...
/etc/systemd/system/minhaapp.service
/etc/nginx/sites-available/minhaapp
/etc/nginx/sites-enabled/minhaapp (link simbólico)
```

---
