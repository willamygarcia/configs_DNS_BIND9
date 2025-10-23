>Olá me chamo Willamy Garcia e com esse guia você irá conseguir instalar e configurar
>um Servidor de domínio sem grandes problemas.
>Espero que gostem e qualquer dúvida pode me chamar nas minhas redes sociais
>
>@willamygarcia no Instagram



# INSTALAÇÃO E CONFIGURAÇÃO
### Substitua-os pelos valores da sua própria rede:

* Domínio da Rede: meudominio.local
* Rede IP: 192.168.1.0/24
* IP do Servidor DNS: 192.168.1.10
* Hostname do Servidor DNS: ns1.meudominio.local

## Instalação do BIND9
### Comandos para Instalação
```
sudo apt update
sudo apt install bind9 bind9utils -y
```

## Configuração do BIND9
### Configuração Principal (named.conf.options)

Abra o arquivo *named.conf.options* no local /etc/bind/

Escreva o código abaixo:
*Obs: Observe os comentários no código*
```
options {
        directory "/var/cache/bind";

        // Adicione os IPs de encaminhadores públicos
        forwarders {
                8.8.8.8; // DNS do Google
                1.1.1.1; // DNS da Cloudflare
        };

        // Restringe quem pode consultar este servidor DNS
        // 'localhost' é permitido por padrão. Adicione sua rede local.
        allow-query { localhost; 192.168.1.0/24; };  // Esse ip de rede local, é o ip da sua rede. 

        dnssec-validation auto;
        listen-on-v6 { any; };
};
```


### Definição das Zonas (named.conf.local)

Abra o arquivo *named.conf.local* no local /etc/bind/
Escreva o código abaixo:
*Obs: Observe os comentários no código*

```
// Zona de Pesquisa Direta (Nome -> IP)
zone "meudominio.local" {
        type master;
        file "/etc/bind/zones/db.meudominio.local"; // Caminho para o arquivo da zona
};

// Zona de Pesquisa Inversa (IP -> Nome)
// "1.168.192" é o reverso de "192.168.1"
zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/zones/db.192.168.1"; // Caminho para o arquivo da zona inversa
};

```

## Criação dos Arquivos de Zona

### Crie o diretório para as zonas
```
sudo mkdir /etc/bind/zones
```

### Crie o Arquivo da Zona Direta (db.meudominio.local)
```
sudo nano /etc/bind/zones/db.meudominio.local
```

*Copie e cole o código abaixo*

```
;
; Arquivo de zona para meudominio.local
;
$TTL    604800
@       IN      SOA     ns1.meudominio.local. admin.meudominio.local. (
                              2025102201   ; Serial (Use a data AAMMDDHH + ID)
                              604800       ; Refresh
                              86400        ; Retry
                              2419200      ; Expire
                              604800 )     ; Negative Cache TTL
;
; Servidores de Nomes (NS)
@       IN      NS      ns1.meudominio.local.

; Registros A (Endereços IPv4)
ns1     IN      A       192.168.1.10
web     IN      A       192.168.1.20
db      IN      A       192.168.1.30
```


### Crie o Arquivo da Zona Inversa (db.192.168.1)

```
sudo nano /etc/bind/zones/db.192.168.1
```
*Copie e cole o código abaixo*

```
;
; Arquivo de zona inversa para 192.168.1.0/24
;
$TTL    604800
@       IN      SOA     ns1.meudominio.local. admin.meudominio.local. (
                              2025102201   ; Serial
                              604800       ; Refresh
                              86400        ; Retry
                              2419200      ; Expire
                              604800 )     ; Negative Cache TTL
;
; Servidor de Nomes
@       IN      NS      ns1.meudominio.local.

; Registros PTR (Ponteiros Inversos)
; Note que usamos apenas o último octeto do IP
10      IN      PTR     ns1.meudominio.local.  ; (Para 192.168.1.10)
20      IN      PTR     web.meudominio.local.  ; (Para 192.168.1.20)
30      IN      PTR     db.meudominio.local.   ; (Para 192.168.1.30)
```


## Verificação da Configuração e Permissões

### Verifique a sintaxe dos arquivos de configuração

```
sudo named-checkconf
```

### sudo named-checkzone meudominio.local /etc/bind/zones/db.meudominio.local

```
sudo named-checkzone meudominio.local /etc/bind/zones/db.meudominio.local
```

### Verifique a sintaxe da zona inversa
```
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1
```

### Ajuste as permissões
```
sudo chown -R bind:bind /etc/bind/zones
sudo chmod 644 /etc/bind/zones/*
```

### Reiniciar e Habilitar o BIND9
```
sudo systemctl restart bind9
sudo systemctl enable bind9
```

### Configuração do Firewall
```
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
# Ou, de forma mais simples, se o UFW tiver o perfil do BIND9:
sudo ufw allow 'BIND'
```
