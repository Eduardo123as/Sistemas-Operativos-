# CLASE: SISTEMA DE ARCHIVOS UNIX Y BASH
## Teoría + Práctica + Ejercicios de Entrega

---

# PARTE 1: EL SISTEMA DE ARCHIVOS (FILE SYSTEM)

## 1. CONCEPTOS FUNDAMENTALES

### 1.1 ¿Qué es un Sistema de Archivos?

Un **Sistema de Archivos (FS)** es una estructura lógica que organiza y almacena datos en el disco de forma jerárquica.

**Analogía del Mundo Real:**
```
Computadora     ←→  Biblioteca
├─ File System       ├─ Sistema de Catalogación
├─ Archivos          ├─ Libros
├─ Directorios       ├─ Estanterías
├─ Permisos          ├─ Restricciones de acceso
└─ Estructura        └─ Organización por categorías
```

### 1.2 Diferencia: Archivo vs Directorio

```
ARCHIVO (Archivo):
├─ Contiene datos (texto, código, imágenes, etc.)
├─ Tiene tamaño específico
├─ Tiene permisos
├─ Tiene timestamp (creación, modificación)
└─ Ejemplo: documento.txt (4KB)

DIRECTORIO (Folder/Carpeta):
├─ Contiene otros archivos y directorios
├─ Es un "contenedor" lógico
├─ También tiene permisos
├─ Estructura jerárquica
└─ Ejemplo: /home/usuario/documentos/
```

### 1.3 Principio UNIX: "Todo es un Archivo"

En UNIX/Linux, CASI TODO se representa como un archivo:

```
ARCHIVOS ORDINARIOS:
├─ documento.txt  (datos de texto)
├─ foto.jpg       (datos binarios)
└─ script.sh      (script ejecutable)

DIRECTORIOS (son "archivos especiales"):
├─ /home/usuario/
├─ /etc/
└─ /var/

ARCHIVOS ESPECIALES (devices):
├─ /dev/sda       (disco duro entero)
├─ /dev/sda1      (partición)
├─ /dev/tty0      (terminal virtual)
├─ /dev/null      (dispositivo "nulo" - descarta todo)
└─ /dev/random    (generador de números aleatorios)

SOCKETS (comunicación entre procesos):
├─ /var/run/docker.sock
├─ /tmp/mysql.sock
└─ Pipes: |

LINKS SIMBÓLICOS (atajos):
├─ /bin/ls → /usr/bin/ls

ARCHIVOS DE SISTEMA VIRTUAL:
├─ /proc/cpuinfo  (información de CPU, read-only)
├─ /sys/devices/  (árbol de dispositivos)
└─ /proc/[PID]/   (información de procesos)
```

**Implicación práctica:**
```bash
# Puedo tratar un dispositivo como un archivo
$ cat /dev/urandom | head -c 16 | od -An -tx1
 af 3c e4 7b 2d 9a f1 8c 4e 6b a3 72 51 c9 d0 e8

# Puedo mandar la salida de un comando a /dev/null (tirarla)
$ echo "esto desaparece" > /dev/null

# Puedo monitorear un log en tiempo real
$ cat /var/log/syslog
```

---

## 2. ESTRUCTURA JERÁRQUICA: EL ÁRBOL DEL SISTEMA

### 2.1 Vista General

```
/  (RAÍZ - root filesystem)
│
├── bin/           Binarios (ejecutables) esenciales
│   ├── ls         (listar archivos)
│   ├── cat        (mostrar contenido)
│   ├── mkdir      (crear directorio)
│   └── chmod      (cambiar permisos)
│
├── sbin/          System binaries (solo root)
│   ├── ifconfig   (configurar red)
│   ├── iptables   (firewall)
│   └── fdisk      (particiones)
│
├── etc/           Configuración global del sistema
│   ├── passwd     (usuarios registrados)
│   ├── shadow     (contraseñas encriptadas)
│   ├── fstab      (sistemas de archivos a montar)
│   ├── hosts      (mapa IP-dominio local)
│   └── ssh/       (configuración SSH)
│
├── home/          Directorios personales de usuarios
│   ├── juan/      (home del usuario juan)
│   │   ├── .bashrc        (configuración bash)
│   │   ├── .ssh/          (claves SSH)
│   │   ├── Documentos/
│   │   ├── Descargas/
│   │   └── .config/       (archivos ocultos)
│   │
│   └── maria/
│       ├── Documentos/
│       └── .bashrc
│
├── root/          Home del usuario root (admin)
│   └── (similar a /home/[user]/)
│
├── var/           Datos variables (logs, caché, etc.)
│   ├── log/       (archivos de log del sistema)
│   │   ├── syslog       (eventos generales)
│   │   ├── auth.log     (intentos de login)
│   │   ├── apache2/     (logs de Apache)
│   │   └── nginx/       (logs de Nginx)
│   │
│   ├── cache/     (archivos de caché)
│   │   ├── apt/         (caché de paquetes)
│   │   └── nginx/
│   │
│   ├── spool/     (colas de impresión, correos)
│   ├── tmp/       (archivos temporales)
│   └── www/       (archivos web públicos, Apache)
│
├── tmp/           Temporal (accesible a todos)
│   ├── archivo.tmp
│   └── socket.sock
│
├── usr/           Aplicaciones de usuario
│   ├── bin/       (ejecutables no-esenciales)
│   │   ├── python
│   │   ├── node
│   │   ├── vim
│   │   └── gimp
│   │
│   ├── local/     (software instalado localmente)
│   │   ├── bin/   (binarios compilados localmente)
│   │   ├── lib/   (librerías locales)
│   │   └── share/
│   │
│   ├── lib/       (librerías de sistema)
│   │   ├── libc.so.6     (C runtime)
│   │   ├── libssl.so.1.1  (OpenSSL)
│   │   └── x86_64-linux-gnu/
│   │
│   ├── share/     (datos compartidos, no ejecutables)
│   │   ├── man/          (manuales)
│   │   ├── doc/          (documentación)
│   │   ├── icons/        (iconos)
│   │   └── fonts/        (fuentes tipográficas)
│   │
│   └── src/       (código fuente)
│       ├── linux/        (kernel)
│       └── firefox/
│
├── lib/           Librerías esenciales (linked to /usr/lib)
│   └── libc.so.6
│
├── lib64/         Librerías de 64-bit
│
├── opt/           Aplicaciones opcionales de terceros
│   ├── google-chrome/
│   ├── discord/
│   └── slack/
│
├── media/         Puntos de montaje para medios removibles
│   ├── cdrom      (montaje de CD)
│   ├── usb_drive  (montaje de USB)
│   └── external_hd
│
├── mnt/           Puntos de montaje temporales
│   ├── windows    (partición Windows montada)
│   ├── backup     (disco externo)
│   └── nfs_share  (recurso de red)
│
├── dev/           Archivos especiales de dispositivos
│   ├── sda        (primer disco duro)
│   ├── sda1       (partición 1)
│   ├── sdb        (segundo disco duro)
│   ├── tty0       (terminal 0)
│   ├── null       (descarta todo)
│   ├── zero       (generador de ceros)
│   ├── urandom    (números aleatorios)
│   ├── pts/       (pseudoterminales)
│   │   ├── 0     (terminal SSH #0)
│   │   ├── 1     (terminal SSH #1)
│   │   └── ...
│   └── loop0      (dispositivo loop - montar ISO)
│
├── proc/          Sistema de archivos virtual (procesos)
│   ├── cpuinfo    (información de CPU)
│   ├── meminfo    (información de memoria)
│   ├── uptime     (tiempo desde que arrancó)
│   ├── 1/         (proceso PID 1)
│   │   ├── cmdline
│   │   ├── environ
│   │   ├── fd/
│   │   └── maps
│   ├── 100/       (proceso PID 100)
│   │   └── ...
│   └── self/      (enlace al proceso actual)
│
├── sys/           Sistema de archivos virtual (dispositivos)
│   ├── devices/
│   ├── bus/       (información de buses)
│   ├── kernel/    (parámetros del kernel)
│   └── power/     (control de energía)
│
├── run/           Información de runtime (reemplazo de /var/run)
│   ├── user/      (archivos temporales de usuario)
│   └── lock/      (archivos de bloqueo)
│
└── boot/          Archivos de arranque
    ├── vmlinuz    (kernel comprimido)
    ├── initrd.img (ramdisk inicial)
    └── grub/      (configuración del bootloader)
```

