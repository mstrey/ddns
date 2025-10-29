# Cloudflare DDNS

A primeira coisa necessária para utilizar o projeto é copiar o arquivo `.env_example` nomeando o novo arquivo como `.env`.
Altere o valor da variável `DOMAIN` para o seu domínio gerenciado pela cloudflare.

Para utilizar essa solução de DDNS você precisa de um Token de API e do ID da Zona para que o Docker possa fazer as alterações de DNS em seu nome.

## Obter o ZONE_ID
Faça login no seu [painel Cloudflare](https://dash.cloudflare.com/).   
Selecione seu domínio.   
Na página Overview, role para baixo e copie o valor de Zone ID.
Edite o arquivo `.env` colando o valor ao lado da variável `ZONE_ID`

## Criar o Token de API
Vamos utilizar o Token API por ser mais seguro que a Chave Global. Esse token dará acesso específicamente para alterar o IP e mais nada.

No painel Cloudflare, clique no seu perfil (canto superior direito) e cliqeu na opção Profile.   
Vá para a aba API Tokens e clique em Create Token.   
Use o modelo 'Edit Zone DNS'.   
Configure em Permissions:   
- Zone -> DNS -> Edit

Configure em Zone Resources:
- Include -> Specific zone -> seu domínio (mesmo valor que foi informado no parametro `DOMAIN` do arquivo `.env`).

Clique em Continue to Summary e depois em Create Token.
Copie o Token de API imediatamente. Ele não será exibido novamente.
Cole o valor do parametro `CF_API_TOKEN` do arquivo `.env`.

## Execute o Container
No seu prompt de comando, navegue até o diretório raiz do projeto e execute:

```bash
docker compose up -d
```

## 🎯 O que o Docker faz agora?
O container será iniciado e será executado em segundo plano (-d).

A cada 5 minutos (API_FREQUENCY=300), ele verificará o seu IP público atual (o IP de internet do seu roteador).

Se o IP for diferente do que está registrado no Cloudflare para o seu hostname, ele usará o CF_API_TOKEN e o ZONE_ID para atualizar automaticamente o Registro A no Cloudflare com o novo IP.

## Verificação Final
Verifique os logs do container para ter certeza de que a atualização inicial foi bem-sucedida:

```bash
docker logs cloudflare-ddns
```

Você deverá ver mensagens de sucesso como: 
- Se o IP for o mesmo já cadastrado:
```
Current IP x.x.x.x matched DNS record y.y.y.y. No update required.
```
- Se o IP tiver mudado:
```
 Updated DNS record for domain.net.br to x.x.x.x
```
