
[Badstore 1.2.3](https://www.vulnhub.com/entry/badstore-123,41/)

  

# **Índice:**

- **0- Introducción - Reconocimiento**
    
- **1- Sanitización del servidor web**
    
- **2- Inyección XSS(Cross-Site Scripting)**
    
- **3- Inyección SQL**
    
- **4- CSRF(Cross-Site Request Forgery)**
    

  
  
  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXc4sjqRvanwvF1HbhND-yoMDR1yDkNnmXW2DChKYSA4jYyCiRabCPRrfoaML1-EJvmdTQ4CrjCTJz-StQRIB_sDqxMyu0OAu1bJ54AOi4UQcPT9VIFzClnQQq4ufGfGS70ECZ1m7g?key=CeGcHJ9xIa42EDbVHqizPil4)

  


# **0- Introducción - Reconocimiento**

Empezaremos por el reconocimiento de la máquina con diferentes herramientas para luego analizar lo encontrado y encontrar vulnerabilidades existentes en la máquina, subrayando y marcando lo más importante para encontrar vulnerabilidades:

|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Nmap**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| # Nmap 7.94SVN scan initiated Wed Nov 13 17:06:03 2024 as: nmap -p- -sS -sC -sV -vvv -n --min-rate 5000 -oN nmap_badstore 192.168.8.11<br><br>Nmap scan report for 192.168.8.11<br><br>Host is up, received arp-response (0.00012s latency).<br><br>Scanned at 2024-11-13 17:06:04 -03 for 16s<br><br>Not shown: 65532 closed tcp ports (reset)<br><br>PORT     STATE SERVICE  REASON         VERSION<br><br>80/tcp   open  http     syn-ack ttl 64 Apache httpd 1.3.28 ((Unix) mod_ssl/2.8.15 OpenSSL/0.9.7c)<br><br>\|_http-server-header: Apache/1.3.28 (Unix) mod_ssl/2.8.15 OpenSSL/0.9.7c<br><br>443/tcp  open  ssl/http syn-ack ttl 64 Apache httpd 1.3.28 ((Unix) mod_ssl/2.8.15(HTTPs) <br><br>3306/tcp open  mysql    syn-ack ttl 64 MySQL 4.1.7-standard |

|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Dirb - Fuzzing web**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---- Scanning URL: http://192.168.8.11/ ----<br><br>==> DIRECTORY: http://192.168.8.11/backup/<br><br>+ http://192.168.8.11/cgi-bin/ (CODE:403\|SIZE:275)<br><br>+ http://192.168.8.11/favicon.ico (CODE:200\|SIZE:1334)<br><br>==> DIRECTORY: http://192.168.8.11/images/<br><br>+ http://192.168.8.11/index (CODE:200\|SIZE:3583)<br><br>+ http://192.168.8.11/index.html (CODE:200\|SIZE:3583)<br><br>+ http://192.168.8.11/robots (CODE:200\|SIZE:316)<br><br>+ http://192.168.8.11/robots.txt (CODE:200\|SIZE:316)<br><br>==> DIRECTORY: http://192.168.8.11/supplier/ |

|                                                                                                                                                                                                                                     |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Whatweb - Reconocimiento de tecnologías utilizadas**                                                                                                                                                                              |
| http://192.168.8.11 [200 OK] Apache[1.3.28][mod_ssl/2.8.15], Country[RESERVED][ZZ], HTTPServer[Unix][Apache/1.3.28 (Unix) mod_ssl/2.8.15 OpenSSL/0.9.7c], IP[192.168.8.11], OpenSSL[0.9.7c], Title[Welcome to BadStore.net v1.2.3s] |

Se puede observar versiones obsoletas, MySQL 4.1.7(Puerto 3306), Apache 1.3.28(Puerto  80 y 443), también se puede ver que el servidor web cuenta con un certificado SSL para utilizar HTTPS y encriptar las consultas al servidor enviadas por un usuario y viceversa(Aunque aparentemente, el certificado no tiene validez por la fecha de expiración).

