# Guía del Desarrollador RM-01

**Leer antes de usar:**

El RM-01 está compuesto por un **Módulo de Inferencia**, un **Módulo de Aplicación** y un **Chip de Cifrado y Gestión** (en adelante, Módulo de Gestión), interconectados a través de un **chip de conmutación Ethernet integrado**, formando una subred LAN interna. Cuando un usuario conecta un host (por ejemplo, PC, smartphone, iPad) a través de la interfaz **USB Type-C**, el RM-01 virtualiza una interfaz Ethernet para el host mediante la funcionalidad USB Ethernet. El host obtiene entonces una dirección IP y se une automáticamente a la subred para el intercambio de datos.

Tras encender el dispositivo y conectarlo al host a través de la interfaz **USB Type-C**, el sistema configurará automáticamente la subred de red local. El **host del usuario** recibirá una dirección IP estática `10.10.99.100`, y el **Chip de Gestión Fuera de Banda** tendrá una dirección IP estática `10.10.99.97`. El **Módulo de Inferencia** (IP: `10.10.99.98`) y el **Módulo de Aplicación** (IP: `10.10.99.99`)—ambos implementan servicios SSH independientes, lo que permite a los usuarios acceder directamente a través de clientes SSH estándar (por ejemplo, OpenSSH, PuTTY). Sin embargo, el Módulo de Gestión requiere acceso a través de una herramienta de puerto serie.

---

## Acerca del Chip de **Gestión Fuera de Banda**

Además de manejar ciertas tareas de cifrado, el Chip de Gestión Fuera de Banda también aloja el **panel de monitoreo de rendimiento del sistema en tiempo real del RM-01** — **RobOS**. Los usuarios pueden acceder a `10.10.99.97` a través de un navegador web para monitorear el **estado de conexión** y el **estado operativo** de cada módulo en tiempo real.

---

## Cómo proporcionar acceso a Internet al RM-01 desde el host (usando macOS como ejemplo)

Después de conectar el **host del usuario** a través de USB Type-C, el RM-01 aparecerá en la lista de interfaces de red como:  
- **`AX88179A`** (Versión para Desarrolladores)  
- **`RMinte RM-01`** (Versión de Lanzamiento Comercial)

Sigue estos pasos para configurar el uso compartido de Internet:

1. Abre **Configuración del Sistema** (System Settings)  
2. Ve a **Red** (Network) → **Compartir** (Sharing)  
3. Habilita **Compartir Internet** (Internet Sharing)  
4. Haz clic en el icono **“i”** junto a la configuración de uso compartido para entrar en la interfaz de configuración:  
   - Configura **“Compartir tu conexión desde”** en: **Wi-Fi**  
   - En **“A computadoras que usen”**, selecciona: **AX88179A** o **RMinte RM-01** (según el modelo del dispositivo)  
5. Haz clic en **Hecho** (Done)

A continuación, regresa a la página de configuración de **Red** (Network) y configura manualmente la interfaz de red del RM-01:  
- **Dirección IP**: `10.10.99.100`  
- **Máscara de subred**: `255.255.255.0`  
- **Router**: `10.10.99.100` (es decir, la IP del propio host)

> **Nota**:  
> Esta configuración establece el host como una puerta de enlace, proporcionando acceso a la red NAT para el RM-01. La puerta de enlace predeterminada y el DNS del RM-01 son asignados automáticamente por el host a través del servicio DHCP. Configurar manualmente la IP asegura que permanezca dentro de la subred `10.10.99.0/24`, consistente con la comunicación de los servicios internos del dispositivo.

---

## Acerca de la tarjeta de almacenamiento **CFexpress Type-B**

La tarjeta de almacenamiento **CFexpress Type-B** es uno de los componentes principales del dispositivo RM-01, responsable del arranque del sistema, el despliegue del marco de inferencia y funciones clave como la distribución de software ISV/SV y la autenticación de autorización.

La tarjeta de almacenamiento está dividida en tres particiones independientes, a saber:

`rm01rootfs` `rm01app` `rm01models`

`rm01rootfs` — Partición del Sistema  
El sistema operativo y el entorno de ejecución principal del **Módulo de Inferencia** están instalados en esta partición. Está estrictamente prohibido que los usuarios o desarrolladores accedan, modifiquen o eliminen el contenido de esta partición. Cualquier cambio no autorizado puede causar que el Módulo de Inferencia no arranque o que las funciones de inferencia queden inoperativas, y cualquier daño de hardware o software resultante no está cubierto por los servicios de garantía.

