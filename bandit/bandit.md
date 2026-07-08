# Bandit - OverTheWire

---

## Lvl 0

Login inicial al servidor.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# estructura: ssh usuario@servidor -p puerto
```

---

## Lvl 1

El archivo se llama `-`, que es un símbolo lógico en bash. Se usa `./` para referirse a él como ruta relativa.

```bash
ls
cat ./-
```

---

## Lvl 2

El archivo tiene espacios en el nombre. Tab autocompleta y agrega los `\` necesarios.

```bash
cat ./spaces\ in\ this\ filename
```

---

## Lvl 3

El archivo está oculto. `ls -a` muestra archivos que empiezan con `.`.

```bash
ls -a
cat ./...Hiding-From-You
```

---

## Lvl 4

Varios archivos, se concatenan todos a la vez con wildcard.

```bash
cat ./*
```

---

## Lvl 5

Búsqueda con condiciones usando `find` en lugar de `ls`.

```bash
find . -size 1033c ! -executable
# -size 1033c   tamaño exacto de 1033 bytes
# ! -executable que no sea ejecutable
```

---

## Lvl 6

Búsqueda desde la raíz con condiciones de usuario y grupo. Los errores se descartan con `2>/dev/null`.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
# 0 = stdin, 1 = stdout, 2 = stderr
# > redirige, /dev/null es el basurero
```

---

## Lvl 7

Buscar una palabra dentro de un archivo con `grep`.

```bash
grep millionth data.txt
```

---

## Lvl 8

`uniq -u` solo funciona con líneas adyacentes, por eso se ordena primero con `sort`.

```bash
sort data.txt | uniq -u
```

---

## Lvl 9

`strings` extrae texto legible de un archivo binario. Luego `grep` filtra por el patrón.

```bash
strings data.txt | grep "=="
# alternativa menos limpia:
grep -a "==" data.txt
```

---

## Lvl 10

El archivo está codificado en base64. `-d` lo decodifica.

```bash
base64 -d data.txt
```

---

## Lvl 11

Cifrado ROT13 — cada letra fue movida 13 posiciones. `tr` traduce un conjunto de caracteres a otro.

```bash
cat data.txt | tr "A-Za-z" "N-ZA-Mn-za-m"
```

---

## Lvl 12

El archivo está en hexadecimal y fue comprimido varias veces. Se revierte el hex y se descomprime en loop hasta llegar al texto.

```bash
# crear carpeta temporal
mktemp -d
cd /tmp/tmp.XXXXXX

# copiar el archivo
cp /home/bandit12/data.txt .

# revertir hexdump a binario
xxd -r data.txt archivo

# loop: correr file primero, luego descomprimir según el tipo
file archivo

# gzip
mv archivo archivo.gz && gunzip archivo.gz

# bzip2
mv archivo archivo.bz2 && bunzip2 archivo.bz2

# tar
tar xf archivo

# repetir hasta que file diga ASCII text, luego:
cat archivo

# nota: /tmp es compartido entre usuarios, no guardar info sensible ahí
```

---

## Lvl 13

Las claves SSH privadas reemplazan contraseñas. Bandit bloquea conexiones SSH desde localhost, así que el archivo se descarga a la máquina local primero. SSH rechaza claves con permisos abiertos, `chmod 600` restringe el acceso solo al dueño. El puerto en `scp` es `-P` mayúscula, distinto a `ssh` que usa `-p` minúscula.

```bash
# descargar la clave privada a la máquina local
scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private ~/sshkey.private

# restringir permisos
chmod 600 ~/sshkey.private

# conectar a bandit14 con la clave
ssh bandit14@bandit.labs.overthewire.org -p 2220 -i ~/sshkey.private
```

# Nivel 14 → 15

## Objetivo
Enviar la contraseña del nivel actual al puerto 30000 en localhost para obtener la siguiente.

## Comando
```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

## Qué aprendí

### echo vs cat
`echo` imprime texto literal que tú escribes. Si haces `echo /ruta/archivo` manda la ruta como texto, no el contenido. `cat` abre el archivo y manda lo que hay adentro.

### nc (netcat)
Conexión directa a un puerto, como SSH pero sin cifrado ni autenticación. Todo lo que le mandas por stdin lo transmite, todo lo que recibe lo imprime.

### localhost
Siempre significa "esta misma máquina donde estoy parado". El puerto 30000 no estaba en mi PC personal, estaba en el mismo servidor de bandit. Mi PC no participó después de abrir la sesión SSH.

### Dónde guarda bandit las contraseñas
`/etc/bandit_pass/banditXX` — cada usuario solo puede leer la suya.

## Contraseña obtenida
8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo

## Nivel 15 → 16

**Concepto:** conexiones SSL/TLS con openssl s_client

**Por qué no nc:** nc hace conexiones en texto plano, el puerto 30001
requiere SSL/TLS.

**Por qué no cat:** cat lee archivos, la password es texto directo.

**Comando:**
echo "password" | openssl s_client -connect localhost:30001 -quiet

**Aprendido:**
- openssl es una navaja suiza de criptografía
- s_client es el subcomando para abrir conexiones SSL
- SSL/TLS cifra el canal, no el contenido específico
- self-signed certificate = el servidor se firmó su propio certificado

## Nivel 16 → 17

**Concepto:** escaneo de puertos, detección SSL, autenticación por llave privada SSH

**Proceso:**
1. nmap -p 31000-32000 -sV localhost → identificar puertos abiertos
2. De 5 puertos: 3 hacían echo, 2 hablaban SSL
3. Probar los 2 SSL con openssl s_client — uno devuelve la llave RSA privada
4. Guardar llave en archivo local con chmod 600
5. ssh bandit17@bandit.labs.overthewire.org -p 2220 -i llave.private

**Lección extra:** SSH desde localhost está bloqueado en el servidor
— la llave hay que usarla desde tu máquina local, no desde adentro.

**Comandos clave:**
nmap -p 31000-32000 -sV localhost
nmap -p 3100x-3100x-3100x-3100x -sV localhost
echo "password16" | openssl s_client -connect localhost:PUERTO -quiet
chmod 600 archivo.private
ssh bandit17@host -p 2220 -i /direcion/shh/privatekey

usuario:contraseña
ssh bandit16@bandit.labs.overthewire.org -p 2220

## Nivel 18 → 19

**Concepto:** SSH con comando directo, evasión de bashrc malicioso

**Situación:** El .bashrc de bandit18 tiene un exit al inicio —
cualquier sesión interactiva se cierra inmediatamente con "Byebye!".

**Solución:** SSH permite ejecutar un comando sin abrir sesión
interactiva, lo que evita que el bashrc cargue.

**Comando:**
ssh bandit18@bandit.labs.overthewire.org -p 2220 'cat readme'

**Aprendido:**
- ssh 'comando' ejecuta sin sesión interactiva y sin cargar bashrc
- Usuarios sin shell (/bin/false, /bin/nologin) usan este mismo
  mecanismo para backups y deploys automatizados
- En pentest: útil cuando el login interactivo está bloqueado
  pero SSH sigue abierto