A su vez también se puede observar que se trata una máquina Linux, debido al TTL en el escaneo de Nmap.

  
1- Sanitización del servidor web

El proceso de sanitización de un servidor web es eliminar toda información confidencial expuesta dentro del servidor web, como puede ser documentos confidenciales, backups, registros, entre otros.

Como pudimos observar en la utilización de la herramienta Dirb para realizar fuzzing del servidor web, este no es el caso de una sanitización correcta.

Vemos que hay dentro de los directorios y ficheros dentro del servidor web que encontró Dirb:



**Archivo robots.txt**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdAqGkkvfo2Hskkg-4FLfEuNf093iMvnMiLwdkLITpOhrUFUIaVVUPP2rSdkXMV5lPcly_h9qLn7iasQ2vS-ffv8lDI2-Fc9SwoealKY0j70z8Sa7zSqxaLJsYzVQOuG_-7X5xLIw?key=CeGcHJ9xIa42EDbVHqizPil4)

El archivo robots.txt es un archivo de texto que indica a los rastreadores de los buscadores qué páginas de un sitio web pueden o no rastrear, osea mostrarse ante la búsqueda de un usuario mediante Google por ejemplo(Facilita mucho el uso de Google Dorks para este tipo de casos).



**Directorio supplier**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXehnVSrB7HDffkYks2qVYK-jk7rkFjXOYrAwRUlayy6ZTd6Jn3Tq2P-b0ZrDzAvq1bDSiqlGQZw736zePUJ4mOdeoUcK6LuIP5HdNWeFry1uCfuppE-7UN9EwOhSlOzrTZNvK0t1Q?key=CeGcHJ9xIa42EDbVHqizPil4)

Se puede ver dentro del directorio supplier un archivo llamado “accounts”, si entramos dentro del mismo para chequear que hay:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcKMTngbiDMrJwfU-siZVcoDkhKHVg3booAF4qnvlNEFoHqn66fYBFMLTysxdFKXllJkHXLbedFsfYmHpnz8Y6_evGONAcEqFaM9uVLziM3vtwfvEVYiUs9JizWCyUtFeguxBlJkQ?key=CeGcHJ9xIa42EDbVHqizPil4)

Podemos verificar que a simple vista parecen cuentas por el nombre del fichero y que están cifradas en el algoritmo de cifrado Base64:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXd-bv5I6YoMA_aNmhkHHct8Q3ASQtZOHiCZqOL7n1rDX-DRS5Yk8yFCl2cHWAfWcpPim9xK4R6oPaxmSZiPGLOJuyS548RepGWLvGfAVGm1mL25n5BJTdqNE12dip4RA1I-rBrQ?key=CeGcHJ9xIa42EDbVHqizPil4)

  

Si procedemos con el descifrado a texto claro utilizando la herramienta base64 de Kali, una vez descargado el recurso accounts con wget, este sería el resultado:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdTPnefC_RbR6Hjv0hv6a689VA2Jcj__0n_DmMkjCN9JlxdXZV6hlZPoa7mGZMSueZrTWsxmarqGvCYs1nz5fA2D9-yKMT8QfI8qqcbBGXh34QZi4kB05xNP2Hd48PK8dWQfSnqXQ?key=CeGcHJ9xIa42EDbVHqizPil4)



# **2- Inyección XSS**

Si observamos la página web, podemos notar que a la izquierda encontramos un campo de búsqueda

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXegiSErJjbMeR5ZsoPMEl3AJFjQ3PbpS-weNuepcDSNdExU-SyQf8dvG7bGalV-V28aibQA1JMXVefWGBhabxhPWxb_e0n2zbRw12NiA0jC4m7Y_6qNLmZeZFCwq9hfH9oCY6eIPQ?key=CeGcHJ9xIa42EDbVHqizPil4)

Si realizamos una inyección XSS básica introduciendo código HTML para evaluar una posible inyección XSS, por ejemplo:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfO_1wQ_5SZFUu0ZI-KJYasDDFv6aVtxKv9teQEjjkZLgaZydN8WviPL_UaBgob6llONTF4xh3_vWyszjaJHiX04tA_jHo9efCVslvzzKGiOyoonDh8wf6Wev18G_5wlW2iZylZbQ?key=CeGcHJ9xIa42EDbVHqizPil4)