`rm01app` — Partición de Aplicación  
Esta partición se utiliza para almacenar temporalmente archivos de imagen `Docker` enviados por usuarios o desarrolladores. Una vez que la imagen se escribe en `rm01app`, el sistema RM-01 la migra automáticamente al almacenamiento **NVMe SSD** integrado del host y completa el despliegue contenedorizado. No ejecute ni modifique directamente los archivos de aplicación en esta partición.

`rm01models` — Partición de Modelos  
Esta partición está dedicada a almacenar modelos de inteligencia artificial a gran escala (por ejemplo, LLM, modelos multimodales, etc.) cargados por usuarios o desarrolladores. Para detalles sobre formatos de modelos, limitaciones de tamaño, procedimientos de carga y requisitos de compatibilidad, consulte la sección “Acerca de los Modelos” a continuación.

---

## Acerca del Módulo de Aplicación

Dirección IP: `10.10.99.99`  
Rango de Puertos: `59000-59299`

Especificaciones de Hardware del Módulo de Aplicación:

```
Procesador: Intel Core i3-N305 (8 núcleos, 8 hilos, frecuencia base 1.8 GHz, frecuencia turbo máxima 3.8 GHz)  
Memoria: 16 GB / 24 GB LPDDR5-4800MT/s (integrada, no ampliable)  
Almacenamiento: 512 GB / 1 TB / 2 TB (opcional) NVMe SSD  
```

Credenciales de Acceso SSH del Módulo de Aplicación:

```
Nombre de usuario predeterminado: rm01  
Contraseña predeterminada: rm01 (preconfigurada de fábrica, solo para el primer inicio de sesión)  
```

**Preinstalado:**

El Módulo de Aplicación tiene **Open WebUI** preinstalado en el puerto `80` para facilitar la depuración simple de modelos y trabajo conversacional. Los usuarios pueden acceder navegando a `10.10.99.99` en su navegador web.

**Aviso de Seguridad:**  
Para garantizar la seguridad del sistema, utiliza inmediatamente el comando `passwd` para cambiar la contraseña predeterminada después del primer inicio de sesión SSH. La contraseña predeterminada es solo para la configuración inicial y no debe usarse en entornos de producción o despliegue.

---

## Acerca del Módulo de Inferencia

Dirección IP: `10.10.99.98`  
Rango de Puertos de Servicio: `58000–58999`

El **Módulo de Inferencia** es la unidad de cómputo principal del RM-01, que soporta varias configuraciones de inferencia de IA de alto rendimiento. Los usuarios pueden seleccionar el modelo adecuado según la escala del modelo y los requisitos de rendimiento. Las cuatro configuraciones de hardware preinstaladas de fábrica son las siguientes:

| Memoria | Ancho de Banda de Memoria | Potencia de Cómputo | Cantidad de Tensor Cores |
|---------|--------------------------|:--------------------|:-------------------------|
| 32 GB   | 204.8 GB/s               | 200 TOPS (INT8)     | 56                       |
| 64 GB   | 204.8 GB/s               | 275 TOPS (INT8)     | 64                       |
| 64 GB   | 273 GB/s                 | 1,200 TFLOPS (FP4)  | 64                       |
| 128 GB  | 273 GB/s                 | 2,070 TFLOPS (FP4)  | 96                       |

El RM-01 viene preinstalado con los siguientes dos marcos de inferencia en la tarjeta de almacenamiento **CFexpress Type-B**, ambos ejecutados en el Módulo de Inferencia:

`vLLM`: Se inicia automáticamente, escucha por defecto el puerto **58000**, proporciona interfaces API compatibles con OpenAI y soporta solicitudes estándar como POST /v1/chat/completions.  
`TEI` (Text Embedding Inference): Requiere inicio manual, utilizado para servicios de incrustación de texto.

Método de Acceso API:  
Tras cargar un modelo con éxito, el servicio de inferencia **vLLM** puede ser accedido a través de la siguiente dirección:  
`http://10.10.99.98:58000/v1/chat/completions`  
Soporta llamadas directas usando clientes OpenAI estándar (por ejemplo, openai-python, curl, Postman).

