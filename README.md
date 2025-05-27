
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
sudo apt install -y wget apt-transport-https software-properties-common
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
sudo apt update
sudo apt install -y dotnet-sdk-6.0
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


# Habilitar clone de repositório do GitHub via SSH

1. Verificar se já existe uma chave SSH
Antes de criar uma nova chave, verifique se já existe uma:

```
ls -al ~/.ssh
```
Se aparecerem arquivos como id_rsa e id_rsa.pub, significa que você já tem um par de chaves. Se quiser reutilizá-las, pule para o passo 4.

2. Gerar uma nova chave SSH
Se não houver uma chave ou você quiser criar uma nova, use o seguinte comando:

```
ssh-keygen -t rsa -b 4096 -C "seu@email.com"
```
-t rsa: Define o tipo da chave como RSA.
-b 4096: Gera uma chave de 4096 bits para mais segurança.
-C "seu@email.com": Adiciona um comentário à chave, geralmente seu e-mail.

Pressione Enter para aceitar o local padrão (~/.ssh/id_rsa) e, se quiser, defina uma senha para proteger a chave.

3. Adicionar a chave ao SSH-Agent
Para evitar digitar a senha toda vez, adicione a chave ao agente SSH:

Inicie o ssh-agent:

```
eval "$(ssh-agent -s)"
```
Adicione a chave ao agente:

```
ssh-add ~/.ssh/id_rsa
```

4. Adicionar a chave ao GitHub/GitLab/Bitbucket
Agora, copie a chave pública para adicionar ao serviço Git remoto:

```
cat ~/.ssh/id_rsa.pub
```
Copie o conteúdo da chave e adicione no seu provedor de Git:

GitHub: Vá em Settings → SSH and GPG keys → New SSH key.

GitLab: Vá em Preferences → SSH Keys.

Bitbucket: Vá em Personal Settings → SSH Keys.

5. Testar a conexão
Agora, teste a conexão com:

```
ssh -T git@github.com
```
Se tudo estiver certo, você verá uma mensagem como:

```
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```

Para outros serviços:

GitLab: ssh -T git@gitlab.com

Bitbucket: ssh -T git@bitbucket.org