Como podemos visualizar en la imagen, este sitio web es vulnerable a inyecciones XSS y podemos insertar código HTML dentro de los campos(dependiendo como el desarrollador del sitio web haya gestionado como texto las entradas del campo o como código HTML). Este tipo de ataques son peligrosos por una sencilla razón, se pueden utilizar varios payloads(carga útil) en HTML para insertarlos en el campo y obtener diversos resultados, por ejemplo redirigir a un usuario a una página web maliciosa de phishing que pida datos, entre otros tipos de payloads. (Se puede montar un servidor Apache en Kali para alojar un sitio web malicioso de Phishing con SEToolkit y redirigir al usuario con código HTML en algún campo donde posteriormente quede registrado la entrada(Código HTML malicioso que redirige a sitio web malicioso) en la página.

  
Podemos encontrar varios [repositorios en GitHub](https://github.com/payloadbox/xss-payload-list/blob/master/Intruder/xss-payload-list.txt) con payloads y explotar la vulnerabilidad XSS

  
# **3- Inyección SQL**

Como podemos verificar en la imagen a continuación, dentro de este sitio web, hay formularios de login

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdqyn52mLRTqeu4CzyNxcSEuylA_MOi5oo9Yx2LwVmwYJAHwcyAdZP9h47QgxJBYbNOXJRzLZ57iIiHfNWhM-qc709OQSWPOShxC5-weR7Xz_EBiUKX8Mj788u_Z2lxBvBVBYrq?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Lo que tratamos de realizar con una inyección SQL es alterar la consulta que recibe la base de datos en simples palabras. Vamos a probar una inyección SQL básica para alterar la consulta y logearnos con el primer usuario que se presenta en la tabla de usuarios de la base de datos(Por lo general suele ser el administrador)

  
## **' or 1=1-- -'**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXd15DqHtBU6HlvcKet4H5BG9hIpMrynK0kfDyQuAE9Yg8ebvBk6NTpYY4VjXdtFXJdI0gKHGOZFg1QUABunv8VcBGUKuCnmz3rqM2NesfhqgD329ZowTaikd7H8fO2nNUAKARi6?key=CeGcHJ9xIa42EDbVHqizPil4)

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXd7txzg5dBMLrgIoHhTmKN2VzbmUQyc587GIRw6JeW_nlodeA-RFtr8_3zn-DKPRs6Fomy9Xynbz3itRphlN2ZC7RA0RLDzkafVx42COPtAX9Dvim9GW57ZL_vWwKkzLQ6QmI2-JQ?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Hemos iniciado sesión sin la necesidad de ninguna contraseña

Vamos a ver estas consultas de fondo con la herramienta Burp Suite de Portswigger

Pero antes vamos a configurar nuestro navegador Firefox de Kali para que Burp Suite haga de intermediario con su proxy

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXd_B63B9UFqtNiShnqp4D5DA4rvsOyoWTZqeASoKepjwPD_BDyeupkV8hqEqweLcz63Dos4g23xiz5dij5k-KAIt011kSp6jhSkejaP885wFeQ_2EtSvb6P73HKDdHSkF0EwnwCKA?key=CeGcHJ9xIa42EDbVHqizPil4)

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdOBggghBaMlxRwUDu66ysmoFgu3MHlwFzD8XquBu7VAgDBHnJycclDgrJsURM30A6K8A566tpchRJlU5uAdkCOKrU1InXRDetnC7yo74P0k87BmB9mFbfphPoX-0S4tBm_uHAaQw?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Probemos crearnos una cuenta en el sitio web interceptando el tráfico mediante el proxy de Burp:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdNwHKdYwE15mXXiz877XFn5mhlny4pMb-bYFzZ6Vx5ajCbb21sNfghO-hgCLH0oI84JzzcrdXZQ7Ezfdgkmr4dz9TkXZqwM5juQ6uldZrF07ul4uqaQSYx6jc3_vfvCI_VGiI7?key=CeGcHJ9xIa42EDbVHqizPil4)

