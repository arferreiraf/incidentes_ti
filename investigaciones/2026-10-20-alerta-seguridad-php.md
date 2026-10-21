# Investigación de incidente: alerta de seguridad PHP (20–21 octubre 2026)

> **Estado:** evaluación preliminar; no se ha confirmado explotación ni compromiso. Documento elaborado a partir de conversaciones internas; contrastar detalles técnicos con avisos oficiales antes de ejecutar cambios.

## 1. Fuentes y origen
- Aviso oficial PHP: https://www.php.net/index.php#2026-10-20-1
- Publicación de El Hacker Net: https://x.com/elhackernet/status/2112874345466892321
- Una imagen compartida en la conversación alertaba sobre posibles ataques masivos. La previsión de ataques **no equivale a evidencia** de ataques contra la empresa.
- Se discutieron dos recomendaciones de configuración de sesiones PHP, especialmente el seguimiento de progreso de cargas; sus valores exactos y aplicabilidad deben verificarse contra el aviso y el manual de la versión instalada.

## 2. Entorno informado
- Servidor web: **Windows Server 2012 R2 Foundation**, con XAMPP y aproximadamente **cinco aplicaciones web**.
- Stack legado reportado: **PHP 7.0**, MySQL 5.7 y Apache.
- Aplicaciones usadas exclusivamente por personal de la empresa, pero con acceso remoto desde redes y dispositivos diversos, incluidos equipos personales.
- Acceso web expuesto públicamente por HTTP según el diagnóstico previo.
- Al menos **dos o tres aplicaciones permiten subir archivos** (PDF, imágenes o planillas).
- No se informó de indicadores de compromiso, explotación confirmada ni resultado de un escaneo específico de la vulnerabilidad.

## 3. Evaluación inicial
**Prioridad: alta**, por la combinación de exposición pública, sistema operativo y PHP fuera de soporte, y múltiples aplicaciones alojadas en el mismo servidor. La gravedad específica del aviso del 20 de octubre, los identificadores CVE y el alcance exacto en PHP 7.0 requieren verificación independiente: no asumir que un aviso de ramas actuales describe automáticamente la rama instalada.

El uso exclusivo por empleados **no reduce la exposición a Internet** mientras los servicios permanezcan publicados. El acceso remoto desde redes cambiantes dificulta una lista fija de IP permitidas.

## 4. Medidas comentadas
1. **Restringir la exposición externa** con VPN o acceso de identidad controlada; evitar dejar XAMPP/Apache directamente publicado. Considerar alternativas gratuitas y de código abierto compatibles con clientes Windows, Android y otros dispositivos.
2. **Revisar la configuración de sesiones PHP** y el ajuste `session.upload_progress.enabled`. Desactivar este seguimiento si las aplicaciones no dependen de él; la subida ordinaria de archivos mediante `$_FILES` no requiere por sí sola seguimiento de progreso.
3. Revisar también `session.use_strict_mode` y su comportamiento efectivo; no considerarlo una mitigación suficiente por sí solo.
4. **Antes de cambiar producción:** respaldar archivos de configuración, inventariar aplicaciones, buscar dependencias de seguimiento de progreso en el código y ensayar subida de archivos y autenticación en un entorno de prueba.
5. Si se modifica `php.ini`, aplicar los cambios con un reinicio controlado de Apache, tras verificar qué archivo de configuración está cargado.
6. Priorizar plan de migración a versiones soportadas de Windows/PHP y una arquitectura con acceso remoto seguro.

## 5. Verificaciones pendientes
- Identificar versión exacta de PHP, módulos, ruta de `php.ini` y configuración activa.
- Confirmar texto completo, CVE, versiones afectadas y correcciones del comunicado oficial.
- Inspeccionar el código para referencias a `PHP_SESSION_UPLOAD_PROGRESS`, `session.upload_progress` y campos de formularios asociados.
- Identificar si se usa seguimiento de progreso nativo o soluciones JavaScript independientes.
- Comprobar si Apache está expuesto directamente y si hay TLS, autenticación adicional o controles perimetrales.
- Revisar logs de Apache, PHP y Windows en busca de anomalías, conservando evidencias antes de alterarlas.
- Verificar copias de seguridad restaurables y diseñar plan de reversión.

## 6. Plan propuesto
**Inmediato:** respaldos, inventario, verificación de exposición, restricción de acceso público y revisión de logs.

**Corto plazo:** pruebas de ajustes de sesión en staging, pruebas de cargas y autenticación, implantación controlada con reversión.

**Mediano plazo:** migración de PHP/XAMPP y sistema operativo, segmentación de servicios, HTTPS y mecanismo de acceso remoto autenticado.

## 7. Precauciones
- **No desactivar `file_uploads`**: las aplicaciones sí requieren cargar archivos.
- No confundir `session.upload_progress.enabled` con la funcionalidad general de carga.
- No declarar el servidor protegido solo por dos parámetros de `php.ini`.
- No publicar en este repositorio credenciales, IP privadas, registros sensibles ni configuraciones completas sin depurar.
- Este documento **no es un informe forense** ni prueba de intrusión.

## 8. Próximo paso acordado
Determinar si las aplicaciones fueron desarrolladas internamente y revisar si alguna depende del mecanismo nativo de seguimiento de progreso antes de tocar la configuración.