### 2.2 Explicación de Directorios Clave

**Para DESARROLLADORES:**
```
/usr/local/   ← Aquí instalo herramientas que compilo
/opt/         ← Aplicaciones comerciales
/home/[user]/ ← Mi espacio personal
/tmp/         ← Archivos temporales durante sesión
```

**Para ADMINISTRADORES:**
```
/etc/         ← Configuración global
/var/log/     ← Problemas y errores
/var/spool/   ← Colas de impresión, correos
/dev/         ← Hardware como archivos
```

**Para EL KERNEL:**
```
/proc/        ← Información de procesos corriendo AHORA
/sys/         ← Árbol de dispositivos
/boot/        ← Necesario para arrancar
```

---

## 3. RUTAS: ABSOLUTA vs RELATIVA

### 3.1 Rutas Absolutas

Una **ruta absoluta** comienza con `/` (raíz).

```bash
# Ruta absoluta (siempre funciona desde cualquier lado)
/home/juan/documentos/tesis.pdf

# Desglose:
/           ← Raíz del filesystem
home/       ← Directorio dentro de raíz
juan/       ← Directorio dentro de home
documentos/ ← Directorio dentro de juan
tesis.pdf   ← Archivo dentro de documentos
```

**Características:**
- Funciona desde cualquier directorio actual
- Inequívoca (no hay confusión)
- Más larga de escribir

```bash
$ pwd
/home/juan/Descargas

$ ls /home/juan/documentos/
tesis.pdf  libro.txt  notas.txt

$ cat /home/juan/.bashrc
# Funciona aunque esté en /var/log/
```

### 3.2 Rutas Relativas

Una **ruta relativa** es relativa al directorio actual.

```bash
# Dónde estoy ahora
$ pwd
/home/juan

# Ruta relativa: documentos/tesis.pdf
# Significa: documentos/ dentro de donde estoy AHORA (/home/juan)

$ ls documentos/
tesis.pdf  libro.txt  notas.txt

# Sin la barra inicial, es relativa
```

**Caracteres especiales en rutas relativas:**

```bash
.          ← Directorio actual
..         ← Directorio padre
~          ← Home del usuario actual
-          ← Directorio anterior

# Ejemplos:
$ pwd
/home/juan/Documentos

# Estos son equivalentes:
$ cat ./tesis.pdf           # . = directorio actual
$ cat tesis.pdf             # implícito

# Subir un nivel
$ cat ../Descargas/archivo.txt

# Ir al padre del padre
$ cd ../../

# Ir al home
$ cd ~
$ cd ~/documentos           # ~/documentos = /home/juan/documentos
$ cd                        # sin argumentos, va a ~

# Ir a directorio anterior
$ pwd
/var/log
$ cd /home/juan/Documentos
$ cd -    # Vuelve a /var/log
```

### 3.3 Resumen: Cuándo Usar Cada Una

```
ABSOLUTA (/home/juan/...):
├─ Scripts que otros ejecutarán
├─ Comandos en crontab
├─ Referencias en documentación
└─ Cuando necesitas ser inequívoco

RELATIVA (./archivo, ../config):
├─ Dentro de un proyecto
├─ Scripts locales
├─ Navegación rápida en terminal
└─ Cuando trabajas dentro de un directorio
```

---

## 4. PERMISOS: EL MODELO UGO

### 4.1 Estructura de Permisos

Cada archivo/directorio tiene 9 bits de permisos, divididos en 3 grupos:

```
-rwxrwxrwx
│ │ │ │ │ │ │ │ │
│ │ │ │ │ │ │ │ └─ Others: execute (x)
│ │ │ │ │ │ │ └─── Others: write (w)
│ │ │ │ │ │ └───── Others: read (r)
│ │ │ │ │ └─────── Group: execute (x)
│ │ │ │ └───────── Group: write (w)
│ │ │ └─────────── Group: read (r)
│ │ └───────────── User: execute (x)
│ └─────────────── User: write (w)
└───────────────── User: read (r)

Primer carácter:
- = archivo ordinario
d = directorio
l = link simbólico
c = character device
b = block device
s = socket
p = pipe
```

### 4.2 Significado de Permisos