Este es el resultado al momento de darle al botón de Register:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeAlHawE-eYY3nZVrQv-0_d2J5F1aLOusalQPDkJlnfir9mSInesefl9fMaZjK_0nnVfb5BmsnL7si58teMtEhwa3ca8WXOTpr2SfwqRbKMwGhLimAUqaWZ0ZXi8rd6wsYFZDAj?key=CeGcHJ9xIa42EDbVHqizPil4)

Podemos notar que entre los parámetros que estamos enviando en la consulta, uno de ellos es: “role=U”, si usamos el sentido común, estamos definiendo el rol que va a tener ese usuario, en este caso es U de Usuario, ahora bien.

  
**¿Qué pasaría si lo cambiamos por una A de Administrador y dejamos que siga adelante la consulta con el botón de “Forward”?**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXc0eLSPWIpuW00a_9uNvJIU52AEuPPa3dbTvlBF8zP5Mw5AUG1XwP-wqQIb-4ywFwvjbfLRX2TpD0T5YuxYQy1gbyxYPvVmDk8Y3kmhFSLbFXIhYVL-OqUOzgSmY14Wu0N-YT2MLA?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Dejamos de interceptar una vez que dejamos pasar las consultas que llegan a Burp Suite y vemos que nos creo el usuario correctamente:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdGAAfsGBTVchYtNYYavHxEM7IqNSayHrZslYvrJjOZdDMrAJJWjMBtv9DSU2KOHVaGDMbP3fYqc0YXTcngNAbwXZQsQPjIQcfXs2AmEcBpEiQvMAkYUtKjGTN9xnleCpmH_ayJTQ?key=CeGcHJ9xIa42EDbVHqizPil4)

Tendremos una cuenta de Administrador alterando la consulta realizada a la base de datos

**¿Y AHORA?**

Realizando una investigación dentro del manual de “Badstore Manual v1.2”, en busca de un panel de administración que conecte con alguna base de datos del servidor.

  

El manual de Badstore lo podemos encontrar en las referencias como se ve en la imagen:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfA5RCJp18MqdMfjZ_BCiFAvgevfSWVp9xJi99ysPWgdDc6zbJmerFDafnXCerNO1SZkw4NeZPdjATATjwuj2swDGDjhxHfaT7nH_eQT0ocEmVMCzQGW1zgP9rb7ggCtFEQ9VVK?key=CeGcHJ9xIa42EDbVHqizPil4)

  
  
Leyendo el manual nos encontramos con esto:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdK3rH1edVLAeNWvHeM-6V-gtfnyKzOUWn4fmAK0hSZn__AfGFcLWJV6ez8ErhhkNqd8DLsxmvHL8zT9brAXvbyaDB4GbI6zf2JAXkW4s90aQHcOloymIs0mhkD8aRIUIs42O2eRw?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Si volvemos al sitio web y vamos a “Mi cuenta”, vemos que la URL es algo así como esto:


![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfBb_NQlnii7UTy81Hh4r2A06eL2RkhtTmTK3zbNDoA8QG16ZA22klfS8Iwdgg_m08711iiOPgRXIyRvCKyTXL26__xUhqqyHLgQUjDDnNvZdM5VodCnSGKz-vNS26oAXRSw_6CWA?key=CeGcHJ9xIa42EDbVHqizPil4)

  
“192.168.8.11/cgi.bin/badstore.cgi?action=myaccount”


Vamos a cambiar el ?action=myaccount por ?action=admin dentro de la URL:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf1dAeswiBVuxkT9KUSGpwlHUn9IEkL5NbMBCyngjgtFYzc8dB8sJb6PWq1AiYTfErAnqFf3dB6Mk_3Y7-lcLH9k1oGzZm_Rn9YNm5oS1Ns2OcvLJPUthETdYnpGe7Yra8O3nw3JQ?key=CeGcHJ9xIa42EDbVHqizPil4)