**Aviso de Seguridad:**  
Para garantizar la seguridad y estabilidad del sistema, el Módulo de Inferencia no proporciona permisos de acceso SSH. Los usuarios y desarrolladores no pueden iniciar sesión directamente ni interactuar con el sistema operativo subyacente de este módulo. Cualquier intento de eludir las políticas de seguridad o acceder directamente al Módulo de Inferencia puede resultar en anomalías del sistema, corrupción de datos o interrupciones del servicio, que no están cubiertos por los servicios de garantía.

---

## Acerca de los Modelos

El RM-01 soporta la inferencia de varios modelos de inteligencia artificial, incluyendo, pero no limitado a: **Modelos de Lenguaje a Gran Escala (LLM)**, **Modelos Multimodales (MLM)**, **Modelos de Visión-Lenguaje (VLM)**, **Modelos de Incrustación de Texto (Embedding)** y **Modelos de Reclasificación (Reranker)**. Todos los archivos de modelos deben almacenarse en la tarjeta de almacenamiento **CFexpress Type-B** integrada en el dispositivo, y los usuarios deben usar un lector de tarjetas **CFexpress Type-B** compatible para cargar, gestionar y actualizar modelos desde el lado del host.

Cuando la tarjeta de almacenamiento **CFexpress Type-B** se conecta al RM-01, el sistema la monta como un volumen de datos de solo lectura llamado `models` en la ruta `/home/rm01/models`. Su estructura de archivos estándar es la siguiente:

```bash
models/
├── auto/                  # Directorio para carga automática de modelos (despliegue de grado de producción)
│   ├── embedding/         # Modelos de incrustación (no cargados automáticamente, ver abajo)
│   ├── llm/               # Modelos de lenguaje a gran escala (archivos de pesos almacenados directamente, ver abajo)
│   └── reranker/          # Modelos de reclasificación (no cargados automáticamente, ver abajo)
└── dev/                   # Directorio de modelos definidos por el desarrollador (mayor prioridad)
    ├── embedding/
    ├── llm/
    └── reranker/
```

> **Nota**:  
> - El directorio `auto/` se utiliza para **despliegue de modelos estandarizado y ligero**, reconocido automáticamente por el sistema.  
> - El directorio `dev/` se utiliza para **control detallado del comportamiento de los modelos por parte de los desarrolladores**, con mayor prioridad que `auto/`. El sistema ignorará los modelos en `auto/` si se utiliza `dev/`.

---

### 1. Modo Automático (auto)

#### Uso:  
Coloca los **archivos de pesos completos** del modelo (por ejemplo, `.safetensors`, `.bin`, `.pt`, `.awq`, etc.) **directamente en el directorio `auto/llm/`**, **sin anidar en subcarpetas**.

> Ejemplo incorrecto:  
> `auto/llm/Qwen3-30B-A3B-Instruct-2507-AWQ/model.safetensors`  
>  
> Ejemplo correcto:  
> `auto/llm/model-001-of-006.safetensors`  
> `auto/llm/config.json`  
> `auto/llm/tokenizer.json`  
> `auto/llm/vocab.json`

#### Comportamiento del Sistema:  
- Al iniciar el dispositivo, el sistema escanea el directorio `auto/llm/` y carga automáticamente los modelos en formatos compatibles.  
- **No se admite la carga automática de modelos de incrustación o reclasificación**, solo LLMs.  
- Tras la carga, el modelo habilita **capacidades de inferencia básicas por defecto** y **no habilita las siguientes funciones avanzadas**:  
  - Decodificación Especulativa  
  - Caché de Prefijo  
  - Prellenado por Bloques  
  - La longitud máxima del contexto (`max_model_len`) está restringida al umbral de seguridad del sistema (típicamente ≤ 8192 tokens).  
- **Optimización de rendimiento limitada**: Para garantizar la estabilidad del sistema y la concurrencia multitarea, los modelos en modo automático utilizan una estrategia de asignación de memoria conservadora (`gpu_memory_utilization` ≤ 0.8).

> **Nota Importante**:  
> El modo automático es adecuado para **verificar rápidamente la compatibilidad del modelo** o **escenarios de despliegue estandarizados**, pero **no es adecuado para inferencia de alto rendimiento en entornos de producción**. Para un rendimiento completo, utiliza el **modo manual (dev)**.

---

### 2. Modo Manual (dev) — Modo Desarrollador

