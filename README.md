# Red Hat linux
# Codigos basicos

Mudar o Host name:
```
$ sudo hostnamectl set-hostname <nome>.enta.pt
```

Atualizar o sistema:
```
$ sudo yum update
```

Onde tu estas:
```
$ pwd
``` 

Entrar como root:
```
$ sudo su -
```

Criar uma pasta:
```
$ mkdir <nome_da_pasta>
```

Remover uma pasta:
```
$ rmdir <nome_da_pasta>
```

Remover uma pasta se não estiver vazia:
```
$ rm -r <nome_da_pasta>
```

# Red Hat Linux Servidor
## 1.1 Instalar nginx

1. Instalar nginx:
```
# amazon -linux-extras install nginx1
```

2. Iniciar o serviso nginx:
```
# systemctl start nginx.service 
```

3. Iniciar o serviso todas as vezes que a maquina for ligada:

```
# systemctl enable --now nginx.service 
```

4. Verificar se a port 80 está aberta: 
```
$ netstat -tapn
```

## 1.2 Configurar nginx

1. Alterar a configuração do nginx:
```
$ cd /etc/nginx/
$ sudo nano nginx.conf
```

2. Não esquecer de reiniciar o serviso depois de alterar o nginx.conf:
```
$ sudo systemctl restart nginx.service 
```
## 1.3 configurar o HTML

1. Alterar o HTML: 
```
$ cd /usr/share/nginx/html
```

2. Editar os hosts para entrar nos sites:
```
$ sudo nano /etc/hosts

Exemplo:
  <Ip_privado_do_servidor> www.central.pt
  <Ip_privado_do_servidor> www.ocidental.pt
  <Ip_privado_do_servidor> www.oriental.pt
```


# Instalar VSFTPD 
## 2.1 Instalar FTP

1. Instalar ftp:
```
# yum install vsftpd
```

2. Ativar ftp:
```
# systemctl start vsftp
```
3. Iniciar o serviso todas as vezes que a maquina for ligada:
```
# systemctl enable --now vsftpd
```

## 2.2 Configurações do FTP 

1. Alterar a configuração do ftp:
```
# nano /etc/vsftpd/vsftpd.conf
  Descomentar e alterar:
  
    anonymous_enable=NO
    chroot_local_user=YES
    
  Adicionar:
  	
  	allow_writeable_chroot=YES
  	pasv_enable=YES
  	pasv_min_port=1024
  	pasv_max_port=1048
	write_enable=YES
    
```

2. Não esquecer de reiniciar o serviso depois de alterar o vsftpd.conf:
```
# systemctl restart vsftpd
```


## 2.3 Adicionar Utilizadores 

1. Adicionar os utilizadores:
```
# adduser <nome_do_user>
# passwd <nome_do_user>
```

2. Mudar o grupo de um ficheiro pu pasta:
```
# chgrp -R <nome_da_pasta/ficheiro> <nome_do_grupo> <localização_da_pasta/ficeiro>
Exemplo:
  chgrp -R central central /usr/share/nginx/central/
```

### Mudar permissões

r = ler

w = escrever

x = executar

utilizador:
```
chmod u+rwx <nome_do_ficheiro/pasta>
```

Groupo:
```
chmod g+rwx <nome_do_ficheiro/pasta>
```

Outros:
```
chmod o+rwx <nome_do_ficheiro/pasta>
```

montar a pasta:
```
mout --bind /var/www/central/ /home/central/
```
# Instalar BIND
## 3.1 Instalar BIND

1. Instalar bind:
```
# yum install bind bind-utils
```

2. Ativar o bind:
```
# systemctl enable --now named
```

3. Verificar se o bind está a funcionar:
```
# systemctl status named
```

## 3.2 Configurar o bind

1. Fazer um backup:
```
# cp /etc/named.conf /etc/named.conf.orig
```

2. editar o .conf:
2.1 Descometas o `listen-on`,`listen-on-v6` e `directory`. 

```
acl clients {192.0.2.0/24;};

options {
        listen-on port 53 { any; };

        listen-on-v6 port 53 { any; };

        directory       "/var/named";
```
2.2 Adicionar `allow-query`
```
    allow-query     { localhost; clients; };
    allow-recursion { localhost; clients; };
    recursion yes;
    allow-update { none; };
    allow-transfer { localhost; };
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
```

2.3 usar o pacote 
```
include "/etc/named.rfc1912.zones";
```

2.4 criar uma configuração de zona extra:
```
include "/etc/named/example.zones";
```
3. Criar uma zona `/etc/named/example.zones`:
```
//forward zone
zone "example.com" IN {
        type master;
        file "example.com.zone";

};

//backward zone
zone "2.0.192.in-addr.arpa" IN {
        type master;
        file "example.com.rzone";

};
```
* type: It defines the zone’s role of the server.
* master: It is an authoritative server and maintains the master copy of the zone data.
* file: It specifies the zone’s database file.

4. ir para o diretorio do DNS `/var/named/`:
```
# cd /var/named/
# ls

data    dynamic  named.ca  named.empty    named.localhost    named.loopback  slaves
```

5. criar um ficheiro com o nome da tua forward zone parameters `/var/named/example.com.zone`: 
```
$TTL    86400
@	IN	SOA	example.com.	root (
		42         ; serial
		3H         ; refresh
		15M        ; retry
		1W         ; expiry
		1D )       ; minimum

	IN	NS	ns.example.com.

ns              IN A            192.0.2.1
station1        IN A            192.0.2.101
station2        IN A            192.0.2.102
station3        IN A            192.0.2.103
```
6. criar um ficheiro com o nome da tua reverse zone parameters `/var/named/example.com.rzone`:
```
$TTL    86400
@	IN	SOA	example.com.	root.example.com. (
		1997022700 ; serial
		28800      ; refresh
		14400      ; retry
		3600000    ; expire
		86400 )    ; minimum

	IN	NS	ns.example.com.

101     IN      PTR     station1.example.com.
102     IN      PTR     station2.example.com.
103     IN      PTR     station3.example.com.
```

7. definir as permissoes da zona:
```
# chown root:named /var/named/example.com.zone /var/named/example.com.rzone
# chmod 640 /var/named/example.com.zone /var/named/example.com.rzone
```

8. reiniciar o BIND:
```
# systemctl restart named
```

2.5 Verificação





# Red Hat Linux Cliente
## Instalar o desktop

```
$ sudo su -
```
```
# amazon-linux-extras install epel
```
```
# yum groupinstall "MATE Desktop"
```
```
# yum install xrdp chromium filezilla
```
```
# systemctl enable --now xrdp
```
This will make all users to have the graphic interface:
```  
# bash -c 'echo PREFERRED=/usr/bin/mate-session > /etc/sysconfig/desktop'
```
```
# adduser <nome_do_utilizador>
# passwd <nome_do_utilizador>
```
```
# echo "/usr/bin/mate-session" > ~/.Xclients && chmod +x ~/.Xclients
```
```
# reboot
```
Depois de reiniciar
```
$ sudo firewall-cmd --add-port=3389/tcp --permanent
```
```
$ sudo firewall-cmd --reload
```