Obtenemos un panel secreto de administración, vamos a seleccionar en la lista desplegable “Show Current Users” y darle en “Do It”, obtendremos la base de datos de todos los usuarios:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeL7XOmYwpQpS0DYgzqrPLkaoVXRm_UpyBZOFk6fDQenGrhmA2ToB2jqdQlKSKkm_hbvvQZsKS8AlbV01XHNSBdwt7t44kuFGiUwuOhsI28aOP_5tssIdXIQkZx_KLs3GvkgqGPpg?key=CeGcHJ9xIa42EDbVHqizPil4)

Una vez ya obtenido acceso al panel de administración, podremos resetear la contraseña de los usuarios interactuando con la base de datos, agregar y eliminar usuarios, mirar la base de datos donde se encuentran los datos de los usuarios, etc

  

# **4- CSRF(Cross-Site Request Forgery)**

El CSRF es una vulnerabilidad de un sitio web en el que comandos no autorizados son transmitidos por un usuario en el cual el sitio web confía mediante la modificación de los métodos POST a GET para generar una URL.

Este tipo de ataques por lo general es muy frecuente verlos en formularios de cambio de contraseña en donde para realizar el cambio de contraseña no te exige la contraseña actual de la cuenta

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdGxpyMpOK6dZxicsp_vRkiXgPLA--K4fMT_f5YxcmvMuAF4A9lZxifeF2_bRJllUbL7lKYo9dMnSGYYtQzU7Y06X-BEK8TdjQq0yQxQGPrInxaHk6nciW7Br8iYIy-JjQ2nDJu?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Vamos a poner BurpSuite en escucha de consultas que pasen por el proxy y vamos a cambiar el usuario y la contraseña

  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcWSo73vzXNtwmCG_roC4FO9r_bufG9NT22jtBMZK3XP4ZAFCvi2vUWZz8kJveJP2P29AxTIx7xycVx8oeS_1cuvZrYLJLMvdTxvI7Gi6rAhdN3FVsD1verkkTE57rQkpaedUSQ3w?key=CeGcHJ9xIa42EDbVHqizPil4)

Vemos que la consulta se está transmitiendo con el método POST, vamos a cambiar la consulta a método GET para generar un link malicioso para que solo con entrar a la URL, si el usuario se encuentra logueado, se le cambie la contraseña y el correo por el que nosotros le indiquemos a la URL

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfHf9OJgZCx4WkAW7aH7SOz_nAXTrkJenBtvCjw4i5AZDkMlTS0u3TUd1c0Jp_W6ER_3guJP_KR0SDPw3BSI5fym7xRHIQcsUClJC63bK4xLOEWMg0TiTYZqylsgAL7Wkb9gdgMLw?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Obtendremos un resultado como este:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcihNoH8o3k2ZyKvRpE-UvvWycsV6dkXNiLji7CXl3mxPotZQ3EgNQgLyKb-cHqwY0tVardXEVcaYaHD3joUt6j3pVCRHwRPX1TSVFf6SYcu7ggXC9hlN5L_JR7ZVBB-4AH1UrJ8w?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Como podremos ver, en la primera línea de la consulta nos encontramos con algo como esto:

“/cgi-bin/badstore.cgi?action=moduser&fullname=&newemail=asd%40gmail.com&newpasswd=hola&vnewpasswd=hola&role=A&email=tomz%40gmail.com&DoMods=Change+Account”

Ahora, ¿Qué pasaría si a esta parte de la primera línea que parece ser una URL, le agregamos la IP del servidor web?

Resulta en algo como esto:

“http://192.168.8.11/cgi-bin/badstore.cgi?action=moduser&fullname=&newemail=asd%40gmail.com&newpasswd=hola&vnewpasswd=hola&role=A&email=tomz%40gmail.com&DoMods=Change+Account”

  
Lo que sucedería es que esta URL que generamos, cambiaría la contraseña de nuestra cuenta por la que le especifiquemos dentro de la URL si accedemos.