```
PARA ARCHIVOS ORDINARIOS:

r (Read = 4):
├─ Puedo LEER el contenido
├─ $ cat archivo.txt  ← Necesito permiso r
└─ Equivalente: Abrir un libro en la biblioteca

w (Write = 2):
├─ Puedo MODIFICAR/BORRAR el archivo
├─ $ echo "nuevo" > archivo.txt  ← Necesito permiso w
├─ $ rm archivo.txt              ← Necesito permiso w
└─ Equivalente: Editar un libro con lápiz

x (Execute = 1):
├─ Puedo EJECUTAR el archivo como programa
├─ $ ./archivo.sh              ← Necesito permiso x
├─ $ python script.py          ← Necesito x en script.py
└─ El archivo NO tiene que ser binario compilado
    (puede ser un script en cualquier lenguaje)


PARA DIRECTORIOS:

r (Read = 4):
├─ Puedo LISTAR el contenido
├─ $ ls directorio/         ← Necesito r en el directorio
├─ $ ls -la directorio/     ← Necesito r en el directorio
└─ Equivalente: Ver qué libros hay en una estantería

w (Write = 2):
├─ Puedo CREAR, ELIMINAR, RENOMBRAR archivos dentro
├─ $ touch directorio/nuevo.txt     ← Necesito w en directorio
├─ $ rm directorio/archivo.txt      ← Necesito w en directorio
├─ $ mv directorio/a.txt b.txt      ← Necesito w en directorio
└─ Equivalencia: Agregar/quitar libros de la estantería

x (Execute = 1):
├─ Puedo ACCEDER/ENTRAR al directorio
├─ $ cd directorio/        ← Necesito x en el directorio
├─ $ cat directorio/file.txt ← Necesito x en directorio (para atravesarlo)
└─ Sin x, no puedo hacer cd, incluso si tengo r

IMPORTANTE: Para acceder a un archivo dentro de un directorio,
            necesito PERMISOS x EN TODOS LOS DIRECTORIOS PADRE
```

### 4.3 Notación Simbólica vs Octal

**Simbólica:**
```bash
# u=usuario, g=grupo, o=otros
# +=agregar, -=remover, ==establecer exacto

$ chmod u+x script.sh        # Agregar ejecución al usuario
$ chmod g-w documento.txt    # Remover escritura del grupo
$ chmod o-r documento.txt    # Remover lectura de otros
$ chmod a+r documento.txt    # Agregar lectura a todos
$ chmod 644 documento.txt    # r--r--r-- (lectura para todos, escritura solo user)
```

**Octal:**
```
r (read)   = 4
w (write)  = 2
x (execute) = 1

rwx = 4+2+1 = 7
rw- = 4+2   = 6
r-x = 4+1   = 5
r-- = 4     = 4
-wx = 2+1   = 3
-w- = 2     = 2
--x = 1     = 1
--- = 0     = 0

EJEMPLOS:

755 = rwxr-xr-x
    = Usuario: rwx (7)
    = Grupo:   r-x (5)
    = Otros:   r-x (5)
    = Típico para directorios

644 = rw-r--r--
    = Usuario: rw- (6)
    = Grupo:   r-- (4)
    = Otros:   r-- (4)
    = Típico para archivos

777 = rwxrwxrwx
    = Todos pueden todo
    = ¡NUNCA HAGAS ESTO EN PRODUCCIÓN!

700 = rwx------
    = Solo el usuario
    = Típico para home de usuario (/home/juan)
    = Para archivos sensibles

600 = rw-------
    = Solo el usuario puede leer/escribir
    = Típico para claves SSH privadas
```

### 4.4 Propietario y Grupo

```bash
$ ls -l documento.txt
-rw-r--r-- 1 juan developers 4096 Jun 15 10:30 documento.txt
│          │ │    │          │
│          │ │    │          └─ Tamaño
│          │ │    └──────────── Grupo propietario
│          │ └───────────────── Usuario propietario
│          └─────────────────── Número de links
└──────────────────────────────── Permisos

# Cambiar propietario
$ sudo chown juan documento.txt

# Cambiar propietario y grupo
$ sudo chown juan:developers documento.txt

# Cambiar solo grupo
$ sudo chgrp developers documento.txt

# Cambiar permisos
$ chmod 644 documento.txt    # Octal
$ chmod u+x documento.txt    # Simbólico
```

### 4.5 SUID, SGID, Sticky Bit

Tres bits especiales adicionales:

```
SUID (Set User ID = 4000):
├─ Cuando ejecutas el archivo
├─ Se ejecuta como el PROPIETARIO, no como tú
├─ Ejemplo: /usr/bin/sudo
│   $ ls -l /usr/bin/sudo
│   -rwsr-xr-x 1 root root  ...
│            ↑
│            La "s" indica SUID
├─ Cuando haces $ sudo ..., se ejecuta como root
└─ PELIGROSO si el propietario es root

SGID (Set Group ID = 2000):
├─ Cuando ejecutas el archivo
├─ Se ejecuta como el GRUPO, no como tu grupo
├─ En directorios: archivos nuevos heredan el grupo
├─ Ejemplo: /usr/bin/mail
└─ Menos común que SUID

STICKY BIT (1000):
├─ En directorios: solo el propietario puede borrar archivos
├─ Ejemplo: /tmp (rwxrwxrwt)
│   $ ls -ld /tmp
│   drwxrwxrwt 1 root root ... /tmp
│               ↑
│               La "t" indica sticky bit
├─ Aunque todos pueden crear en /tmp
├─ Solo puedes borrar lo QUE TÚ creaste
└─ Impide que Juan borre lo de María

NOTACIÓN OCTAL COMPLETA:

4755 = SUID + rwxr-xr-x
2755 = SGID + rwxr-xr-x
1755 = Sticky + rwxr-xr-x
7755 = Todos
```

---

# PARTE 2: BASH (Bourne Again Shell)

## 5. ¿QUÉ ES UN SHELL?

### 5.1 Definición

Un **shell** es un programa que:
- Lee comandos del usuario (o script)
- Interpreta qué significan
- Los ejecuta usando el kernel
- Muestra resultados

**No es el SO, es una APLICACIÓN que HABLA CON el SO.**

```
Usuario
  ↓ (escribe comando)
BASH (shell)
  ↓ (interpreta)
Kernel (SO)
  ↓ (ejecuta)
Hardware
  ↑ (resultado)
Kernel (SO)
  ↑
BASH
  ↑
Usuario (ve resultado)
```

### 5.2 Historia del Shell

```
1971: Thompson Shell (sh)
  └─ El primer shell de Unix
  └─ Muy simple

1977: Bourne Shell (sh)
  └─ Mejoras significativas
  └─ Estándar de POSIX
  └─ Aún existe como /bin/sh

1980: C Shell (csh)
  └─ Sintaxis similar a C
  └─ Popular en Berkeley Unix

1983: Korn Shell (ksh)
  └─ Extensiones del Bourne
  └─ Scripting avanzado

1987: Bash (Bourne Again Shell)
  ├─ GNU Bourne Again SHell
  ├─ Combina lo mejor de sh, csh, ksh
  ├─ Es el shell estándar en Linux
  ├─ Shell por defecto en macOS (antes de Catalina)
  └─ ¡EL QUE VAMOS A APRENDER!

1989: Z Shell (zsh)
  ├─ Muy personalizable
  ├─ Shell por defecto en macOS (desde Catalina)
  └─ Más moderno que bash

1990: Fish (Friendly Interactive Shell)
  ├─ Enfocado en usabilidad
  ├─ Autocompletado inteligente
  └─ Sintaxis diferente a sh
```

