# SOC Investigation #01 — Unusual Listening Port

## Objetivo

Investigar una alerta simulada relacionada con un puerto TCP inusual en estado `LISTEN` dentro de un sistema Linux.

El objetivo del laboratorio fue identificar el proceso responsable, analizar el servicio asociado, recopilar evidencia y determinar si existía información suficiente para clasificar la actividad como maliciosa.

> **Environment:** Kali Linux Home Lab  
> **Alert:** Unusual Listening Port  
> **Port:** 9000/TCP  
> **Severity:** Medium  
> **Final Classification:** Suspicious Activity — Further Investigation Required

---

## Alerta inicial

Se detectó un servicio escuchando en el puerto TCP 9000.

La alerta por sí sola no fue considerada un incidente confirmado, por lo que se inició una investigación para determinar qué proceso estaba utilizando el puerto.

---

## 1. Validación del puerto

Se utilizó:

```bash
ss -tln
```

El resultado confirmó que el puerto TCP `9000` se encontraba en estado:

```text
LISTEN
```

Esto confirmó la alerta inicial, pero todavía no permitía determinar si la actividad era legítima o maliciosa.

---

## 2. Identificación del proceso

Se utilizó:

```bash
sudo ss -tlnp
```

Se identificó:

```text
Process: python3
PID: 8078
FD: 3
Local Address: 0.0.0.0:9000
State: LISTEN
```

El proceso se encontraba enlazado a `0.0.0.0:9000`, indicando que estaba escuchando en las interfaces IPv4 disponibles del sistema.

---

## 3. Análisis del proceso

Con el PID identificado se ejecutó:

```bash
ps -fp 8078
```

Se encontró:

```text
User: kali
PID: 8078
Start Time: 07:26
Command: python3 -m http.server 9000
```

Esto permitió identificar que el proceso correspondía a un servidor HTTP ejecutado mediante Python.

---

## 4. Directorio de trabajo

Para determinar desde qué directorio se estaba ejecutando el proceso:

```bash
readlink /proc/8078/cwd
```

Resultado:

```text
/home/kali/Documents/soc-portafolio/01-network-investigation
```

Esto permitió obtener evidencia directa del directorio de trabajo utilizado por el proceso.

---

## 5. Análisis del servicio HTTP

Se realizó una petición al servicio:

```bash
curl http://127.0.0.1:9000
```

La respuesta mostró un listado de archivos que incluía:

```text
investigation.txt
```

Posteriormente se comprobó que el archivo podía ser recuperado mediante HTTP.

El archivo no contenía información sensible.

---

## Hallazgos

Durante la investigación se determinó que:

- El puerto `9000/TCP` estaba en estado `LISTEN`.
- El proceso responsable era `python3`.
- El proceso estaba ejecutando `python3 -m http.server 9000`.
- El servicio estaba enlazado a `0.0.0.0:9000`.
- Se identificó el directorio de trabajo del proceso.
- El servidor permitía visualizar y recuperar archivos mediante HTTP.
- No se encontró evidencia suficiente para confirmar actividad maliciosa.
- No se observaron conexiones activas de clientes durante la revisión.

---

## Clasificación

**Suspicious Activity — Further Investigation Required**

La existencia de un servicio HTTP inesperado y la exposición de archivos justifican continuar la investigación.

Sin embargo, la evidencia recopilada no permite clasificar la actividad como un ataque confirmado.

En un entorno real sería necesario validar si el servicio estaba autorizado y revisar fuentes de información histórica como logs y telemetría de red.

---

## Comandos utilizados

```bash
ss -tln
sudo ss -tlnp
ps -fp 8078
readlink /proc/8078/cwd
curl http://127.0.0.1:9000
```

---

## Conclusión

Este laboratorio permitió practicar un flujo básico de investigación SOC:

**Alert → Validate → Identify → Investigate → Analyze → Classify**

El principal aprendizaje fue que una alerta no debe considerarse automáticamente un incidente de seguridad.

Las conclusiones deben estar respaldadas por evidencia y, cuando la información disponible no sea suficiente, la investigación debe continuar antes de determinar si una actividad es legítima o maliciosa.

---

## Disclaimer

This investigation was performed in a controlled home laboratory for educational purposes. No external or unauthorized systems were targeted.