Hagamos la prueba:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfoq5up5zlX_eKLVhzlpb40KBhtH7Xt6F6To2xw1P4AjRJpEICtO-DuTrmNDv2zHtvSORRuOkUigB3sNTTkLJmRQ36FVRbzeyck3PIlvpUgQdl4IphpVQlUQ2qfZJtZtUvxH_4Aew?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Ahora vamos a cambiar la contraseña por 2 hola con el enlace malicioso que creamos con Burp. Una vez ya modificada la contraseña en el enlace y ya entrado al mismo, veremos algo como esto:


![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfS2kAKBCorXo3ibW1McD150JoSDA91MAlYKjJw7lSc-SOIXANqmHuin7LWhmO10gbHSCnHkU73xDIav1EvgtJfaKDPLEn4Ruz9OO1LjRz_4zJL1BKi8QN5N9j4jiYVDd75lRTvaQ?key=CeGcHJ9xIa42EDbVHqizPil4)


Hagamos la prueba de logearnos con la contraseña “hola”:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXedQDVhyOR6V2s0-jGt7qo8AmPo3eNQO-U6myaOhPvVhfzwX5f9OrAZZzdAQ_lmg3OBZ-dXdQLxI8NXtODaPcTv1u0vp0ZJy-vv7oTPqpUVOjRSttvM-v4b6q-cLTunHmOAZbXywQ?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Nos da contraseña incorrecta


![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXe264G4rq_Ztt_v_eDAqJpQUkr25K6SLlEbF8oxsLkWfhogjOaZtHs0pBs0-pti53SyxBtkIhvgOKeo4It2u7bmFIqtvbNUYtR9kwJTXP62khxwd05_NGCK7KQAIOxFyg4pII8f-w?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Y si probamos con la contraseña que pusimos en el enlace malicioso, “holahola”


![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXesv6ODev8PCjo92DlPY8RePOLmMevt3dzvnYty9_m1yDxkd1l-BLcjjL8iUVoNZMpXiFl2GkAj7rorcY3V8nvV2Hv_jWqU0tV9MDNFJIYXQ6_Br7_8ZKAzmWwnjp5Bh2tfVbpc7g?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Nos inicia correctamente:

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfQHi3RyFHK-K-0_h4mxhE1sWWWenlGHdTaBLr6o0cvJj2aHiOqTktIIsC3d0JhmYDUA0ydNTjRPRAdXI49lksZDyhLm2McxL6N-up_mVlGERxI9ImPzSFz2Le7nGaYTgsKC2Rmcg?key=CeGcHJ9xIa42EDbVHqizPil4)

  
Esto resulta útil para realizar algún phishing en webs que son vulnerables, para cambiarle la contraseña a un usuario que ya se encuentre logueado en ese mismo sitio

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

Bibliografía:

- [https://www.vulnhub.com/entry/badstore-123,41/](https://www.vulnhub.com/entry/badstore-123,41/)
    
- [https://github.com/payloadbox/xss-payload-list](https://github.com/payloadbox/xss-payload-list)
    
- [https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_SQL)
    
- [https://es.wikipedia.org/wiki/Cross-site_request_forgery](https://es.wikipedia.org/wiki/Cross-site_request_forgery)
    
- [https://es.wikipedia.org/wiki/Cross-site_scripting](https://es.wikipedia.org/wiki/Cross-site_scripting)
    
- [https://portswigger.net/burp/documentation](https://portswigger.net/burp/documentation)
    
- [https://nmap.org/docs.html](https://nmap.org/docs.html)
    
- [https://www.geeksforgeeks.org/introduction-to-dirb-kali-linux/](https://www.geeksforgeeks.org/introduction-to-dirb-kali-linux/)
    
- [https://github.com/urbanadventurer/whatweb/wiki](https://github.com/urbanadventurer/whatweb/wiki)
    
- [https://www.dcode.fr/cipher-identifier](https://www.dcode.fr/cipher-identifier)
    
- [https://www.baeldung.com/linux/cli-base64-encode-decode](https://www.baeldung.com/linux/cli-base64-encode-decode)
    

  
  
**