# Guía del alumno — Laboratorios Wi-Fi (Hacking Ético)

> **Entorno 100 % virtualizado.** Todas las redes de esta práctica son radios
> simuladas con `mac80211_hwsim` dentro de tu máquina. No se emite nada por
> radio real y los ataques solo funcionan contra los puntos de acceso del
> laboratorio. **Uso exclusivamente educativo.** Atacar redes reales sin
> autorización expresa por escrito es delito (en España, arts. 197 bis y 264
> del Código Penal).
>
> **Esta guía es genérica.** Vale para cualquier laboratorio generado con
> `lab_generator.py`. Tu despliegue concreto puede incluir **todos** los
> escenarios que se describen aquí o **solo algunos**, con SSID, canales y
> número de clientes distintos. No des por hecho ningún nombre de red ni ningún
> canal: identifícalos siempre en cada sesión (ver §1 y §2).

---

## 0. Puesta en marcha

```bash
sudo ./despliegue.sh
```

Al terminar, el script imprime un cuadro con **todos los escenarios de tu
laboratorio**. Por cada red muestra:

- **SSID** (nombre de la red),
- **canal**,
- **BSSID** (dirección MAC del AP) — **cambia en cada despliegue**, apúntalo
  cada sesión,
- una **pista** del objetivo didáctico.

También te indica **cuáles son tus interfaces de trabajo** (`wlan0`, `wlan1`,
… hasta el número que haya configurado tu profesor). Para desmontarlo todo:
`sudo ./limpia.sh`.

### Reglas del laboratorio

- Trabaja **solo** con tus interfaces `wlan0 … wlanN` (las que anuncia el
  despliegue). Puedes usar varias a la vez.
- **No toques** las interfaces `wlan_ap_*` / `wlan_cl*` ni los *network
  namespaces* `ns_*`: son la infraestructura de los escenarios.
- Las claves **no están en claro** en el sistema y **cambian en cada
  despliegue**. No está permitido obtener resultados leyendo los scripts o la
  configuración del laboratorio: **cada resultado debe salir del ataque** a la
  red correspondiente.
- Herramientas disponibles: suite **aircrack-ng**, **hcxdumptool / hcxtools**,
  **hashcat**, **reaver / bully** (WPS), **tshark / Wireshark**. Diccionario:
  el que te indique el profesor (por defecto `/usr/share/wordlists/rockyou.txt`).

---

## 1. Reconocimiento (hazlo siempre primero)

Como los nombres y los tipos cambian de un laboratorio a otro, **lo primero es
identificar qué red es cada una**: su seguridad, su canal y si tiene clientes.

**Objetivo del paso:** para cada SSID del cuadro, determinar su tipo de
seguridad (Abierta / WEP / WPA2-PSK / WPA2-Enterprise / WPA3) y si hay
**clientes asociados**. Con esos dos datos eliges la técnica adecuada de las
secciones siguientes.

**Pistas de dónde mirar:** la columna de cifrado y la de autenticación del
escáner, la presencia o no de estaciones asociadas, y si el AP anuncia **WPS**.
Una misma etiqueta (p. ej. "WPA2") puede corresponder a varios escenarios
distintos (handshake, PMKID con clientes, PMKID sin clientes): fíjate en los
clientes para distinguirlos.

**Preguntas:**

1. ¿Qué campo de las tramas *beacon* te dice el tipo de cifrado y
   autenticación de cada red?
2. ¿Cómo distingues una WPA2-**Personal** (PSK) de una WPA2-**Enterprise**
   (802.1X) solo observando el tráfico?
3. ¿Qué redes de tu laboratorio tienen clientes asociados y cuáles no?

---

## 2. Catálogo de escenarios

Resuelve **únicamente los escenarios que existan en tu laboratorio**. Cada uno
sigue el mismo formato: *objetivo* (qué tienes que conseguir) y *preguntas*
(qué debes razonar). **No se incluye la solución**: la técnica la investigas y
la aplicas tú.

| Seguridad | Variante | Qué tienes que conseguir |
|---|---|---|
| Abierta | — | Las credenciales que envía un cliente |
| WEP | — | La clave de la red |
| WPA2-PSK | Handshake | La clave de la red |
| WPA2-PSK | PMKID con clientes | La clave de la red (por otra vía) |
| WPA2-PSK | PMKID sin clientes | La clave de la red sin ningún cliente |
| WPA2-Enterprise | 802.1X / EAP | La identidad y el método EAP (y las credenciales) |
| WPA2 + WPS | PIN | La clave de la red a través del PIN de WPS |
| WPA3-SAE | Puro | Valorar la viabilidad del ataque de diccionario |
| WPA3 | Modo transición | La clave aprovechando la compatibilidad con WPA2 |
| WPA3 | *Downgrade* | Forzar a un cliente a una variante más débil |

---

### 2.1. Red abierta

Sin cifrado. Hay un cliente conectado que inicia sesión periódicamente en una
aplicación web.

**Objetivo:** obtener el usuario y la contraseña que envía.

**Preguntas:**

1. ¿Por qué has podido leer las credenciales? ¿Qué capa debería haberlas
   protegido?
2. ¿Cambiaría algo si la web usara HTTPS? ¿Y si la red fuera WPA2-PSK y tú
   conocieras la clave?

---

### 2.2. WEP

Red WEP con clientes conectados que generan tráfico.

**Objetivo:** obtener la clave de la red.