---

## 6. BASH: CARACTERÍSTICAS Y TIPOS

### 6.1 Tipos de Shells / Modos de Ejecución

```
INTERACTIVE vs NON-INTERACTIVE:

┌─────────────────────────────────────────────────────┐
│ INTERACTIVE SHELL                                   │
├─────────────────────────────────────────────────────┤
│ - Conectado a una terminal                          │
│ - Lee comandos del usuario                          │
│ - Muestra prompts ($ )                              │
│ - Ejecuta línea por línea                           │
│                                                     │
│ $ bash                 ← Inicia bash interactivo    │
│ $ chmod +x script.sh   ← Ejecutas un comando        │
│ $ ls                   ← Y ves resultado            │
│                                                     │
│ Cómo abrir:                                         │
│ ├─ Terminal en tu escritorio                        │
│ ├─ SSH a un servidor                                │
│ ├─ Subshell: $ bash (dentro de otra shell)          │
│ └─ Script con #!/bin/bash + chmod +x               │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ NON-INTERACTIVE SHELL (Script Mode)                 │
├─────────────────────────────────────────────────────┤
│ - No está conectado a terminal                      │
│ - Lee comandos de un archivo (script)               │
│ - NO muestra prompts                                │
│ - Ejecuta el script completo                        │
│ - Se cierra al terminar                             │
│                                                     │
│ $ bash script.sh       ← Ejecuta script             │
│ $ ./script.sh          ← Si tiene #!/bin/bash       │
│ $ source script.sh     ← En la shell actual         │
│                                                     │
│ Casos de uso:                                       │
│ ├─ Scripts de automatización                        │
│ ├─ Cron jobs                                        │
│ ├─ CI/CD pipelines                                  │
│ └─ Backups automáticos                              │
└─────────────────────────────────────────────────────┘

LOGIN vs NON-LOGIN SHELL:

┌─────────────────────────────────────────────────────┐
│ LOGIN SHELL                                         │
├─────────────────────────────────────────────────────┤
│ - Accedes mediante login (username + password)      │
│ - Lee: /etc/profile, ~/.profile, ~/.bash_profile    │
│ - Carga variables de entorno del sistema            │
│ - ¿Cuándo ocurre?                                   │
│   ├─ SSH a un servidor: ssh user@host               │
│   ├─ TTY: Ctrl+Alt+F2 (consola pura)                │
│   ├─ Su: sudo -i (login de root)                    │
│   └─ GDM/LightDM: Login gráfico                     │
│                                                     │
│ Archivos de configuración (en orden):               │
│ 1. /etc/profile       (global)                      │
│ 2. ~/.bash_profile    (usuario, si existe)          │
│ 3. ~/.bashrc          (usuarioNO se ejecuta en     │
│                        login shells de nuevo)       │
│                                                     │
│ Típicamente ~/.bash_profile contiene:               │
│ if [ -f ~/.bashrc ]; then                           │
│     source ~/.bashrc                                │
│ fi                                                  │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ NON-LOGIN SHELL                                     │
├─────────────────────────────────────────────────────┤
│ - Abres una terminal dentro del desktop             │
│ - NO es un login (ya estás logueado en el sistema)  │
│ - Lee: ~/.bashrc solamente                          │
│ - Hereda variables del shell padre                  │
│ - ¿Cuándo ocurre?                                   │
│   ├─ Terminales en GNOME/KDE/etc                    │
│   ├─ Subshells: $ bash                              │
│   ├─ Scripts: $ bash script.sh                      │
│   └─ VSCode/Sublime terminals                       │
│                                                     │
│ Archivos de configuración:                          │
│ - SOLO ~/.bashrc                                    │
│ - /etc/bash.bashrc (en algunas distros)             │
└─────────────────────────────────────────────────────┘

DIAGRAMA VISUAL:

                 ¿Es login?
                    /\
                   /  \
                 SÍ    NO
                /        \
         ┌─────┐      ┌────────┐
         │LOGIN│      │NON-LOGIN
         └──┬──┘      └────┬───┘
            │              │
        Lee:           Lee:
        /etc/prof    ~/.bashrc
        ~/.bash_prof /etc/bash.bashrc
        ~/.bashrc
            │              │
            │         ┌─────┴──────┐
            │         │            │
           Shell    Terminal   Subshell  Script
           activo   gráfica    $ bash   $ ./file.sh
```

### 6.2 Archivos de Configuración de Bash

```
~/.bashrc
═════════════════════════════════════════════════════════
CUANDO SE CARGA:
├─ Cada vez que abres una terminal (non-login)
├─ Dentro de scripts: source ~/.bashrc
└─ Automáticamente en shells interactivos

QUÉ CONTIENE:
├─ Aliases (atajos)
├─ Funciones propias
├─ Prompt personalizado (PS1)
├─ Historial de comandos
├─ Autocompletado
├─ Variables locales
└─ Configuración específica del usuario

EJEMPLO:
─────────────────────────────────────────────────────────
# ~/.bashrc

# Evitar inicialización recursiva
if [[ $- != *i* ]] ; then
    return
fi

# Aliases
alias ll='ls -lh'
alias la='ls -lha'
alias grep='grep --color=auto'

# Funciones
mcd() {
    mkdir -p "$1"
    cd "$1"
}

# Prompt personalizado
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
# Muestra: user@host:/ruta/actual$

# Historial
HISTFILE=~/.bash_history
HISTSIZE=1000
HISTFILESIZE=2000

# Autocompletado
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi
─────────────────────────────────────────────────────────


~/.bash_profile
═════════════════════════════════════════════════════════
CUANDO SE CARGA:
├─ Solo en login shells
├─ Típicamente NO se edita en moderno Bash
├─ Ahora suele solo hacer source de ~/.bashrc
└─ Principalmente para variables de entorno

EJEMPLO:
─────────────────────────────────────────────────────────
# ~/.bash_profile

# source bashrc si existe
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi

# Agregar a PATH
export PATH="$PATH:$HOME/.local/bin"
export PATH="$PATH:$HOME/.cargo/bin"

# Variables de entorno
export LANG=en_US.UTF-8
export EDITOR=vim
export PAGER=less
─────────────────────────────────────────────────────────


/etc/profile
═════════════════════════════════════════════════════════
CUANDO SE CARGA:
├─ TODOS los login shells lo leen
├─ Configuración GLOBAL del sistema
├─ Requiere permisos de root para editar

CONTIENE:
├─ PATH del sistema completo
├─ Variables de entorno globales
├─ Configuración de umask (permisos por defecto)
├─ Carga de módulos/scripts globales

TÍPICAMENTE:
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
export LANG=C.UTF-8
umask 022
─────────────────────────────────────────────────────────


/etc/bash.bashrc
═════════════════════════════════════════════════════════
CUANDO SE CARGA:
├─ En algunos shells (Debian/Ubuntu), antes que ~/.bashrc
├─ Configuración global de TODOS los shells no-login

CONTIENE:
├─ Aliases globales
├─ Funciones del sistema
├─ Prompt del sistema por defecto
└─ Configuración de paquetes
─────────────────────────────────────────────────────────

ORDEN DE CARGA:

LOGIN SHELL (ej: SSH):
1. /etc/profile
2. ~/.bash_profile (típicamente source ~/.bashrc)
   ↓
   ~/.bashrc
   /etc/bash.bashrc (en Debian)
   
NON-LOGIN SHELL (ej: Terminal gráfica):
1. ~/.bashrc
   (opcionalmente: /etc/bash.bashrc)

SCRIPT BASH (non-interactive):
- NO carga ninguno automáticamente
- Debes hacer: source ~/.bashrc manualmente si lo necesitas
- Cada script debería ser independiente
```

