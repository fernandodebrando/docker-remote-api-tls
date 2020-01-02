# API remota do Docker com autenticação de client TLS por meio do container

Essa imagem publica a API do Docker Remote por um container.
Um client Docker deve autenticar com um certificado de client-TLS.
Essa é uma maneira alternativa (mais fácil e igualmente eficaz), em vez de [configurar o TLS no Docker diretamente](https://docs.docker.com/engine/security/https/).

## Api Remota com geração automática de CA, certificados e chaves

A imagem `docker-remote-api-tls` gera CA, certificados e chaves automaticamente.
Crie um arquivo docker-compose.yml, especificando uma senha e o nome do host, no qual a API remota será acessível posteriormente. O nome do host será gravado no certificado do servidor.

```yml
version: "3.4"
services:
  remote-api:
    image: fernandosilva/docker-remote-api-tls
    ports:
     - 2376:443
    environment:
     - CREATE_CERTS_WITH_PW=supersenha
     - CERT_HOSTNAME=remote-api.example.com
    volumes:
     - <local cert dir>:/data/certs
     - /var/run/docker.sock:/var/run/docker.sock:ro
```
Agora, execute o container com o `docker-compose up -d` ou `docker stack deploy --compose-file=docker-compose.yml remoteapidocker`.
_Obs.: Para usar docker stack o docker swarm deve ser inciado, se o mesmo ainda não estiver, basta executar:_ `docker swarm init --advertise-addr ip_da_instancia`. 
Certificados serão criados em `<local cert dir>`.
Você encontrará os certificados de client em `<local cert dir> /client/`. Os arquivos são ca.pem, cert.pem e key.pem.


## Utilização do client Docker

Se você não tiver uma instalação local do Docker, precisará fazer o download do client Docker, que é um executável simples.
Você encontra o client Docker para Linux, MacOS e Windows em [download.docker.com](https://download.docker.com/).

### Criar alias (para Linux)

Alias para conexão HTTPS com host Docker:
`alias dockerhost="docker --tlsverify -H=myserver.example.com:2376 --tlscacert=/home/user/cert/ca.pem --tlscert=/home/user/cert/cert.pem --tlskey=/home/user/cert/key.pem"`

### Criar .bat (para Windows)

Crie um arquivo "dockerhost.bat".
Para conexão HTTPS com host Docker:
`alias dockerhost="docker --tlsverify -H=myserver.example.com:2376 --tlscacert=/home/user/cert/ca.pem --tlscert=/home/user/cert/cert.pem --tlskey=/home/user/cert/key.pem"`  

*Obs.: Esta solução se baseou em alguns scripts criados por @kekru em [kekru/docker-remote-api-tls
](https://github.com/kekru/docker-remote-api-tls).*