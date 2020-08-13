## VPN NO MIKROTIK COM OPENVPN

Use o terminal do mikrotik para executar os comandos, ou pelo ssh.

### *1 - Certificado Servidor:*

/certificate add name=ca country=”br” state=”sao_paulo” locality=”cidade” organization=”organizacao” unit=”setor|departamento” common-name=”ca” key-size=4096 days-valid=3650 key-usage=crl-sign,key-cert-sign

/certificate sign ca ca-crl-host=127.0.0.1 name=”ca”

/certificate add name=server country=”br” state=”sao_paulo” locality=”cidade” organization=”organizacao” unit=”setor|departamento” common-name=”ip_publico” key-size=4096 days-valid=3650 key-usage=digital-signature,key-encipherment,tls-server

/certificate sign server ca=”ca” name=”server”

### *2 - Certificado cliente:*

/certificate add name=client country=”br” state=”sao_paulo” locality=”cidade” organization=”organizacao” unit=”setor|departamento” common-name=”client” key-size=4096 days-valid=3650 key-usage=tls-client

/certificate add name=”nome-cliente” copy-from=client common-name=”nome-cliente”

/certificate sign nome-cliente ca=”ca” name=”nome-cliente” 

### *3 - Exportando certificados:*

/certificate export-certificate ca export-passphrase=””

/certificate export-certificate nome-cliente export-passphrase=senha-cliente

### *4 - Criar Pool*

Criar uma Pool com o nome de vpn e colocar o range que vai ser dado ao usuário quando conectar.

/ip pool name=vpn ranges=10.10.10.2-10.10.10.20

### *5 – Adicionar profile* 

/ppp profile add name=vpn local-address=10.10.10.1 remote-address=vpn change-tcp-mss=yes use-upnp=default

### *6 - Criar um novo PPP Secret*

/ppp secret add name=joao.santos service=ovpn password=#123!321# profile=vpn

name = nome do cliente  
password = senha do cliente

### *7 – Configure OVPN Server*

/interface ovpn-server server set enabled=yes port=1194 mode=ip netmask=24 max-mtu=1500 keepalive-timeout=60 default-profile=vpn certificate=server require-client-certificate=yes auth=sha1 cipher=aes256

port = porta da vpn (padrão 1194)

### *8 - Faça o download dos seguintes certificados:*

ca.crt  
nome-cliente.crt  
nome-cliente.key