### 6.3 Comparación: Bash vs Otros Shells

```
┌─────────┬──────────┬──────────┬─────────┬──────────┐
│ Feature │ bash     │ zsh      │ fish    │ ksh      │
├─────────┼──────────┼──────────┼─────────┼──────────┤
│Sintaxis │ POSIX    │ POSIX++  │ NO-POSIX│ POSIX    │
│Std.Linux│ Defecto  │ Alternati│Alternati│ Raro     │
│Customiza│ Media    │ Muy Alta │ Alta    │ Media    │
│Speed    │ Rápido   │ Rápido   │ Lento   │ Rápido   │
│Portabil.│ Excelent │ Bueno    │ Pobre   │ Bueno    │
│Scripts  │ Sí (best)│ Sí       │ No      │ Sí       │
│Learning │ Fácil    │ Medio    │ Fácil   │ Medio    │
│Autocomp.│ Básico   │ Avanzado │Avanzado │ Básico   │
└─────────┴──────────┴──────────┴─────────┴──────────┘

RECOMENDACIÓN:
├─ Para scripting serio: bash (POSIX estándar)
├─ Para usuario interactivo: zsh (más features)
├─ Para usuarios novatos: fish (muy amigable)
└─ Para producción/servers: bash (máxima compatibilidad)

NUESTRA CLASE USA: BASH (estándar de facto)
```

---

# PARTE 3: PRÁCTICA GUIADA CON BASH

## 7. COMANDOS ESENCIALES DEL FILESYSTEM

### 7.1 Navegar el Sistema

```bash
# pwd: Print Working Directory (¿Dónde estoy?)
$ pwd
/home/juan/Documentos

# ls: Listar archivos
$ ls                    # Listado simple
$ ls -l                 # Listado largo (detalles)
$ ls -la                # Incluyendo archivos ocultos
$ ls -lh                # Tamaños legibles (1.2K, 3.4M)
$ ls -lS                # Ordenado por tamaño
$ ls -lt                # Ordenado por tiempo
$ ls -R                 # Recursivo (subdirectorios)
$ ls /tmp               # Especificando ruta

# cd: Cambiar directorio
$ cd /home/juan         # Ruta absoluta
$ cd documentos         # Ruta relativa
$ cd ..                 # Padre
$ cd ../..              # Abuelo
$ cd ~                  # Home
$ cd ~/documentos       # Home + ruta
$ cd -                  # Anterior

# tree: Ver estructura (mejor que ls -R)
$ tree                  # Árbol bonito
$ tree -L 2             # Máximo 2 niveles
$ tree -I '*.o|*.a'     # Excluir .o y .a
```

### 7.2 Crear y Eliminar

```bash
# mkdir: Crear directorio
$ mkdir midir           # Crear un directorio
$ mkdir -p dir1/dir2/dir3  # Crear padres si no existen
$ mkdir -v dir1 dir2    # Verbose (muestra lo que hace)

# touch: Crear archivo vacío / actualizar timestamp
$ touch archivo.txt     # Crear vacío
$ touch archivo1 archivo2  # Múltiples
$ touch -d "2024-01-01" archivo.txt  # Timestamp específico

# rm: Eliminar archivo
$ rm archivo.txt        # Borrar
$ rm -i archivo.txt     # Pedir confirmación
$ rm -f archivo.txt     # Forzar (sin confirmar)
$ rm *.txt              # Patrón glob (todos los .txt)

# rmdir: Eliminar directorio vacío
$ rmdir directorio      # Solo si está vacío
$ rm -r directorio      # ¡PELIGROSO! Borra TODO recursivo
$ rm -rf directorio     # ¡¡EXTREMADAMENTE PELIGROSO!!

# cp: Copiar
$ cp origen.txt destino.txt     # Archivo a archivo
$ cp archivo.txt ~/backup/      # Archivo a directorio
$ cp -r directorio/ backup/     # Recursivo (directorios)
$ cp -v archivo.txt backup/     # Verbose
$ cp -i archivo.txt backup/     # Preguntar si existe

# mv: Mover / Renombrar
$ mv viejo.txt nuevo.txt        # Renombrar
$ mv archivo.txt ~/documentos/  # Mover a directorio
$ mv -i archivo.txt backup/     # Preguntar si existe

# ln: Crear links
$ ln -s /usr/bin/python python  # Link simbólico (atajo)
$ ln archivo.txt enlace         # Link duro (copia de inode)
```

### 7.3 Ver Contenido de Archivos

```bash
# cat: Mostrar contenido completo
$ cat archivo.txt           # Mostrar
$ cat archivo1.txt archivo2.txt  # Varios archivos
$ cat archivo.txt | head    # Primeras 10 líneas

# less: Paginador (para archivos grandes)
$ less archivo.txt
# Dentro de less:
#   j/k o flechas: navegar línea por línea
#   space: página siguiente
#   b: página anterior
#   /palabra: buscar
#   q: salir

# head: Primeras líneas
$ head archivo.txt          # Primeras 10 líneas
$ head -20 archivo.txt      # Primeras 20
$ head -5 archivo.txt       # Primeras 5

# tail: Últimas líneas
$ tail archivo.txt          # Últimas 10
$ tail -20 archivo.txt      # Últimas 20
$ tail -f archivo.txt       # Follow (monitorear en vivo)
# Útil para logs: $ tail -f /var/log/syslog

# wc: Word Count
$ wc archivo.txt            # Líneas, palabras, bytes
$ wc -l archivo.txt         # Solo líneas
$ wc -w archivo.txt         # Solo palabras
$ wc -c archivo.txt         # Solo bytes
```

