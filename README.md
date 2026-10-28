# Portal médico DRBOT

Portal web para que el personal médico consulte sus citas, registre la atención y gestione las prescripciones de sus pacientes.

## Funcionalidades

- Inicio y cierre de sesión con Firebase Authentication.
- Consulta de citas asociadas al médico autenticado.
- Filtros por estado y UID del paciente.
- Visualización de datos de la cita, del centro médico y del paciente.
- Registro del diagnóstico principal y del informe de atención. La cita puede cerrarse cuando el paciente ya ha hecho check-in.
- Búsqueda de medicamentos en el catálogo CIMA de la AEMPS.
- Creación, edición y retirada de prescripciones, con conservación del historial. Las prescripciones de una cita cerrada se muestran en modo de solo lectura.

## Requisitos

- Un navegador moderno con conexión a Internet.
- Un servidor web local o un alojamiento HTTPS. El portal carga Firebase como módulos JavaScript y no debe abrirse directamente mediante `file://`.
- Un proyecto Firebase configurado para la aplicación, con Authentication y Cloud Firestore habilitados.
- Una cuenta médica en Firebase Authentication y un documento correspondiente en la colección `Doctors`, con el campo `authUid` igual al UID de esa cuenta.
- Reglas de Firestore que permitan al médico acceder únicamente a sus datos y realizar las operaciones que usa el portal. Las consultas compuestas pueden requerir índices de Firestore.

## Ejecución local

No hay un paso de compilación ni un gestor de dependencias: Firebase se carga desde sus módulos CDN.

Desde la carpeta del proyecto, inicia un servidor estático. Por ejemplo, con Python:

```bash
python -m http.server 8000
```

Abre [http://localhost:8000/doctor_portal.html](http://localhost:8000/doctor_portal.html) en el navegador. Si el comando de Python no está disponible, utiliza cualquier servidor estático equivalente.

## Configuración de Firebase

La configuración de Firebase está declarada en `scripts/script.js`, en el objeto `firebaseConfig`. Sustitúyela por la configuración de la aplicación web del proyecto Firebase correspondiente antes de desplegar el portal.

En Firebase Console:

1. Habilita el proveedor de inicio de sesión por correo y contraseña.
2. Crea las cuentas del personal médico y sus documentos en `Doctors`.
3. Vincula cada documento con la cuenta mediante `authUid`. El portal utiliza el ID del documento de `Doctors` como `doctorId` para buscar las citas.
4. Configura reglas de Firestore y los índices que indiquen los errores de consulta.

El portal utiliza las colecciones `Doctors`, `Slots`, `Queues` y las colecciones de citas y prescripciones asociadas a los pacientes. La estructura y las reglas deben ser compatibles con las consultas de `scripts/script.js`.

## Estructura del proyecto

```text
doctor_portal.html       Página de acceso y panel médico
scripts/script.js        Autenticación, citas, informes y prescripciones
css/styles.css           Estilos
assets/                  Imágenes, fuentes y datos auxiliares
```

## Integración con CIMA

La búsqueda consulta desde el navegador la API de CIMA de la AEMPS. Requiere conexión a Internet y que el navegador pueda realizar la petición al servicio. Si la consulta falla, revisa el mensaje del portal y la consola del navegador.

## Flujo de atención

1. El médico inicia sesión con sus credenciales.
2. El portal busca su perfil en `Doctors` mediante `authUid` y carga las citas asociadas a su ID.
3. Para completar una cita, el paciente debe figurar en sala de espera (`checkedIn`).
4. El médico registra el diagnóstico y el informe; al guardarlos, el portal cierra la cita y actualiza los registros relacionados.
5. Desde la cita también puede buscar un medicamento y guardar la pauta individual del paciente.

## Seguridad

Las credenciales y autorizaciones deben gestionarse en Firebase. Las reglas de Firestore son la protección efectiva de los datos: no confíes en filtros o controles visuales del navegador para restringir el acceso. No publiques credenciales de médicos ni datos clínicos reales en archivos del repositorio.