**Preguntas:**

1. ¿Por qué WEP se puede romper **independientemente** de lo robusta que sea la
   clave?
2. ¿Qué has hecho para **acelerar** el ataque y por qué funciona?

---

### 2.3. WPA2-PSK — captura del handshake

Red WPA2 personal con clientes conectados.

**Objetivo:** obtener la clave de la red.

**Preguntas:**

1. ¿Qué material criptográfico has necesitado capturar y por qué permite atacar
   la clave **sin estar conectado** a la red?
2. ¿Qué has tenido que hacer para conseguirlo y qué efecto tiene sobre los
   clientes legítimos?
3. ¿Qué haría inviable el ataque aunque capturaras lo mismo?

---

### 2.4. WPA2-PSK — PMKID con clientes

Red WPA2 personal con al menos un cliente conectado. Admite una vía de ataque
**distinta** a la del handshake.

**Objetivo:** obtener la clave de la red **sin** forzar el handshake 4-way de
un cliente.

**Preguntas:**

1. ¿Qué es el PMKID, en qué mensaje viaja y cómo se calcula?
2. En esta red, ¿cuántas vías distintas había para atacar la clave? ¿En qué se
   diferencian en cuanto a "ruido" sobre los clientes?

---

### 2.5. WPA2-PSK — PMKID sin clientes (*clientless*)

Red WPA2 personal **sin ningún cliente conectado**.

**Objetivo:** obtener la clave de la red.

**Preguntas:**

1. ¿Por qué los métodos que dependen de un cliente **no** funcionan aquí, y sí
   funcionaban en §2.3?
2. Completa la tabla:

   | | PMKID con clientes | PMKID sin clientes |
   |---|---|---|
   | ¿Hay clientes conectados? | | |
   | ¿Necesitas un cliente para el ataque? | | |
   | Material que puedes obtener | | |

3. ¿Qué conclusión sacas sobre la seguridad de WPA2-PSK frente a este ataque?

---

### 2.6. WPA2-Enterprise (802.1X / EAP)

Red WPA2 corporativa con un cliente que se autentica contra un servidor EAP
(PEAP/MSCHAPv2).

**Objetivo:** identificar, a partir del tráfico, la **identidad** con la que se
autentica el usuario y el **método EAP** que usa la red.

**Preguntas:**

1. ¿Qué información de la autenticación viaja **sin cifrar** y cuál va protegida
   por el túnel TLS?
2. Describe qué ataque permitiría capturar las **credenciales** del usuario en
   este tipo de red y qué configuración del cliente lo evitaría.

---

### 2.7. WPA2 con WPS

Red WPA2 personal con **WPS** (PIN de 8 dígitos) habilitado.

**Objetivo:** recuperar la clave de la red aprovechando el PIN de WPS.

**Preguntas:**

1. ¿Por qué el PIN de WPS reduce el espacio de búsqueda muy por debajo de los
   10⁸ valores teóricos? (Piensa en cómo se valida el PIN.)
2. ¿Por qué este ataque es **online** (contra el AP) y no offline, al contrario
   que el del handshake de WPA2?
3. ¿Qué medida del AP detiene o ralentiza el ataque?

---

### 2.8. WPA3-SAE (puro)

Red WPA3 que usa **SAE** (*Simultaneous Authentication of Equals*,
*Dragonfly*).

**Objetivo:** valorar si un ataque de diccionario es viable y, en su caso,
obtener la clave.

**Preguntas:**

1. ¿Por qué el ataque de diccionario **offline** que funciona en WPA2 **no**
   sirve contra SAE?
2. ¿Qué tipo de ataque de diccionario **sí** queda (y por qué es mucho más
   lento)? ¿Qué propiedad aporta SAE aunque la contraseña sea débil?

---

### 2.9. WPA3 en modo transición

El AP ofrece **a la vez** WPA3-SAE y WPA2-PSK para ser compatible con equipos
antiguos.

**Objetivo:** obtener la clave aprovechando la compatibilidad hacia atrás.

**Preguntas:**

1. Si el AP acepta WPA2 además de WPA3, ¿qué parte atacas y por qué vuelve a ser
   aplicable el método de §2.3?
2. ¿De qué sirve desplegar WPA3 si se mantiene el modo transición? ¿Qué
   configuración cierra esta vía?

---

### 2.10. WPA3 *downgrade*

Escenario centrado en un **ataque de degradación** (*Dragonblood*): un AP gemelo
en WPA2 induce al cliente a conectarse con la variante más débil.

**Objetivo:** conseguir que el cliente use WPA2 y capturar el material que
permita el ataque offline.

**Preguntas:**

1. ¿Por qué el cliente acepta "bajar" a WPA2? ¿Qué está comprobando (y qué no)
   antes de asociarse?
2. ¿Qué defensa del lado del cliente (y del AP) evita la degradación?

---

## 3. Entrega

Entrega un informe con, **para cada escenario de tu laboratorio**:

- El **SSID, BSSID y canal** del objetivo en tu despliegue, y el **tipo de
  seguridad** que identificaste en el reconocimiento (§1).
- El **resultado** obtenido (clave, credenciales, identidad/método o PIN).
- Los **comandos** que has ejecutado y **por qué**, con capturas de pantalla que
  muestren el resultado.
- Las **respuestas** a las preguntas del escenario.
- Una **medida defensiva** concreta que habría evitado el ataque.