### 7.4 Buscar y Filtrar

```bash
# find: Búsqueda potente
$ find . -name "*.txt"              # Por nombre
$ find . -type f -name "*.py"       # Solo archivos
$ find . -type d -name "temp*"      # Solo directorios
$ find . -size +10M                 # Mayor a 10MB
$ find . -size -1M                  # Menor a 1MB
$ find . -mtime -7                  # Modificado últimos 7 días
$ find . -user juan                 # Propiedad de juan
$ find . -perm 644                  # Con permisos específicos
$ find . -exec rm {} \;             # Ejecutar comando en cada resultado

# grep: Buscar patrón en archivos
$ grep "error" archivo.txt          # Líneas con "error"
$ grep -i "ERROR" archivo.txt       # Case-insensitive
$ grep -n "error" archivo.txt       # Con número de línea
$ grep -c "error" archivo.txt       # Contar coincidencias
$ grep "^error" archivo.txt         # Al inicio de línea
$ grep "error$" archivo.txt         # Al final de línea
$ grep -r "error" ./                # Recursivo en directorio
$ grep -v "error" archivo.txt       # Invertir (sin "error")
$ grep -E "error|warning" archivo.txt  # Múltiples patrones (regex)

# locate: Búsqueda rápida (indexada)
$ locate archivo.txt                # Busca en base de datos
$ locate *.pdf                      # Todos los PDF
$ updatedb                          # Actualizar índice (como root)

# which: Dónde está un comando
$ which ls                          # /usr/bin/ls
$ which python                      # /usr/bin/python3
$ which java                        # /usr/bin/java

# whereis: Busca binarios, fuentes, manuales
$ whereis ls                        # Binario, fuente, manual
```

### 7.5 Operaciones en Archivos

```bash
# chmod: Cambiar permisos
$ chmod 644 archivo.txt             # Octal
$ chmod u+x script.sh               # Simbólico: agregar x al user
$ chmod g-w archivo.txt             # Remover w del grupo
$ chmod o-r archivo.txt             # Remover r de otros
$ chmod -R 755 directorio/          # Recursivo
$ chmod a=rx archivo                # Todos = r y x solo

# chown: Cambiar propietario
$ sudo chown juan archivo.txt           # Cambiar user
$ sudo chown juan:grupo archivo.txt     # User y grupo
$ sudo chown -R juan:grupo directorio/  # Recursivo

# chgrp: Cambiar grupo
$ sudo chgrp grupo archivo.txt      # Cambiar grupo

# umask: Máscara de permisos por defecto
$ umask                             # Ver actual (típicamente 0022)
$ umask 0077                        # Archivos privados por defecto
# Archivos nuevos: 777 - 022 = 755
# Archivos nuevos con 0077: 777 - 077 = 700

# stat: Ver metadatos completos
$ stat archivo.txt
  File: archivo.txt
  Size: 4096      Blocks: 8          IO Block: 4096
  Access: (0644/-rw-r--r--)  Uid: ( 1000/juan)   Gid: ( 1000/juan)
  Access: 2024-06-15 10:30:45.123456789 +0000
  Modify: 2024-06-15 10:30:40.987654321 +0000
  Change: 2024-06-15 10:30:42.654321987 +0000
```

---

## 8. FLUJO Y REDIRECCIÓN

### 8.1 Entrada (stdin), Salida (stdout), Errores (stderr)

```
CONCEPTOS CLAVE:

stdin  (0): Entrada estándar (teclado)
stdout (1): Salida estándar (pantalla)
stderr (2): Salida de errores (pantalla, pero separada)

CADA PROCESO ABIERTO COMIENZA CON ESTOS 3:

┌─────────────────┐
│   PROCESO       │
├─────────────────┤
│ stdin (0)   ← ┐ │ Normalmente: Teclado
│ stdout (1)  → ┐ │ Normalmente: Pantalla  
│ stderr (2)  → ┐ │ Normalmente: Pantalla
└─────────────────┘
```

### 8.2 Redirecciones

```bash
# Redireccionar stdout (>)
$ ls > archivo.txt          # Guardar salida en archivo
$ ls > archivo.txt 2>&1     # Guardar stdout Y stderr

# Redireccionar stderr (2>)
$ comando 2> errores.txt    # Errores a archivo
$ comando 2> /dev/null      # Descartar errores

# Redireccionar entrambos
$ comando > salida.txt 2>&1     # Clásico: salida + errores
$ comando &> salida.txt         # Moderno: lo mismo
$ comando 2>&1 | less           # Mostrar ambos en less

# Append (>>)
$ echo "texto" >> archivo.txt   # Agregar al final

# Input redirection (<)
$ cat < archivo.txt             # Equivalente a: cat archivo.txt
$ wc -l < archivo.txt           # Contar líneas

# Here-document
$ cat << EOF
Texto multilinea
Puede tener variables: $HOME
Cierra con: EOF
EOF

# Here-string
$ cat <<< "Texto simple"
```

### 8.3 PIPES (|) - El Corazón de UNIX

Una pipe conecta la salida (stdout) de un comando a la entrada (stdin) de otro.

```bash
# Sintaxis básica
comando1 | comando2 | comando3

# Los pipes que te salvarán la vida:

# Ver logs filtrados
$ cat /var/log/syslog | grep "error" | tail -20

# Contar ocurrencias
$ ls -la | wc -l

# Ordenar y mostrar
$ ps aux | grep firefox

# Pipeline completo: búsqueda → ordenado → mostrado
$ cat archivo.txt | grep "error" | sort | uniq -c | sort -rn

# Canalizar a less (para archivos grandes)
$ ls -R / | less
```

### 8.4 Redirección Avanzada

```bash
# Process substitution (bash avanzado)
$ diff <(ls dir1) <(ls dir2)
# Crea archivos temporales con las listas y los compara

# Duplicar file descriptors
$ comando > archivo.txt 2>&1    # stderr → stdout → archivo
$ comando 2> /dev/null > salida.txt  # stderr descartado, stdout guardado

# Canalizar a tee (guardar Y mostrar)
$ ls | tee listado.txt          # Muestra en pantalla Y guarda
$ comando | tee -a salida.txt   # Append (-a)

# Canalizar a xargs (pasar como argumentos)
$ echo "archivo1.txt archivo2.txt" | xargs rm
# Equivalente a: rm archivo1.txt archivo2.txt

$ find . -name "*.pyc" | xargs rm -f
# Elimina todos los .pyc
```

---

## 9. VARIABLES Y EXPANSIONES

### 9.1 Variables en Bash

