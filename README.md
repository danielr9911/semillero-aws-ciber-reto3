# Reto 3: Despliegue de control automático de seguridad

## 1. Contexto y objetivo

El Reto 3 del Semillero AWS Ciber representa un paso fundamental hacia la implementación de controles de seguridad automatizados en la nube. En este reto, los participantes aprenderán a diseñar e implementar soluciones que evalúen automáticamente el cumplimiento de políticas de seguridad organizacionales.

### Objetivo general: 
- Desarrollar las competencias fundamentales en AWS de manera práctica, desde el nivel principiante hasta un nivel práctico intermedio, capacitando a los participantes para diseñar, implementar y mantener arquitecturas seguras y eficientes en la nube que cumplan con los estándares y políticas de la organización, preparándolos así para contribuir efectivamente en iniciativas de transformación digital y proyectos de migración dentro de la organización.

### Objetivo específico del Reto 3: 
- Aprender a implementar controles automáticos de seguridad utilizando AWS Config Rules personalizadas
- Desarrollar competencias en la creación de funciones Lambda para evaluación de cumplimiento
- Comprender la importancia del etiquetado de recursos como mecanismo de gobernanza
- Implementar soluciones de monitoreo continuo para garantizar el cumplimiento de políticas organizacionales

## 2. Uso responsable de Cuenta AWS Sandbox

La organización cuenta con un conjunto de cuentas independiente a las productivas para que los equipos realicen pruebas de concepto.

Para el desarrollo del Semillero, utilizaremos la cuenta CiberseguridadSBX. Esta cuenta genera una facturación por uso para la organización, por lo que debemos ser muy responsables en la creación de nuevos recursos, limitándonos a los indicados en el reto.

**Enlace para Consola de AWS SBX:**  
https://d-906705dbfe.awsapps.com/start#/

**Nombramiento de recursos:**  
Con el fin de identificar los recursos que han sido creados en el semillero y realizar una posterior depuración de estos al finalizar, seguiremos el siguiente nombramiento de TODOS los recursos que se creen para cada uno de los retos:

```
semillero-[USUARIO]-[NOMBRE-DEL-RECURSO]
```

Ejemplo:
```
semillero-danirend-control-etiquetado
```

## 3. Necesidad identificada para implementación del reto:

En el entorno corporativo, la correcta identificación y trazabilidad de los recursos de AWS es fundamental para la gestión de riesgos, la respuesta a incidentes y el cumplimiento de políticas de seguridad. Los recursos desplegados en la nube deben poder ser rastreados hasta sus propietarios responsables para garantizar una gestión efectiva en caso de:

- **Incidentes de seguridad**: Identificar rápidamente al responsable del recurso afectado
- **Vulnerabilidades detectadas**: Contactar al equipo apropiado para la remediación
- **Auditorías de cumplimiento**: Demostrar la responsabilidad y ownership de cada recurso
- **Gestión de costos**: Asignar correctamente los gastos a los proyectos y equipos correspondientes
- **Operaciones de mantenimiento**: Coordinar actualizaciones y cambios con los equipos correctos

Para cumplir con estos requerimientos organizacionales, se ha establecido un estándar de etiquetado obligatorio que requiere que **todos los recursos de AWS** contengan tres etiquetas (tags) específicas:

1. **email**: Dirección de correo electrónico del responsable o equipo propietario del recurso
2. **app-code**: Código único que identifica la aplicación o proyecto al que pertenece el recurso
3. **app-name**: Nombre descriptivo de la aplicación o servicio que utiliza el recurso

El objetivo de este reto es implementar un control automático que evalúe continuamente el cumplimiento de esta política de etiquetado en las **instancias EC2**, asegurando que ningún recurso quede sin la identificación apropiada.

### ¿Qué son las etiquetas (Tags) en AWS?

Las etiquetas en AWS son metadatos definidos por el usuario que consisten en pares clave-valor que se pueden asignar a la mayoría de los recursos de AWS. Estas etiquetas permiten:

- Organizar y categorizar recursos
- Implementar políticas de facturación y control de costos
- Aplicar controles de acceso granulares
- Automatizar tareas operativas
- Facilitar la búsqueda y filtrado de recursos

