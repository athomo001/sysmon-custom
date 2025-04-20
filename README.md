# sysmon-custom
# 🛡️ Sysmon Config: Detección Mejorada (Foco en IDs 1, 11, 12, 13)

[![Detection](https://img.shields.io/badge/focus-Sysmon%20IDs%201,11,12,13-blue.svg)](https://shields.io/)

## Descripción

Este repositorio contiene una configuración XML para Sysmon, basada en Olaf Hartong, pero **modificada** añadiendo o ajustando reglas específicamente para los **Event IDs 1, 11, 12 y 13**.

El objetivo sigue siendo mejorar la detección de actividades potencialmente maliciosas o de alto riesgo, como las que ocurren tras un **inicio de sesión administrativo** o un intento de **elevación de privilegios**, utilizando la información detallada que proporcionan estos eventos específicos de Sysmon.

Estos eventos están pensados para ser recolectados y analizados por sistemas HIDS (como Wazuh), SIEMs o EDRs.

## Motivación

Si bien la configuración base es excelente, estas modificaciones buscan afinar la detección concentrándose en eventos clave que a menudo revelan actividades maliciosas o abusos de privilegios, facilitando su análisis en plataformas de seguridad centralizadas.

## Fuente Original y Agradecimientos

**MUY IMPORTANTE:** Esta configuración es una **modificación** y se basa fundamentalmente en el gran trabajo realizado por Olaf Hartong.

* **Repositorio Original:** https://github.com/olafhartong/sysmon-modular

Se recomienda encarecidamente revisar y entender la configuración base original antes de aplicar estas modificaciones. ¡Todo el crédito por la base es para el autor original!

## Entendiendo los Eventos de Sysmon Enfocados (IDs 1, 11, 12, 13)

Para comprender el propósito de las modificaciones, es útil saber qué registra cada uno de estos Event IDs clave de Sysmon:

* **Event ID 1: Process Creation (Creación de Proceso)**
    * **¿Qué registra?:** Cada vez que se inicia un nuevo proceso en el sistema.
    * **¿Por qué es importante?:** Es fundamental. Permite ver qué programas se están ejecutando, quién los ejecuta (contexto de usuario), con qué argumentos (línea de comandos completa) y cuál fue el proceso padre que lo lanzó. Esencial para detectar la ejecución de malware, herramientas de hacking, scripts sospechosos, y comandos utilizados por atacantes durante el reconocimiento o la post-explotación.

* **Event ID 11: File Creation (Creación de Archivo)**
    * **¿Qué registra?:** Cuando un archivo es creado o sobrescrito por un proceso. Técnicamente, el evento se genera cuando se cierra el "handle" del archivo después de una operación de escritura.
    * **¿Por qué es importante?:** Crucial para detectar cuándo un malware o un atacante "dropea" (descarga/crea) archivos en el sistema, como ejecutables maliciosos, scripts, cargas útiles (payloads), o notas de rescate de ransomware. Permite ver qué proceso creó qué archivo y dónde.

* **Event ID 12: Registry Event (Object Create and Delete - Creación/Eliminación de Clave de Registro)**
    * **¿Qué registra?:** La creación o eliminación de claves completas dentro del Registro de Windows.
    * **¿Por qué es importante?:** El Registro es un lugar primordial para técnicas de **persistencia** (asegurar que el malware se ejecute al reiniciar) y **configuración/evasión**. Este evento ayuda a detectar cuándo se crean claves para ejecutar programas automáticamente (ej. en `Run`, `Services`) o cuándo se eliminan claves para intentar ocultar rastros o deshabilitar configuraciones de seguridad.

* **Event ID 13: Registry Event (Value Set - Establecimiento de Valor de Registro)**
    * **¿Qué registra?:** La modificación o creación de valores específicos *dentro* de las claves del Registro.
    * **¿Por qué es importante?:** Complementa al ID 12. Muchas técnicas de persistencia, evasión y configuración maliciosa implican modificar valores existentes o crear nuevos valores dentro de claves legítimas (ej. cambiar el valor de `Shell` en `Winlogon`, añadir una ruta a un malware en una clave `Run`, deshabilitar UAC o Windows Defender modificando un valor DWORD).

Comprender qué información provee cada uno de estos eventos ayuda a entender por qué ajustar las reglas para ellos puede mejorar significativamente la visibilidad sobre actividades sospechosas.

## Características Principales de las Modificaciones (Foco en IDs 1, 11, 12, 13)

Basándose en la importancia de los eventos descritos arriba, las adiciones o ajustes principales realizados sobre la configuración base se centran en:

* **Event ID 1 (Process Creation):**
    * Se añadieron/ajustaron reglas para detectar la **creación de procesos** específicos que podrían indicar el uso de herramientas de hacking, la ejecución de scripts sospechosos (PowerShell ofuscado, `cscript`, `wscript`, etc.) o acciones comunes post-escalada.
    * Se busca identificar comandos o ejecutables (`whoami`, `net user`, `nltest`, etc.) usados frecuentemente durante el reconocimiento interno o por malware.

* **Event ID 11 (File Creation):**
    * Nuevas reglas o modificaciones para monitorear la **creación de archivos ejecutables (.exe, .dll, .scr, etc.) o scripts (.ps1, .vbs, .bat)** en ubicaciones inusuales (ej. `C:\Users\`, `C:\Windows\Temp\`, directorios de perfil).
    * Puede ayudar a detectar la descarga o el "dropping" de malware, ransomware, o herramientas de atacantes.

* **Event ID 12 (Registry Object Create and Delete) & Event ID 13 (Registry Value Set):**
    * Se incorporaron reglas para vigilar **cambios en claves y valores del Registro de Windows** que son comúnmente utilizados para:
        * **Persistencia:** (Ej. `Run`, `RunOnce`, `Services`, `Scheduled Tasks`, `Startup Approved`, `Winlogon` keys).
        * **Evasión de Defensas:** (Ej. Deshabilitar LSA Protection, modificar configuraciones de Windows Defender, UAC, Firewall).
        * **Actividad de Malware:** (Ej. Claves específicas usadas por familias de malware conocidas).

* **Filtrado/Exclusiones:** Posiblemente se ajustaron o añadieron exclusiones relacionadas con estos IDs para reducir el ruido de procesos y cambios legítimos conocidos (¡requiere tuning!).

## Requisitos

* **Sysmon:** Debe estar instalado en los endpoints Windows. [Descarga aquí](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).
* **Permisos:** Privilegios de Administrador para aplicar la configuración.
* **Sistema de Recolección de Logs:** Un HIDS, SIEM o EDR configurado para recolectar los eventos `Microsoft-Windows-Sysmon/Operational`.

## 🚀 Instalación / Aplicación

*(Esta sección permanece igual que en la respuesta anterior, con los pasos detallados de descarga, preparación, aplicación y verificación)*
Sigue estos pasos para instalar Sysmon (si aún no lo has hecho) y aplicar esta configuración personalizada:

**Paso 1: Descargar Sysmon**

* Descarga la última versión de Sysmon directamente desde la página oficial de Microsoft Sysinternals:
    * [**https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon**](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)

**Paso 2: Preparar los Archivos**

* Descomprime el archivo ZIP de Sysmon que descargaste. Encontrarás `Sysmon.exe` (para sistemas de 32 bits) y `Sysmon64.exe` (para sistemas de 64 bits).
* Descarga el archivo de configuración XML modificado de **este** repositorio (ej. `tu_configuracion_modificada.xml`).
* **Recomendación:** Crea una carpeta dedicada, por ejemplo `C:\SysmonTools`, y copia **tanto el ejecutable de Sysmon** (usa `Sysmon64.exe` si tienes un Windows de 64 bits, que es lo más común) **como tu archivo XML** (`tu_configuracion_modificada.xml`) dentro de esa carpeta.

**Paso 3: Abrir Consola como Administrador**

* Haz clic derecho en el menú Inicio de Windows y selecciona "**Terminal (Administrador)**", "**Windows PowerShell (Administrador)**" o "**Símbolo del sistema (Administrador)**". Es **fundamental** ejecutar la consola con privilegios elevados.

**Paso 4: Navegar al Directorio**

* En la consola de administrador que abriste, navega hasta la carpeta donde guardaste los archivos en el Paso 2. Ejemplo:
    ```powershell
    cd C:\SysmonTools
    ```

**Paso 5: Instalar Sysmon y/o Aplicar la Configuración**

* Ahora, ejecuta **uno** de los siguientes comandos, dependiendo de tu situación:

    * **A) Si es la PRIMERA VEZ que instalas Sysmon en esta máquina:**
        * Este comando instalará Sysmon usando tu archivo de configuración y aceptará el acuerdo de licencia automáticamente.
        ```powershell
        .\Sysmon64.exe -i tu_configuracion_modificada.xml -accepteula
        ```
        * *(Nota: Si estás en un sistema de 32 bits, usa `Sysmon.exe` en lugar de `Sysmon64.exe`)*

    * **B) Si Sysmon YA ESTÁ INSTALADO y solo quieres ACTUALIZAR la configuración:**
        * Este comando aplicará tu nuevo archivo de configuración a la instalación existente de Sysmon.
        ```powershell
        .\Sysmon64.exe -c tu_configuracion_modificada.xml
        ```
        * *(Nota: Usa `Sysmon64.exe` o `Sysmon.exe` según corresponda a tu sistema)*

**Paso 6: Verificar la Instalación/Actualización**

* Abre el **Visor de Eventos** de Windows (busca `eventvwr.msc`).
* Navega a `Registros de aplicaciones y servicios` -> `Microsoft` -> `Windows` -> `Sysmon` -> `Operational`.
* Deberías empezar a ver eventos de Sysmon (IDs 1, 11, 12, 13, etc.).
* Busca un evento con **ID 255 (Configuration change)** o **ID 16 (Sysmon config state changed)** que indica que tu archivo de configuración XML fue cargado correctamente.


## Integración con HIDS/SIEM

El valor real se obtiene al enviar estos eventos (especialmente IDs 1, 11, 12, 13) a tu plataforma de seguridad para:
* Crear **reglas de correlación** (Ej: "Alerta si se crea un archivo .exe en `C:\Users\` Y el proceso padre es `powershell.exe`").
* Generar **alertas** basadas en patrones o eventos específicos.
* **Investigar** incidentes correlacionando estos eventos con otros logs.

## Consideraciones Importantes

* **TUNING ES NECESARIO:** Fundamental. Esta configuración **necesitará ajustes** (más exclusiones) para tu entorno específico.
* **Impacto en Rendimiento:** Monitoriza el rendimiento del endpoint.
* **Evasión:** Sysmon no es infalible.
* **Actualizaciones:** Mantén Sysmon y la configuración al día.

## Licencia

Ciclista sin Licencia
## Descargo de Responsabilidad

Configuración proporcionada "tal cual". **Úsala bajo tu propio riesgo.** Prueba exhaustivamente antes de desplegar en producción. El autor de estas modificaciones no se responsabiliza por problemas derivados de su uso.

---
*README actualizado el domingo, 20 de abril de 2025.*
