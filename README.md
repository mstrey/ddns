# Cloudflare DDNS

A primeira coisa necessária para utilizar o projeto é copiar o arquivo `.env_example` nomeando o novo arquivo como `.env`.
Altere o valor da variável `DOMAINS` com a lista de domínios gerenciados pela cloudflare.

Para utilizar essa solução de DDNS você precisa de um Token de API para que o Docker possa fazer as alterações de DNS no seu nome.

## Criar o Token de API
Vamos utilizar o Token API por ser mais seguro que a Chave Global. Esse token dará acesso específicamente para alterar o IP e mais nada.

Faça login no seu [painel Cloudflare](https://dash.cloudflare.com/).   
No painel Cloudflare, clique no seu perfil (canto superior direito) e clique na opção `Profile`.   
Vá para a aba `API Tokens` e clique em `Create Token`.   
Use o modelo `Edit Zone DNS`.   
No grupo `Permissions` configure:   
- Zone -> DNS -> Edit

No grupo `Zone Resources`:
- Para liberar atualização para todos os dominios de uma conta:
    - Include -> All zones from an account -> selecione a conta.
- Para liberar atualização para um ou mais dominios, para cada domínio informado no parâmetro `DOMAINS` do arquivo `.env`:
    - Include -> Specific zone -> selecione o domínio.

Clique em `Continue to Summary` e depois em `Create Token`.
Copie o Token de API agora. Ele não será exibido novamente.
Cole o valor do parametro `CF_API_TOKEN` do arquivo `.env`.

## Execute o Container
No seu prompt de comando, navegue até o diretório raiz do projeto e execute:

```
docker compose up -d
```

## 🎯 O que o Docker faz agora?
O container será iniciado e será executado em segundo plano (-d).

A cada 5 minutos (API_FREQUENCY=300), ele verificará o seu IP público atual (o IP de internet do seu roteador).

Se o IP for diferente do que está registrado no Cloudflare para o seu hostname, ele usará o CF_API_TOKEN e o ZONE_ID para atualizar automaticamente o Registro A no Cloudflare com o novo IP.

## Verificação Final
Verifique os logs do container para ter certeza de que a atualização inicial foi bem-sucedida:

```
docker logs cloudflare-ddns --timestamps
```

Você deverá ver mensagens de sucesso como: 
```
2025-10-31T13:37:58.110136443Z 🌐 Detected the IPv4 address 192.168.1.1
2025-10-31T13:37:58.110180536Z 🤷 The A records of domain.net.br are already up to date (cached)
2025-10-31T13:37:58.192034574Z 🌐 Detected the IPv6 address 0000:0000:0000:0000:0000:ffff:C0A8:0101
2025-10-31T13:37:58.192066889Z 🤷 The AAAA records of domain.net.br are already up to date (cached)
2025-10-31T13:37:58.192174519Z ⏰ Checking the IP addresses in about 5m0s . . .
```
