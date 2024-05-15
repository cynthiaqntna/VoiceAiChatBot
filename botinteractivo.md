---
title: VoiceAIChatBot
author: Cynthia Quintana Reyes
---
<h1 style="color: #004286; font-family: 'Arial', sans-serif;">Documentación del Bot de WhatsApp para Intercambio de Mensajes de Audio</h1>



Este documento proporciona una guía detallada para recrear y entender el bot de WhatsApp diseñado para intercambiar mensajes de audio. Utilizando Landbot y Make, este bot innovador ofrece una experiencia interactiva donde los usuarios envían un mensaje de audio y reciben una respuesta automática en formato de audio. Este proceso se logra mediante la integración de tecnologías avanzadas de procesamiento de voz y texto, incluyendo Deepgram para la transcripción de voz, OpenAI para la generación de texto y la conversión de texto a voz (TTS), y Google Cloud Storage para el almacenamiento eficiente de archivos de audio. El objetivo de este manual es proporcionar un enfoque paso a paso que detalle cada componente y proceso, facilitando la replicación o adaptación de esta solución para satisfacer diversas necesidades.

## Requisitos Previos

Antes de comenzar la configuración del bot de WhatsApp para el intercambio de mensajes de audio, es esencial asegurarse de que tienes acceso a las siguientes cuentas y herramientas, y que dispones de las credenciales necesarias:

1. **Cuenta de Landbot**
   - **Descripción**: Necesitarás una cuenta en Landbot para crear y administrar el bot interactivo.
   - **Creación de Cuenta**: Puedes registrarte y crear una cuenta en [Landbot](https://landbot.io).

2. **Cuenta de Deepgram**
   - **Descripción**: Deepgram se utiliza para la transcripción de audio. Deberás tener una cuenta para acceder a sus servicios de API.
   - **API Key**: Asegúrate de tener la API key de Deepgram accesible, la cual se utiliza para autenticar las solicitudes a su API.
   - **Creación de Cuenta y Obtención de API Key**: Regístrate y obtén una API key en [Deepgram](https://www.deepgram.com).

3. **API Key de OpenAI**
   - **Descripción**: OpenAI se utiliza para la generación de texto y la conversión de texto a voz.
   - **API Key**: Necesitarás una API key de OpenAI para hacer solicitudes a sus modelos de inteligencia artificial.
   - **Obtención de API Key**: Puedes obtener la API key registrándote y configurando tu acceso en [OpenAI](https://www.openai.com).

4. **Cuenta de Make (anteriormente Integromat)**
   - **Descripción**: Make se utiliza para integrar y automatizar flujos de trabajo entre diferentes servicios, como Landbot, OpenAI y Google Cloud Storage.
   - **Creación de Cuenta**: Crea una cuenta en [Make](https://www.make.com).

5. **Cuenta de Google Cloud Storage**
   - **Descripción**: Google Cloud Storage se utiliza para almacenar y gestionar los archivos de audio generados y utilizados por el bot.
   - **Creación de Cuenta**: Regístrate y configura tu almacenamiento en [Google Cloud](https://cloud.google.com/storage).
   - **Configuración de Permisos**: Asegúrate de configurar los permisos adecuados para acceder y manipular los objetos almacenados en tus buckets de Google Cloud Storage.

Asegúrate de guardar de manera segura todas las API keys y credenciales, ya que serán necesarias para configurar correctamente las integraciones y los flujos de trabajo del bot.

## Creación del Bot en Landbot

### - Acceso y Creación
1. **Iniciar Sesión**: Accede a tu cuenta en Landbot mediante la página [Landbot.io](https://landbot.io).
2. **Crear un Nuevo Bot**:
   - En el dashboard de Landbot, selecciona **"Create a Bot"**.
   - Escoge **"WhatsApp"** como la plataforma para tu bot.
   - Elige **"Start from scratch"** para iniciar la construcción de tu bot desde cero.

### - Configuración del Primer Bloque: Solicitar un Archivo de Audio

Una vez creado el espacio de trabajo para tu bot, el primer paso es añadir un bloque que solicite un archivo de audio al usuario.

#### Configuración del Bloque "Ask for a File"

- **Pregunta (Question text)**: "Hola, ¿me podrías pasar un audio para ayudarte?"
   - Este texto actúa como el mensaje inicial que ve el usuario, invitándolo a enviar un archivo de audio para procesar su solicitud.

- **Mensaje de Error de Validación (Validation error message)**: 
   - "I'm afraid I didn't understand, could you try again, please?"
   - Este mensaje guía al usuario para enviar el archivo correctamente si el primer intento falla.

- **Guardar la Respuesta (Save answers in the variable)**:
   - `@input_audio`
   - Esta variable almacena el archivo de audio enviado por el usuario, facilitando su acceso en los bloques siguientes del flujo de conversación.

![Bloque Ask For a File](https://i.imgur.com/qMPd1fF.png)

### - Configuración del Webhook para Transcripción de Audio con Deepgram

#### Descripción General
Este bloque se encarga de enviar el archivo de audio recibido del usuario a Deepgram para su transcripción mediante un webhook. La respuesta de Deepgram será utilizada posteriormente para generar una respuesta textual o audiovisual a través de OpenAI.

#### Configuración del Webhook

1. **URL y Método**
   - **Método**: POST
   - **URL**: `https://api.deepgram.com/v1/listen`
     - Asegúrate de incluir los parámetros adecuados en la URL, como el modelo y el idioma que deseas utilizar para la transcripción.
     - Ejemplo completo de URL: `https://api.deepgram.com/v1/listen?model=nova-2&language=es&smart_format=true`
   
2. **Headers**
   - **Content-Type**: `application/json` - Especifica que el cuerpo de la solicitud está en formato JSON.
   - **Authorization**: `Token tu_token_de_api` - Utiliza tu API key de Deepgram. Asegúrate de reemplazar `tu_token_de_api` con tu token real.

3. **Customizar el Cuerpo de la Solicitud**
   - **Body**: El cuerpo de la solicitud debe contener la URL del archivo de audio que se transcribirá.
   ```json
   {
     "url": "@input_audio"
   }

##### Nota sobre Variables
- **Nota**: `@input_audio` es una variable que contiene la URL del archivo de audio enviado por el usuario. **Asegúrate de que esta variable esté correctamente asignada en tu flujo de Landbot antes de usarla aquí.**

4. **Guardar la Respuesta**
- **Variable**: `@deepgram_response`
  - **Guarda toda la respuesta del cuerpo en la variable `@deepgram_response` para su uso posterior en el flujo del bot.**

##### Nota Importante
- **Asegúrate de que `@deepgram_response` esté configurada para capturar y almacenar la respuesta completa para un manejo adecuado de los datos recibidos.**

![Deepgram 1](https://i.imgur.com/OIdkgRz.png)
![Deepgram 2](https://i.imgur.com/degstSG.png)
![Deepgram 3](https://i.imgur.com/gViuriP.png)


### - Configuración del Bloque de Fórmulas para Extraer Transcripción

#### Descripción General
Este bloque de fórmulas se utiliza para extraer la transcripción del audio enviado por el usuario, que ha sido procesada por Deepgram. Utilizando la respuesta de la API de Deepgram, este bloque extrae la transcripción de texto específicamente.

#### Configuración del Bloque de Fórmulas

1. **Objetivo del Bloque**:
   - El bloque de fórmulas está configurado para extraer el texto transcrito del audio recibido y almacenar este texto en una variable para su uso posterior en el flujo del bot.

2. **Configuración Específica**:
   - **Output**: `@audio_in_text`
     - Esta variable almacenará el resultado de la fórmula, que es el texto transcrito del audio.
   - **Fórmula Usada**:
    
     `GetValue(@deepgram_response, "results", "channels", 0, "alternatives", 0, "transcript")`
     
     - Esta fórmula navega a través de la estructura JSON de la respuesta de Deepgram para extraer la transcripción exacta del audio enviado por el usuario.

3. **Uso de la Fórmula**:
   - La función `GetValue` es una herramienta poderosa en Landbot que permite acceder a elementos específicos dentro de una respuesta JSON estructurada. En este caso, se usa para localizar el texto exacto dentro de la respuesta de la API de Deepgram.

#### Más Información
- Para más detalles sobre cómo utilizar y configurar fórmulas en Landbot, visita la [documentación oficial de Landbot sobre fórmulas](https://help.landbot.io/article/zh7wqs85mb-formulas-array-new).

![Fórmulas Deepgram 1](https://i.imgur.com/rOtcZiD.png)
![Fórmulas Deepgram 2](https://i.imgur.com/tfievZv.png)

### - Configuración del Webhook para Generar Respuestas con OpenAI

#### Descripción General
Este bloque configura un webhook que envía la transcripción de texto extraída del audio a la API de OpenAI para generar respuestas inteligentes basadas en el contenido del audio.

#### Configuración del Webhook

1. **URL y Método**
   - **Método**: POST
   - **URL**: `https://api.openai.com/v1/chat/completions`
   - Este endpoint permite interactuar con los modelos de lenguaje de OpenAI, enviando preguntas o declaraciones y recibiendo respuestas generadas.

2. **Headers de la Solicitud**
   - **Content-Type**: `application/json` - Indica que el cuerpo de la solicitud está en formato JSON.
   - **Authorization**: `Bearer sk-yourapikeyhere` - Utiliza tu API key de OpenAI. Reemplaza `sk-yourapikeyhere` con tu token real de API.

3. **Cuerpo de la Solicitud**
   - **Modelo**: `gpt-4-turbo-preview` - Especifica el modelo de OpenAI que se utilizará para generar la respuesta.
   - **Mensajes**:
     ```json
     [
       {
         "role": "user",
         "content": "@audio_in_text"
       }
     ]
     ```
     - `@audio_in_text` debe ser la variable que contiene la transcripción del audio que se envía a OpenAI.

4. **Guardar la Respuesta**
   - **Variable**: `@json_chat_response`
   - Configura la respuesta completa del cuerpo del webhook para ser guardada en esta variable, permitiendo su uso posterior en el flujo del bot.

#### Uso del Webhook

- Este webhook transforma la entrada textual en respuestas conversacionales, aprovechando las capacidades de los modelos avanzados de lenguaje de OpenAI.
- Asegúrate de probar el webhook para confirmar que la integración funciona correctamente y que las respuestas son las esperadas.

#### Más Información
- Para más detalles sobre la API de OpenAI y cómo interactuar con diferentes modelos, puedes consultar la [documentación oficial de OpenAI](https://beta.openai.com/docs/).

![OpenAI 1](https://i.imgur.com/f2nqxJC.png)
![OpenAI 2](https://i.imgur.com/BSx52um.png)
![OpenAI 3](https://i.imgur.com/pIHDABY.png)
![OpenAI 4](https://i.imgur.com/EMMhMJW.png)

### - Configuración del Bloque de Fórmulas para Extraer la Respuesta de OpenAI

#### Descripción General
Este bloque de fórmulas está diseñado para extraer la respuesta generada por OpenAI a partir de la respuesta JSON obtenida a través del webhook. La respuesta es procesada y almacenada en una variable para su uso posterior en el flujo del bot.

#### Detalles del Bloque de Fórmulas

1. **Objetivo del Bloque**:
   - El bloque de fórmulas se utiliza para acceder a la respuesta específica de OpenAI y almacenarla para futuras interacciones dentro del chatbot.

2. **Configuración de la Fórmula**:
   - **Output Variable**: `@chat_response`
     - Esta variable almacenará el texto de la respuesta de OpenAI, facilitando su uso en el flujo de chat para proporcionar una interacción fluida con el usuario.
   - **Fórmula Usada**:
     ```plaintext
     GetValue(@json_chat_response, "choices", 0, "message", "content")
     ```
     - Esta fórmula descompone la estructura JSON de la respuesta obtenida de OpenAI para extraer el contenido textual de la respuesta. Navega a través del JSON para localizar la parte exacta donde OpenAI devuelve el texto generado en respuesta a la entrada del usuario.


#### Uso del Bloque de Fórmulas

- Es vital asegurarse de que la variable `@json_chat_response` contenga la respuesta JSON completa de OpenAI antes de aplicar esta fórmula. Cualquier error en la extracción de datos puede llevar a respuestas incorrectas o a fallos en el flujo del chatbot.

#### Más Información
- Para más detalles sobre cómo utilizar y configurar fórmulas en Landbot, puedes consultar la [documentación oficial de Landbot sobre fórmulas](https://help.landbot.io/article/zh7wqs85mb-formulas-array-new).

![Fórmula OpenAI 1](https://i.imgur.com/qGJABfc.png)
![Fórmula OpenAI 2](https://i.imgur.com/QORDEdt.png)

### - Configuración del Bloque "Trigger Automation" para Conectar con Make

#### Descripción General
Este bloque se utiliza para conectar Landbot con Make, donde se gestionará la transformación de texto a voz y el almacenamiento del audio resultante en una URL accesible. Este paso es esencial debido a que los servicios de TTS (Text-to-Speech) como los de OpenAI devuelven directamente el archivo de audio, mientras que Landbot requiere una URL para poder reproducir estos audios dentro de la plataforma.

#### Configuración del Bloque

1. **Pegar la URL del Webhook de Make**:
   - **URL del Webhook**: La URL se proporciona por Make y es el punto de entrada para enviar datos desde Landbot a Make.
   - Ejemplo de URL: `https://hook.eu2.make.com/tt8cqimduii9p7q76kxbyg`
   - Esta URL es específica para tu flujo en Make y se generará cuando configures el trigger de webhook en Make.

2. **Establecer Datos a Enviar (Variables)**:
   - **Variable**: `@chat_response`
   - Este paso permite enviar la respuesta del chat, que se obtiene del flujo de diálogo en Landbot, a Make. Aquí se utilizará para procesar o almacenar según se necesite en los pasos siguientes del flujo automatizado.

#### Importancia de la Conexión con Make

- **Procesamiento de TTS**: Make manejará el proceso de Text-to-Speech utilizando OpenAI para convertir el texto de `@chat_response` en audio.
- **Almacenamiento en Google Cloud Storage**: Posteriormente, Make almacenará el audio en Google Cloud Storage, generando una URL pública que será devuelta a Landbot para su uso en el bot, como la reproducción de audio para el usuario.

#### Próximos Pasos

- **Configuración en Make**: En los siguientes pasos, aprenderemos a configurar Make para:
  - Generar audio utilizando TTS de OpenAI.
  - Almacenar este audio en Google Cloud Storage.
  - Generar una URL pública del archivo de audio para su uso en Landbot.

#### Más Información

- Este proceso es crucial para la funcionalidad del bot, ya que permite una integración fluida y funcional de capacidades de audio avanzadas dentro de la plataforma de Landbot. Para más detalles sobre cómo configurar webhooks y manejar datos en Make, visita la [documentación oficial de Make](https://www.make.com/en/help).

![Captura de pantalla 2024-05-15 135518](https://i.imgur.com/kKzC0lq.png)

## Configuración de Webhook en Make

### - Creación del Escenario
Antes de poder configurar cualquier integración o proceso automatizado, necesitas crear un nuevo escenario en Make. Esto proporciona un espacio de trabajo donde puedes añadir y configurar bloques para realizar diversas tareas automáticamente, basándote en los datos recibidos o eventos disparados.

### - Configuración del Escenario en Make para Integración con OpenAI y Google Cloud Storage

#### Configuración del Webhook Customizado

1. **Webhook de Entrada - Custom Webhook**:
   - **Objetivo**: Este webhook actúa como el punto de entrada del escenario. Recibe datos externos que disparan los procesos automatizados configurados en Make.
   - **Configuración**: Debes configurar este webhook para que escuche las solicitudes entrantes desde otras aplicaciones, en este caso desde Landbot.

![Custom Webhook Make](https://i.imgur.com/cLRuhIF.png) 

2. **Propiedades del Webhook**:
   - **URL del Webhook**: La URL proporcionada por Make es donde Landbot enviará los datos. Copia esta URL y úsala en la configuración del trigger en Landbot para conectar ambos sistemas.
   - **Número Máximo de Resultados**: Configura el máximo número de iteraciones que el webhook procesará por ciclo. Esto ayuda a gestionar el rendimiento y evitar sobrecargas en el servidor.

   ```plaintext
   URL de ejemplo: https://hook.eu2.make.com/tt8cqimduii9p76kxbyg36phvqasu
    
   ```
### - Configuración de OpenAI para la Generación de Audio en Make

#### Módulo de TTS (Text-to-Speech) de OpenAI

1. **Conexión**:
   - Asegúrate de tener una conexión establecida con OpenAI. Si aún no has configurado una, necesitarás crearla utilizando tu API key proporcionada por OpenAI.

2. **Configuración del Módulo**:
   - **Input**:
     - `@chat_response`: Esta es la variable que contiene el texto recibido de Landbot que quieres convertir a audio.
   - **Modelo**:
     - Selecciona `tts-1` para utilizar el modelo de texto a voz de OpenAI.
   - **Voz**:
     - Elige `Echo` u otra voz disponible según tus preferencias y necesidades. OpenAI ofrece varias opciones de voz que puedes explorar en su [guía de opciones de voz](https://openai.com/api/tts-voices).

3. **Configuración de Salida**:
   - **Output Filename**:
     - Especifica el nombre del archivo de salida donde se guardará el audio generado. Por ejemplo, `openai-tts-output`. No es necesario añadir la extensión del archivo, ya que OpenAI manejará esto automáticamente basándose en el formato seleccionado, en este caso aac.

![tts Make](https://i.imgur.com/I9qEvHk.png)  

#### Uso del Audio Generado

El audio generado por este módulo será utilizado en los pasos siguientes del flujo de trabajo en Make, donde puede ser almacenado en Google Cloud Storage para su uso posterior, como la reproducción en Landbot o el envío a otros sistemas.

#### Documentación y Soporte

Para más detalles sobre la configuración y opciones disponibles en el módulo de TTS de OpenAI, puedes consultar la [documentación oficial de OpenAI](https://openai.com/api/).

### - Configuración de Google Cloud Storage en Make

#### Establecimiento de Conexión

Para configurar Google Cloud Storage en Make, primero debes establecer una conexión con tu cuenta de Google Cloud. Sigue los pasos detallados en la [documentación de ayuda de Make para Google Cloud Storage](https://www.make.com/en/help/app/google-cloud-storage) para configurar esta conexión.

#### Creación de Proyecto y Bucket en Google Cloud

Antes de configurar la conexión, asegúrate de tener un proyecto y un bucket configurados en Google Cloud. Usa los siguientes comandos en la CLI de Google Cloud para crearlos:


1.`gcloud projects create [PROJECT_ID] --name="[PROJECT_NAME]"`

2.`gcloud storage buckets create gs://[BUCKET_NAME] --project=[PROJECT_ID] --location=US`


#### Configuración del Bloque de Carga en Make

Una vez establecida la conexión y configurado el bucket, procede a configurar el bloque en Make para cargar los archivos de audio:

1. **Conexión**: Asegúrate de seleccionar o agregar la conexión configurada previamente para Google Cloud Storage.
2. **Project ID**: Introduce el ID de tu proyecto de Google Cloud.
3. **Bucket**: Especifica el nombre de tu bucket donde se almacenará el audio.
4. **File**: Aquí se define el nombre del archivo. Por ejemplo, `openai-tts-output`. Make manejará automáticamente la extensión basada en el tipo de contenido especificado.
5. **Content Type**: Establece como `audio/aac`, que es un formato de audio común para los archivos de WhatsApp.
6. **Upload Type**: Selecciona `media`, lo que indica que el contenido subido es un archivo multimedia.

![Google Cloud Storage Make](https://i.imgur.com/H7wcDNs.png)  

#### Beneficios del Almacenamiento en Google Cloud Storage

Al utilizar Google Cloud Storage y configurar el nombre de archivo de manera constante (`openai-tts-output`), cada ejecución del flujo sobrescribe el archivo anterior. Esto garantiza que el enlace al archivo de audio siempre apunte al archivo más reciente generado por el flujo, facilitando la gestión y el acceso continuo sin necesidad de actualizar o gestionar múltiples archivos de audio.

#### Comandos Adicionales

Para obtener más información sobre cómo manejar proyectos y buckets en Google Cloud, puedes consultar la [documentación oficial de Google Cloud SDK](https://cloud.google.com/sdk/gcloud).

### - Configuración del Webhook de Retraso en Landbot

Este bloque de Webhook se coloca después del trigger y está configurado para introducir un retraso de 20 segundos en el flujo del bot. El retraso es crucial cuando las respuestas generadas tardan en actualizarse; esto asegura que el bot maneje la respuesta más reciente en lugar de una anterior.

#### URL y Método
- **Método**: GET
- **URL**: `https://httpstat.us/201?sleep=20000`
  - Esta URL utiliza un servicio externo para simular un retraso mediante la consulta `sleep=20000`, que representa un retraso de 20 segundos.

#### Guardar Respuestas Como Variables
- Aunque este paso es opcional y no afecta la funcionalidad del retraso, guardar la respuesta del webhook en una variable (`@res`) asegura que el webhook se ejecutó correctamente. Sin guardar la respuesta en una variable, no habría confirmación dentro del flujo de que el retraso fue efectivo.

![WebHook de Retraso 1](https://i.imgur.com/jdbX2GY.png)
![Webhook de Retraso 2](https://i.imgur.com/VegkpDr.png)


Este bloque en el flujo de Landbot ayuda a controlar el timing de las operaciones, especialmente útil en situaciones donde las respuestas del sistema deben estar completamente actualizadas antes de continuar con la siguiente acción.

### - Bloque de Envío de Mensaje en Landbot

Para enviar el audio final al usuario, utilizamos un bloque "Send a Message" en Landbot. Este bloque se configura para enviar un mensaje de tipo audio, que se enlaza directamente a la URL pública del archivo de audio alojado en Google Cloud Storage. Esto asegura que el usuario reciba siempre el audio más reciente generado por el flujo.

#### Configuración del Bloque:

1. Selecciona la opción "From URL" en el bloque de "Select media".
2. Ingresa la URL pública del audio. Esta URL se encuentra en los detalles del objeto en Google Cloud Storage, bajo el campo "URL pública".

#### Configuración de Permisos en Google Cloud:

Para garantizar que el archivo de audio sea accesible públicamente, debes configurar los permisos del bucket en Google Cloud. Utiliza el siguiente comando para establecer el acceso público al bucket:

`gsutil iam ch allUsers:objectViewer gs://[BUCKET_NAME]`


![Detalles del objeto Google Cloud](https://i.imgur.com/n7xGyD6.png)
![Send audio](https://i.imgur.com/w6FVRn0.png)

### - Finalizando la Conversación en Landbot

Para gestionar el final de la conversación de manera funcional, puedes configurar Landbot para preguntar si el usuario necesita más ayuda. Esto se realiza mediante un bloque de **Reply Buttons** que facilita respuestas rápidas:

#### Configuración de los Botones de Respuesta
1. **Pregunta**: "¿Te puedo ayudar con algo más?" — Esto permite al usuario expresar si necesita más ayuda.
2. **Botones**:
   - **Sí**: Si el usuario necesita más ayuda, el flujo puede redirigirse al inicio del bot para continuar con la asistencia.
   - **No**: Si el usuario selecciona esta opción, se muestra un mensaje de despedida y se cierra la conversación.

![Repply Buttons](https://i.imgur.com/fTuAMOY.png)

#### Flujo de cierre 
- Si el usuario responde "Sí", el bot redirecciona al inicio para comenzar de nuevo.
- Si el usuario responde "No", se muestra un mensaje final, como "Ha sido un placer ayudarte", y se cierra el chat. Esto se gestiona con un bloque **Close Chat** para asegurar que la sesión se termina correctamente.

![Flujo de Cierre Landbot](https://i.imgur.com/x5lCP59.png)

Incorporar este método asegura que el usuario tenga una experiencia de cierre clara y que todas sus necesidades sean atendidas antes de terminar la interacción con el bot.

### Flujo de Landbot

![Flujo Landbot](https://i.imgur.com/hKRVjFz.png)

### Flujo de Make

![Flujo Make](https://i.imgur.com/D89zgPb.png)

### Demostración del Funcionamiento del Bot

Para obtener una mejor comprensión de cómo funciona el bot y verlo en acción, puedes acceder al siguiente enlace, donde se proporciona un vídeo demostrativo:

[Ver Vídeo Demostrativo](https://drive.google.com/file/d/1GS71yjGYFp55ml62_sdWVGSLQ30UpHhX "Demostración del Bot")































