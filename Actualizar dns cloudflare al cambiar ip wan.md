# Script para actualizar la IP de tu dominio en Cloudflare

Por [nosololinux ](https://nosololinux.es/author/admin/ "Ver todas las entradas de nosololinux")/ mayo 20, 2024

En  el mundo dinámico de las redes, mantener la información de DNS  actualizada es crucial para garantizar que tu sitio web esté siempre  accesible. Uno de los retos más comunes para quienes no tienen una IP  estática es la actualización constante de la dirección IP asociada a su  dominio. Afortunadamente, Cloudflare ofrece una API robusta que permite  automatizar esta tarea, asegurando que tu dominio apunte siempre a la IP  correcta.

En esta entrada del blog, te guiaremos paso a paso para  crear un script en Bash que actualice automáticamente la IP de un  dominio en Cloudflare. Este script es especialmente útil si utilizas  servicios de Internet con direcciones IP dinámicas, como las que ofrecen  muchos ISP. Al final de esta guía, tendrás una solución automatizada  que se puede ejecutar en tu servidor o dispositivo local, eliminando la  necesidad de actualizar manualmente tu IP en Cloudflare.

### ¿Qué aprenderás en este artículo?

- Cómo interactuar con la API de Cloudflare para obtener información de tus registros DNS.
- Cómo actualizar el registro DNS de tu dominio con la nueva dirección IP utilizando un script en Bash.
- Prácticas recomendadas para asegurar y ejecutar tu script de manera eficiente.

### ¿Por qué es importante mantener tu DNS actualizado?

Imagina  que diriges un pequeño negocio en línea desde casa. Si tu dirección IP  cambia y no actualizas tu DNS, tus clientes no podrán acceder a tu sitio  web, lo que puede resultar en una pérdida de ventas y credibilidad.  Automatizar la actualización de la IP en Cloudflare garantiza que tu  sitio web esté siempre accesible, sin importar cuántas veces cambie tu  dirección IP.

### Requisitos previos

Antes de comenzar, asegúrate de tener:

1. Una cuenta activa en Cloudflare.
2. Acceso al panel de control de Cloudflare para obtener tu clave de API y el ID de zona.
3. Un entorno de terminal en tu sistema operativo, con `curl` instalado para realizar solicitudes HTTP.

### Paso a Paso

¡Comencemos con la configuración del entorno y las credenciales necesarias para interactuar con la API de Cloudflare!

Necesitaremos crear un API Token en el panel de Cloudflare para obtener la API_KEY y así poder especificarla en el script:

![](https://nosololinux.es/wp-content/uploads/2024/05/api-tokens.png)

Deberemos anotar también la Zone ID del dominio que queramos actualizar la IP:

![](https://nosololinux.es/wp-content/uploads/2024/05/API-Zone.png)

Por  otra parte, necesitaremos obtener el RECORD ID de tu dominio ejecutando  el siguiente script ya que desde la GUI de Cloudflare no se proporciona  este dato. **La API_KEY será la «Global API Key»**

```
#!/bin/bash

# Configuración de las credenciales y detalles del dominio
EMAIL="tu_correo@ejemplo.com"
API_KEY="tu_api_key"
ZONE_ID="id_de_tu_zona"
DOMAIN_NAME="tudominio.com"

# Función para obtener el ID del registro DNS
get_record_id() {
    response=$(curl -sS -X GET "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records?type=A&name=${DOMAIN_NAME}" \
    -H "X-Auth-Email: ${EMAIL}" \
    -H "X-Auth-Key: ${API_KEY}" \
    -H "Content-Type: application/json")

    echo "Respuesta de la consulta al servidor de Cloudflare: $response"
    record_id=$(echo "$response" | grep -oP '"id":"\K[^"]+')

    if [ -z "$record_id" ]; then
        echo "Error: No se pudo encontrar el ID del registro DNS para ${DOMAIN_NAME}."
        exit 1
    else
        echo "El ID del registro DNS para ${DOMAIN_NAME} es: $record_id"
    fi
}

# Obtener el ID del registro DNS
get_record_id
```

Una vez tengamos el RECORD ID y el resto de  parámetros necesarios como el API_KEY, ZONE_ID, DOMAIN… configuraremos  el siguiente script que será el encargado de actualizar la IP de nuestro  dominio cuando haya cambiado.

En este script **la API_KEY será el token que solamente Cloudflare nos permite visualizar una única vez cuando lo creamos**:

```
#!/bin/bash

# Configuración
API_KEY="TU-API-KEY"
ZONE_ID="TU-ZONA-ID"
RECORD_ID="TU-RECORD-ID-DE-TU-DOMINIO"  # Aquí debes colocar el record_id
DOMAIN="TU-DOMINIO"
PROXIED="true"  # Cambia a "false" si no deseas habilitar el modo proxy

# Función para obtener la IP pública actual
get_current_ip() {
    curl ifconfig.me
}

# Función para actualizar la entrada DNS en Cloudflare
update_dns() {
    local ip="$1"
    local url="https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}"
    local data='{"type":"A","name":"'"${DOMAIN}"'","content":"'"${ip}"'","proxied":'"${PROXIED}"'}'
    curl -s -X PUT "${url}" \
         -H "Authorization: Bearer ${API_KEY}" \
         -H "Content-Type: application/json" \
         --data "${data}" \
         | jq .
}

# Función principal
main() {
    current_ip=$(get_current_ip)
    # Obtener la última IP registrada en Cloudflare
    last_ip=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
                -H "Authorization: Bearer ${API_KEY}" \
                -H "Content-Type: application/json" \
                | jq -r '.result.content')
    # Actualizar solo si la IP ha cambiado
    if [[ "${current_ip}" != "${last_ip}" ]]; then
        update_dns "${current_ip}"
        echo "IP actualizada en Cloudflare: ${current_ip}"
    else
        echo "La IP no ha cambiado."
    fi
}

# Ejecutar la función principal
main
```

Le damos permisos de ejecución al script:

```
chmod +x nombre-del-script.sh
```

Y ya estaría todo. si queréis podéis [crear una tarea en crontab para que se ejecute periódicamente](https://nosololinux.es/como-configurar-y-gestionar-tareas-cron-en-linux-para-automatizar-tareas-repetitivas/) y no tengáis que ejecutar el script manualmente cuando haya cambiado la IP.