### *8.1 - Clique em Files*
![Imagem-1](https://uploaddeimagens.com.br/images/002/827/458/full/1.png?1597336852)

### *8.2 - Clique com o direito no arquivo e em Download*

![Imagem-2](https://uploaddeimagens.com.br/images/002/827/470/full/2.png?1597337043)

### *8.3 – Escolha onde salvar*
![Imagem-3](https://uploaddeimagens.com.br/images/002/827/472/original/3.png?1597337098)

### *9 – Crie o arquivo: nome-arquivo.ovpn*

Obs.

\<ca> conteúdo do arquivo ca.crt \</ca>  
\<cert> conteúdo do arquivo nome-cliente.crt \</cert>  
\<key> conteúdo do arquivo nome-cliente.key \</key>

Exemplo do conteúdo do arquivo:

client  
dev tun  
proto tcp-client  
\# remote (ip válido do mikrotik)  
remote 172.100.3.4  
\# port (essa porta foi escolhida no passo 7, ex. 1194)  
port 1194  
nobind  
persist-key  
persist-tun  
tls-client  
remote-cert-tls server  
verb 2  
mute 10  
cipher AES-256-CBC  
auth SHA1  
auth-user-pass secret.txt  
auth-nocache  
route 10.10.10.0 255.255.255.0  
\# 192.168.0.0 (rede local mikrotik)  
\# 10.10.10.1 (ip tunel vpn mikrotik)  
route 192.168.0.0 255.255.255.0 10.10.10.1  
\<ca>  
-----BEGIN CERTIFICATE-----
MIIF3jCCA8agAwIBAgIIJ3AwVTizeB8wDQYJKoZIhvcNAQELBQAwYjELMAkGA1UE
BhMCYnIxEjAQBgNVBAgMCXNhb19wYXVsbzESMBAGA1UEBwwJcGVkZXJjaXR5MREw
DwYDVQQKDAhzaXJsaW51eDELMAkGA1UECwwCaXQxCzAJBgNVBAMMAmNhMB4XDTIw
MDgxMzA5NDMwMVoXDTMwMDgxMTA5NDMwMVowYjELMAkGA1UEBhMCYnIxEjAQBgNV
BAgMCXNhb19wYXVsbzESMBAGA1UEBwwJcGVkZXJjaXR5MREwDwYDVQQKDAhzaXJs
aW51eDELMAkGA1UECwwCaXQxCzAJBgNVBAMMAmNhMIICIjANBgkqhkiG9w0BAQEF
AAOCAg8AMIICCgKCAgEA1kf1V9XjYzxnkjMamiTPE8kaaXsuh7nzMoYj4aHMWy0q
rLC5baunv/sa4YPQhAsifLOSVRZ5O79QJBPOJu03hnz9j8niXmAvLFOeEYEaQpqT
mMdoyK2zweDifa5EckkxCUP79B0N6hg222VOMemBUE7ZtVa5I4gHNQs4rScXntAq
u2zqsRoUEUESvT0mZr+X8V0E01+SoYTksLmn1z76clqa/vXePnkMNGwA7pAlg8QU
6L7NuzsvrUhIkW3INVXVBI3hN53dEdaouhGF9Gyr2CfE5lfOQ10aIyLcll8f3319
5H2tSo2hksWDtrTnWbiz0+67599oOonU8bGftHWKqEd1yrf9BbmNvrB+b+DXHrjy
u8XC/PaPGjMa8DUVBE07yNgTdnx+/bb/qwvSGHY6bI5bXmxeigywrzHpxmqxDccN
0vDFp9ZEX+aY97sXBSDlShIyCSrQfa1RaAfFHUMZK0RPCt8OE378gVjhW9+PfElf
nl65SC48dBTBgabrrV4xmKNwdauQswNoUeaevaFGjyDeQFlyOkcacYpEt3YLSrk2
wpQEbYxmbGixUy9IwUBj4QlKS+AfqjBB43JVOEbmIkJruabrzyYaUPXhqhFmFTsK
BKDv9V0AvEvfn9tqw1qBNo74190xwizOUjgkWm5pNAKtPO0s9mfb1xO9aJEXJEEC
AwEAAaOBlzCBlDAPBgNVHRMBAf8EBTADAQH/MA4GA1UdDwEB/wQEAwIBBjAdBgNV
HQ4EFgQUl1uXgp2VgtsoujobeQ8em8AmXvowLAYDVR0fBCUwIzAhoB+gHYYbaHR0
cDovLzEyNy4wLjAuMS9jcmwvMTIuY3JsMCQGCWCGSAGG+EIBDQQXFhVHZW5lcmF0
ZWQgYnkgUm91dGVyT1MwDQYJKoZIhvcNAQELBQADggIBAKcwgkOTPCGvjzhqxIcU
vImp+ph++2Vi3tRDQla7xrYy+wkeXYtJg37jfaI44vwPWeEpuyzjvJPFbaurdQmN
aHtLcKl4JRqeC/xg9sk4gQYUVh1KRvXM9WG1LAikpzmTaF8BCGzwurA+LwA+TbcR
zvYA0IavShcK0Am+Tt6kpuXYQsRxD59CgLdjH9n/TLlViY/FVXLDLwm4Qc5YwkaD
8YctwlNGa9ocn4RIlO4/oXJERNDZCdkhj/dIMtjVDNAR6S22eW0wzP+/7kfMy76N
2YEtJdVNeQWh41x4F0y3iCJVidGHHKKjf6Oitb8a28OjsJI0yBACgN5MPajLkBgA
4TE21AaxD7tQLsG5PfcuptXhNh94IcJBQZYNutsHIGXtuSjx11/G2ty2WOjr9JpE
x/1cJBFZHqgJalmz5YWMkb2UNL6EPcMMI6Mp6rTTCEOiU55/CNMNnX+QXSYNoPvc
Qu4uzuxrpbYxVc0emhA3y5RsZd46sbdutdj6YY0f4fra0RpK+/IDMdP7vHjwNTZD
6skLkw8BmaiW02hJ9fo5IGE/ddHVnk/BgzWQCtrkGTbUE50f+G14kRWige0YOfOZ
YaFpWE6foB4hyexQWGRU97Y5RGTyXMF/nK8syaHZvz/pkcjaQTOTBmGUo2izwyCN
o+uxMDGV0HJaPS9jXGN6TyvZ
-----END CERTIFICATE-----  
\</ca>  
\<cert>  
-----BEGIN CERTIFICATE-----
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
cVdydB4=
-----END CERTIFICATE-----  
\</cert>  
\<key>  
-----BEGIN ENCRYPTED PRIVATE KEY-----
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
6xGWUO9GygLgjqteV4L54Xycpg==
-----END ENCRYPTED PRIVATE KEY-----  
\</key>

## *10 – Crie o arquivo secret.txt*

Conteúdo do arquivo, não coloque espaços ou saltos de linhas:
```diff
- -->(inicio do arquivo)
```
joao.santos  
#123!321#  

```diff
- <--(fim do arquivo)
```
Obs.  
name (joao.santos) e password (#123!321#) foram criados no passo 6

## *11 – Copie os arquivos: nome-arquivo.ovpn e secret.txt para o computador do cliente.*