> **El modo manual tiene mayor prioridad que el modo automático**. Cuando existe un archivo de configuración `.yaml` en los directorios `embedding`, `llm` o `reranker` bajo `dev/`, el sistema **ignora completamente todos los modelos en `auto/`**.

#### Estructura de los Archivos de Configuración:  
Los directorios `embedding`, `llm` y `reranker` bajo `dev/` deben contener los siguientes tres archivos de configuración YAML para iniciar los servicios de modelos correspondientes:

| Nombre del Archivo    | Propósito                            |
|-----------------------|--------------------------------------|
| `embedding_run.yaml`  | Iniciar el servicio de modelo de incrustación de texto |
| `llm_run.yaml`        | Iniciar el servicio de modelo de lenguaje a gran escala |
| `reranker_run.yaml`   | Iniciar el servicio de modelo de reclasificación |

> Todos los archivos deben estar ubicados en las carpetas correspondientes bajo `dev/`, y **los nombres de los archivos deben coincidir exactamente**, sensibles a mayúsculas y minúsculas.

#### Ejemplo de Archivo de Configuración (`llm_run.yaml`):

```yaml
model: /home/rm01/models/dev/llm/Qwen3-30B-A3B-Instruct-2507-AWQ
port: 58000
gpu_memory_utilization: 0.85
max_model_len: 24576
served_model_name: RM-01 LLM
enable_prefix_caching: true
enable_chunked_prefill: true
max_num_batched_tokens: 512
block_size: 16
tensor_parallel_size: 1
dtype: auto
```

> **Nota**:  
> - La ruta `model` debe ser una **ruta absoluta** que apunte al directorio de los archivos del modelo (no un archivo comprimido).  
> - El `port` debe estar en el rango `58000–58999` y no debe entrar en conflicto con otros servicios.  
> - Se recomienda configurar `gpu_memory_utilization` entre 0.7–0.9 para maximizar el rendimiento.  
> - Todos los parámetros siguen la **documentación oficial de vLLM** `https://docs.vllm.ai/en/latest/index.html`. Asegúrate de la compatibilidad con la versión de vLLM utilizada actualmente en la tarjeta de almacenamiento **CFexpress Type-B**.

#### Proceso de Inicio:  
1. Copia los archivos del modelo (directorio completo) a `dev/llm/` (o `embedding/`, `reranker/`).  
2. Crea y coloca el archivo de configuración `.yaml` correspondiente.  
3. Inserta la tarjeta de almacenamiento **CFexpress Type-B** y reinicia el RM-01.  
4. El sistema cargará automáticamente la configuración en `dev/` e iniciará el servicio correspondiente.  
5. Una vez iniciado el servicio, se puede acceder a través de interfaces como `http://10.10.99.98:58000/v1/chat/completions`.

> **Recomendación**: Para el primer despliegue, utiliza las opciones `--load-format auto` y `--dtype auto` proporcionadas por vLLM para adaptarse automáticamente al formato del modelo.

---

### Notas de Seguridad y Mantenimiento

- **Prohibido iniciar sesión SSH directamente en el Módulo de Inferencia**: Toda la gestión de modelos debe realizarse a través de la tarjeta de almacenamiento **CFexpress Type-B**.  
- **Los archivos de modelo deben ser pesos brutos**: No utilice archivos comprimidos (.zip/.tar.gz), paquetes cifrados o formatos no estándar.  
- **Permisos de Archivos**: Todos los archivos de modelo deben ser legibles (`chmod 644`), y los directorios deben ser ejecutables (`chmod 755`).  
- **Control de Versiones**: Se recomienda usar Git o convenciones de nomenclatura de archivos (por ejemplo, `Qwen3-30B-A3B-Instruct-v1.2-20250930`) para gestionar las versiones de los modelos.  
- **Recomendación de Respaldo**: Haz una copia de seguridad de los directorios `dev/` y `auto/` antes de actualizar los modelos para evitar la pérdida de configuración.

---

### Recomendaciones para la Selección de Modo

| Escenario                        | Modo Recomendado                     |
|--------------------------------|--------------------------------------|
| Verificar rápidamente la compatibilidad del modelo | Modo Automático (auto)               |
| Inferencia de alto rendimiento en producción | Modo Manual (dev) + configuración afinada |
| Despliegue paralelo de múltiples modelos | Modo Manual (dev) + múltiples archivos `.yaml` |
| Depuración de desarrollo, validación de prototipos | Modo Manual (dev)                  |