```bash
# Definir variable
nombre="Juan"
edad=25
ruta="/home/juan/documentos"

# Usar variable (con $)
echo $nombre                    # Juan
echo "Hola $nombre"             # Hola Juan
echo 'Hola $nombre'             # Hola $nombre (comillas simples NO expanden)

# Comillas dobles vs simples
echo "Home: $HOME"              # Home: /home/juan (expande variables)
echo 'Home: $HOME'              # Home: $HOME (literal)

# ${} - Expansión segura
echo ${nombre}                  # Juan
echo "Mi nombre es ${nombre}!"  # Evita confusión con !

# Valores por defecto
echo ${variable:-"defecto"}     # Si no está definida, usa "defecto"
echo ${variable:="defecto"}     # Y además asígnala
echo ${variable:+con valor}     # Si existe, muestra esto
echo ${variable:?error message} # Si no existe, error

# Longitud de variable
echo ${#nombre}                 # 4 (número de caracteres)

# Substring
ruta="/home/juan/documentos"
echo ${ruta:0:5}                # /home
echo ${ruta:6:4}                # juan
echo ${ruta: -10}               # documentos (últimos 10)

# Reemplazo
archivo="documento.txt.bak"
echo ${archivo/.bak/}           # documento.txt (reemplaza .bak)
echo ${archivo//./ _}           # documento txt bak (reemplaza todos los .)
echo ${archivo%.*}              # documento.txt (elimina extensión)
echo ${archivo%%.*}             # documento (elimina todo desde el primer .)
echo ${archivo#*/}              # home/juan/documentos (quita primera parte)
echo ${archivo##*/}             # documentos (quita todo hasta el último /)
```

### 9.2 Variables de Entorno

```bash
# Variables globales (heredadas por procesos hijos)
export nombre="Juan"            # Crear variable de entorno
export PATH="$PATH:$HOME/bin"   # Agregar al PATH

# Ver todas las variables
env                             # Muestra todas
echo $VARIABLE                  # Ver una específica

# Variables comunes del sistema
echo $HOME                      # /home/juan
echo $USER                      # juan
echo $PWD                       # /home/juan/Documentos
echo $PATH                      # /usr/local/sbin:/usr/local/bin:...
echo $SHELL                     # /bin/bash
echo $HOSTNAME                  # nombre-computadora
echo $LANG                      # en_US.UTF-8
echo $TERM                      # xterm-256color

# Variables especiales de bash
echo $0                         # Nombre del script actual
echo $1, $2, $3                 # Argumentos 1, 2, 3
echo $@                         # Todos los argumentos
echo $#                         # Número de argumentos
echo $?                         # Código de retorno del último comando
echo $$                         # PID del proceso actual
echo $!                         # PID del último proceso de fondo
echo $_                         # Último argumento del comando anterior
```

### 9.3 Expansiones en Bash

```bash
# Expansión de glob (*,?)
ls *.txt                        # Todos los .txt
ls archivo?.txt                 # archivo1.txt, archivo2.txt, etc
ls [abc]*.txt                   # Comienzan con a, b, o c
ls [0-9]*.txt                   # Comienzan con número

# Expansión de llaves {}
mkdir dir_{1,2,3}               # Crea dir_1, dir_2, dir_3
touch archivo_{a,b,c}.txt       # Crea archivo_a.txt, etc
echo {01..10}                   # 01 02 03 ... 10
echo {a..z}                     # a b c ... z
echo {0..100..10}               # 0 10 20 ... 100

# Expansión aritmética $((
echo $((5 + 3))                 # 8
echo $((10 * 2))                # 20
numero=5; echo $((numero++))    # 5 (post-incremento)
echo $numero                    # 6

# Sustitución de comandos $()
fecha=$(date)                   # Ejecuta date, guarda resultado
echo "Hoy es: $fecha"
lista=$(ls *.txt)               # Lista archivos
echo "${lista[@]}"
```

---

# PARTE 4: EJERCICIOS DE ENTREGA

## 10. ASIGNACIÓN 1: Exploración del Filesystem

**Objetivo**: Familiarizarte con la navegación y comandos básicos.

**Tareas**:

1. **Mapear la estructura**
   ```bash
   # Crea un archivo llamado "mapa_filesystem.txt" que contenga:
   # - Ruta absoluta de tu home
   # - Ruta absoluta de /tmp
   # - Cuántos archivos hay en /bin
   # - Cuántos directorios hay en /usr
   # - Listar los 3 directorios más grandes en /var (si acceso)
   ```

3. **Búsqueda**
   ```bash
   # Crea un archivo "busquedas.sh" que:
   # - Encuentre todos los archivos .sh en tu home
   # - Cuente cuántos archivos comienzan con "." (ocultos)
   # - Busque la palabra "bash" en todos los archivos .txt del home
   # El script debe imprimir resultados
   ```

**Archivos a entregar**:
- `mapa_filesystem.txt`
- `permisos_analisis.txt`
- `busquedas.sh` (ejecutable)

---

## 11. ASIGNACIÓN 2: Scripting Básico con Bash

**Objetivo**: Crear scripts funcionales que demuestren conocimiento de variables, condicionales y loops.

**Problema 1: Organizador de Archivos**

Crea un script `organizer.sh` que:

```bash
#!/bin/bash

# El script debe:
# 1. Crear un directorio "organizados" en el home
# 2. Leer archivos del directorio actual
# 3. Organizarlos en subdirectorios según extensión:
#    - documentos/ (pdf, txt, doc, docx)
#    - imagenes/ (jpg, png, gif)
#    - audio/ (mp3, wav, flac)
#    - video/ (mp4, mkv, avi)
#    - otros/ (todo lo demás)
# 4. Mover cada archivo a su respectiva carpeta
# 5. Mostrar un resumen de cuántos archivos se movieron

# Estructura esperada:
# $ ./organizer.sh
# Creando directorio organizados...
# Organizando archivos...
# RESUMEN:
# Documentos: 5 archivos
# Imágenes: 12 archivos
# Audio: 3 archivos
# Video: 2 archivos
# Otros: 1 archivo
# Total: 23 archivos movidos
```

**Requisitos del código**:
- Usar variables para contar archivos
- Usar `if/else` para validar extensiones
- Usar loops (`for`) para procesar archivos
- Validar que el directorio "organizados" no exista antes de crearlo
- Mostrar mensajes informativos con `echo`

---

## 12. ASIGNACIÓN 3: Sistema de Backup

**Objetivo**: Crear un script de backup que copie archivos y los registre.

Crea un script `backup.sh` que:

```bash
#!/bin/bash

# El script debe aceptar argumentos:
# $ ./backup.sh [directorio_origen] [directorio_destino]
#
# Requisitos:
# 1. Validar que los argumentos se proporcionaron
# 2. Validar que el directorio origen existe
# 3. Crear directorio destino si no existe
# 4. Copiar TODOS los archivos recursivamente
# 5. Crear un archivo "backup.log" que contenga:
#    - Fecha y hora del backup
#    - Directorio origen
#    - Directorio destino
#    - Número de archivos copiados
#    - Tamaño total copiado
# 6. Si el backup ya existe, hacer incrementalidad (no copiar si no cambió)
# 7. Mostrar progreso con información visual

# Ejemplo de uso:
# $ ./backup.sh /home/juan/Documentos /home/juan/Backups
# Iniciando backup...
# Origen: /home/juan/Documentos
# Destino: /home/juan/Backups
# Archivos a copiar: 42
# Progreso: ████████░░░░░░░░░░ 50%
# ✓ Backup completado
# Resumen guardado en: /home/juan/Backups/backup.log
```

**Requisitos técnicos**:
- Argumentos: `$1`, `$2`
- Validación: `[[ -d $1 ]] && echo "Existe"`
- Contar archivos: `find . -type f | wc -l`
- Tamaño total: `du -sh [directorio]`
- Timestamp: `date "+%Y-%m-%d %H:%M:%S"`
- Usando `cp -r` o `rsync`

---

## 13. ASIGNACIÓN 4: Sistema de Logs

**Objetivo**: Crear funciones bash para logging y analizar logs.

Crea un archivo `logging.sh` que defina funciones:

```bash
#!/bin/bash

# Función 1: log_info
# log_info "Mensaje aquí"
# Salida: [2024-06-15 10:30:45] INFO: Mensaje aquí

# Función 2: log_error
# log_error "Algo falló"
# Salida: [2024-06-15 10:30:46] ERROR: Algo falló (en rojo en terminal)

# Función 3: log_warning
# log_warning "Advertencia"
# Salida: [2024-06-15 10:30:47] WARNING: Advertencia (en amarillo)

# Función 4: log_to_file
# log_to_file "archivo.log" "ERROR" "Mensaje de error"
# Guarda en archivo.log con timestamp

# Función 5: parse_logs
# parse_logs "archivo.log"
# Analiza y muestra estadísticas:
#   - Cuántos INFO
#   - Cuántos ERROR
#   - Cuántos WARNING
#   - Primero registro
#   - Último registro

# EJEMPLO DE USO:
# $ source logging.sh
# $ log_info "El script comenzó"
# [2024-06-15 10:30:45] INFO: El script comenzó
# $ log_to_file "app.log" "INFO" "Conexión establecida"
# $ parse_logs "app.log"
# === ANÁLISIS DE LOGS ===
# INFO: 2
# WARNING: 0
# ERROR: 1
# Período: 2024-06-15 10:30:45 - 2024-06-15 10:35:10
```

**Requisitos**:
- Definir funciones bash
- Usar timestamps
- Validar argumentos en funciones
- Usar colores en terminal (ANSI codes)
- Procesamiento de archivos con grep/awk

---

## 14. ASIGNACIÓN 5: Utilidad Interactiva

**Objetivo**: Crear un menú interactivo que permita operaciones en archivos.

Crea un script `file_manager.sh`:

```bash
#!/bin/bash

# Menú interactivo con opciones:
# 1. Ver contenido de archivo
# 2. Copiar archivo
# 3. Mover archivo
# 4. Eliminar archivo
# 5. Cambiar permisos
# 6. Ver estadísticas de archivo
# 7. Buscar archivos
# 8. Salir

# Requisitos:
# - Mostrar menú en bucle
# - Validar entrada del usuario
# - Confirmación antes de operaciones destructivas
# - Manejo de errores
# - Historial de operaciones (opcional +20 puntos)

# Ejemplo de uso:
# $ ./file_manager.sh
# ===== GESTOR DE ARCHIVOS =====
# 1) Ver contenido
# 2) Copiar archivo
# 3) Mover archivo
# 4) Eliminar archivo
# 5) Cambiar permisos
# 6) Ver estadísticas
# 7) Buscar archivos
# 8) Salir
# Seleccione opción: 1
# Ingrese ruta del archivo: /home/juan/documento.txt
# [contenido del archivo]
#
# Seleccione opción: 8
# Saliendo...
```

**Requisitos técnicos**:
- Bucle `while` para menú
- `read` para entrada del usuario
- `case` para opciones
- Funciones para cada operación
- Validación de rutas
- Confirmación: `read -p "¿Continuar? (s/n): "`

---
---

# INDICACIONES DE ENTREGA

## Formato de Entrega
```
entrega_bash/
├── README.md (descripción de todo)
├── asignacion1/
│   ├── mapa_filesystem.txt
│   ├── permisos_analisis.txt
│   └── busquedas.sh
├── asignacion2/
│   └── organizer.sh
├── asignacion3/
│   └── backup.sh
├── asignacion4/
│   └── logging.sh
└── asignacion5/
    └── file_manager.sh
```

## Documento Debe Contener
```markdown
# Entrega Bash - Nombre del Estudiante

## Asignación 1: Exploración del Filesystem
- Descripción breve
- Cómo ejecutar
- Resultados principales

[Mismo formato para cada asignación]

## Cómo Ejecutar Cada Script

Para cada script:
\`\`\`bash
$ chmod +x asignacionX/script.sh
$ ./asignacionX/script.sh [args si aplica]
\`\`\`

## Notas Personales
[Dificultades encontradas, qué aprendiste, etc.]
```

---



---

# RECURSOS DE AYUDA

## Cheat Sheet Bash

```bash
# Variables
VAR="value"
echo $VAR
export VAR

# Condicionales
if [ $? -eq 0 ]; then
    echo "Éxito"
else
    echo "Error"
fi

[ -f archivo ]    # ¿existe archivo?
[ -d directorio ] # ¿existe directorio?
[ -z "$VAR" ]     # ¿VAR vacío?
[ "$A" = "$B" ]   # ¿iguales?

# Loops
for i in {1..10}; do
    echo $i
done

for archivo in *.txt; do
    echo $archivo
done

while [ $contador -lt 10 ]; do
    echo $contador
    ((contador++))
done

# Funciones
myfunc() {
    echo "Argumentos: $@"
    return 0
}

myfunc arg1 arg2

# Redirección
comando > archivo.txt      # stdout
comando 2> errores.txt     # stderr
comando &> salida.txt      # ambos
comando | otro_comando     # pipe

# Comillas
"texto $VAR"     # Expande variables
'texto $VAR'     # Literal
```

## Documentación en Línea
```bash
man bash                # Manual de bash
help if                 # Ayuda de comando
bash -x script.sh       # Debuggeo (muestra comandos)
set -e                  # Terminar si hay error
set -u                  # Error si variable undefined
```

---