**Documentación oficial**: [Etiquetado de recursos de AWS](https://docs.aws.amazon.com/es_es/general/latest/gr/aws_tagging.html)

### ¿Qué es AWS Config y Config Rules?

AWS Config es un servicio que permite evaluar, auditar y evaluar las configuraciones de los recursos de AWS. Las Config Rules son reglas que representan la configuración deseada para los recursos y evalúan automáticamente si los recursos cumplen con estas configuraciones.

**Documentación oficial**: [AWS Config Rules](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config.html)

# Guía para la implementación del control automático de seguridad

Para implementar la solución de control automático de etiquetado, deberás crear un template de CloudFormation que incluya:

1. **Una función Lambda** que evalúe si las instancias EC2 tienen las tres etiquetas obligatorias (email, app-code, app-name)
2. **Una Config Rule personalizada** que utilice la función Lambda para realizar las evaluaciones
3. **Los roles y permisos IAM** necesarios para que los servicios funcionen correctamente

> **Nota importante**: AWS Config ya está habilitado y configurado en la cuenta sandbox.

## Recursos requeridos en tu template CloudFormation:

- `AWS::Lambda::Function` - Función que evalúa el cumplimiento de etiquetado
- `AWS::IAM::Role` - Rol de ejecución para la función Lambda
- `AWS::Lambda::Permission` - Permiso para que Config invoque la función Lambda
- `AWS::Config::ConfigRule` - Regla personalizada que usa la función Lambda

## Validación del funcionamiento

Una vez desplegado tu stack, deberás validar que la Config Rule funciona correctamente:

### Paso 1: Crear instancias de prueba

Crea dos instancias EC2 para probar tu Config Rule:

1. **Instancia no conforme**: Crea una instancia EC2 usando el nombre `semillero-[USUARIO]-instancia-no-conforme` **sin agregar** las etiquetas obligatorias
2. **Instancia conforme**: Crea una instancia EC2 usando el nombre `semillero-[USUARIO]-instancia-conforme` **con** las tres etiquetas obligatorias:
   - `email`: tu-email@bancolombia.com.co
   - `app-code`: RETO3-TEST
   - `app-name`: Instancia de prueba reto 3

### Paso 2: Verificar evaluaciones en AWS Config

Para verificar que tu Config Rule está funcionando correctamente:

1. **Acceder a AWS Config**:
   - En la consola de AWS, busca "Config" y accede al servicio
   - En el menú lateral izquierdo, haz clic en "Rules"

2. **Localizar tu Config Rule**:
   - Busca la regla con el nombre que definiste en tu template (debe incluir tu prefijo semillero-[USUARIO])
   - Haz clic en el nombre de la regla

3. **Revisar el estado de cumplimiento**:
   - En la sección "Compliance" podrás ver el resumen de recursos evaluados
   - Debe mostrar al menos 2 recursos evaluados
   - Deberías ver 1 recurso "Compliant" y 1 recurso "Noncompliant"

4. **Verificar recursos específicos**:
   - En la pestaña "Resources in scope" podrás ver todas las instancias EC2 evaluadas
   - Haz clic en cada instancia para ver los detalles de la evaluación
   - Confirma que la instancia sin etiquetas aparece como "Noncompliant"
   - Confirma que la instancia con etiquetas aparece como "Compliant"

### Paso 3: Tomar capturas de pantalla como evidencia

Deberás capturar:
- Vista general de la Config Rule mostrando el resumen de cumplimiento
- Lista de recursos evaluados con sus estados de cumplimiento
- Detalle de la evaluación de la instancia no conforme
- Detalle de la evaluación de la instancia conforme

# Tareas opcionales adicionales (Puntos extras)

Para llevar el control de seguridad al siguiente nivel, puedes implementar **remediación automática** de los hallazgos de incumplimiento. Dado que todos los participantes trabajan en la misma cuenta AWS, la remediación automática debe ser **específica para tus recursos únicamente**.

### Remediación automática personalizada

La remediación automática debe funcionar **solo con los recursos que tú hayas creado** para evitar interferencias entre participantes.

**Implementación sugerida:**

1. **EventBridge Rule** que se active cuando Config detecte un recurso NON_COMPLIANT de tu Config Rule específica
2. **Función Lambda de remediación** que:
   - **Filtre solo instancias EC2 que contengan tu prefijo** `semillero-[USUARIO]` en el nombre o ID
   - Identifique las etiquetas faltantes en la instancia EC2
   - Aplique etiquetas con valores predeterminados específicos para ti:
     - `email`: "[USUARIO]@bancolombia.com.co"
     - `app-code`: "RETO3-AUTO-[USUARIO]"
     - `app-name`: "Auto-remediado por [USUARIO]"
   - Registre la acción en CloudWatch Logs

**Consideraciones importantes para el entorno compartido:**
- **Filtro obligatorio**: La Lambda debe validar que el recurso pertenece a tu usuario antes de realizar cualquier modificación
- **Nombres únicos**: Usar tu prefijo de usuario en todos los nombres de recursos
- **Logs diferenciados**: Crear grupos de logs específicos con tu prefijo
- **Permisos limitados**: La función solo debe tener permisos sobre recursos con tu prefijo

**Ejemplo de filtro en la función Lambda:**
```python
# Verificar que el recurso pertenece al usuario actual
if 'semillero-[USUARIO]' not in instance_name and 'semillero-[USUARIO]' not in instance_id:
    print(f"Instancia {instance_id} no pertenece a este usuario, omitiendo remediación")
    return
```

# Sistema de puntuación

La evaluación del Reto 3 se realizará de acuerdo con el siguiente sistema de puntuación:

## Implementación del control automático (80 puntos)

### Función Lambda de evaluación (40 puntos)
- AWS::Lambda::Function correctamente definida (15 puntos)
- AWS::IAM::Role para Lambda con permisos apropiados (15 puntos)
- AWS::Lambda::Permission para Config (5 puntos)
- Código Python que evalúa correctamente las etiquetas (5 puntos)

### Config Rule personalizada (40 puntos)
- AWS::Config::ConfigRule correctamente configurada (20 puntos)
- Config Rule evalúa correctamente instancias conformes (10 puntos)
- Config Rule detecta instancias no conformes (10 puntos)

## Funcionamiento del control (20 puntos)
- Despliegue exitoso del stack (5 puntos)
- Template CloudFormation bien estructurado (5 puntos)
- Evidencias de evaluación correcta de recursos (5 puntos)
- Documentación clara de la implementación (5 puntos)

## Tareas opcionales - Remediación automática personalizada (20 puntos)
- Implementación de EventBridge Rule para detección de incumplimientos (5 puntos)
- Función Lambda de remediación que funciona solo con recursos propios (10 puntos)
- Documentación y pruebas de la remediación sin afectar otros participantes (5 puntos)

## Registro de avance en Planner

Para registrar tu avance, deberás subir:

1. **Template CloudFormation desarrollado**
   - Archivo .yaml completo
   - Pantallazo de la validación exitosa del template en CloudFormation

2. **Despliegue exitoso**
   - Captura del stack con estado CREATE_COMPLETE
   - Captura de los recursos creados en el stack

3. **Control funcionando correctamente**
   - Captura de tu Config Rule mostrando el resumen de evaluaciones
   - Captura de instancias EC2 evaluadas mostrando estados conformes y no conformes
   - Captura del detalle de evaluación de una instancia no conforme
   - Captura del detalle de evaluación de una instancia conforme

4. **Remediación automática personalizada (opcional)**
   - Template CloudFormation actualizado con recursos de remediación
   - Código de la función Lambda de remediación con filtros por usuario
   - Evidencia de funcionamiento de la remediación solo en tus recursos
   - Logs de CloudWatch mostrando las acciones de remediación

¡Buena suerte implementando tu primer control automático de seguridad en AWS!

# Guía para la eliminación de recursos

Al finalizar el reto, es importante eliminar todos los recursos creados para evitar costos innecesarios y mantener limpia la cuenta AWS.

## Eliminar instancias EC2 de prueba

Antes de eliminar el stack de CloudFormation, elimina las instancias EC2 que creaste para las pruebas:

1. Accede a la consola de EC2
2. Selecciona las instancias que creaste para el reto
3. Haz clic en "Instance State" > "Terminate instance"
4. Confirma la terminación

## Eliminar el stack completo de CloudFormation

1. Accede a la consola de CloudFormation 
2. Selecciona tu stack semillero-[USUARIO]-control-etiquetado 
3. Haz clic en "Eliminar" 
4. Confirma la eliminación

CloudFormation eliminará automáticamente todos los recursos que fueron creados por el stack en el orden correcto.

## Verificación de eliminación completa

1. Espera a que el estado del stack cambie a "DELETE_COMPLETE" 
2. Si hay errores en la eliminación, revisa los eventos para identificar qué recursos no se pudieron eliminar 
3. Para recursos que no se eliminaron automáticamente, elimínalos manualmente

## Verificación final

Utiliza la barra de búsqueda global con el prefijo semillero-[USUARIO] para asegurarte de que no queden recursos sin eliminar.

> Importante: La eliminación de estos recursos es definitiva y los datos no se podrán recuperar.
