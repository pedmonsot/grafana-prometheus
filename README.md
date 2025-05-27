# RA 5.3

## Ejercicio 3.1.1 - Monitorización con Prometheus y Grafana

En esta práctica hemos montado un stack completo de monitorización utilizando **Prometheus**, **Grafana**, **cAdvisor** y **Node Exporter**. El objetivo ha sido aprender a visualizar métricas en tiempo real del sistema y de los contenedores usando herramientas modernas y eficientes.

Primero, nos situamos en el directorio `prometheus_grafana_example` y lanzamos todos los servicios con `docker-compose up -d`. Esto nos desplegó los contenedores definidos en el archivo `docker-compose.yml`, entre ellos `traefik`, `nodeapp`, `prometheus`, `grafana` y `cadvisor`. Se construyeron las imágenes necesarias, especialmente la de `nodeapp`, y se descargaron las imágenes oficiales de los servicios restantes.  

![2025-05-26_23-19](https://github.com/user-attachments/assets/eb097ee7-d74e-4de9-91a5-688c034ff0ec)

Una vez todo estuvo arriba, accedimos a `localhost:8080` y comprobamos que **cAdvisor** estaba funcionando correctamente. Esta herramienta nos da una visión general del uso de CPU, memoria y almacenamiento de los contenedores activos. Nos apareció la interfaz con los diferentes *subcontainers*, las gráficas de uso y el estado de cada uno.  

![2025-05-26_23-21](https://github.com/user-attachments/assets/64bc7b1a-61c5-4dd6-b417-7ec0775f14e4)

En un primer intento, **Grafana** no mostraba datos. Aunque la interfaz estaba accesible, los paneles aparecían con “N/A” o “No data”. Investigando más a fondo, descubrimos que **Prometheus** estaba teniendo problemas para interpretar el archivo `prometheus.yml`. El error estaba relacionado con una versión no válida de la API del `alertmanager`.  

![2025-05-26_23-29_1](https://github.com/user-attachments/assets/820ffdd3-8448-4a57-b876-63ae4377aae2)

Tras revisar los logs de Prometheus, nos dimos cuenta de que el problema era que usábamos `api_version: v1`, cuando la versión esperada era `v2`. Hicimos el cambio en el archivo de configuración y relanzamos los contenedores. Esta vez, todo se levantó correctamente y Prometheus ya podía leer las métricas.

![2025-05-27_11-51](https://github.com/user-attachments/assets/22d5b6a5-bd91-4e34-9998-761aa2315c04)

![2025-05-27_11-54](https://github.com/user-attachments/assets/892bdd64-8970-46b3-b45a-40802b6197d3)

Finalmente, conseguimos que **Prometheus** reconociera correctamente todos los *endpoints* de monitorización, y **Grafana** empezó a mostrar todos los gráficos con las métricas del sistema y los contenedores en tiempo real.

![2025-05-27_11-59](https://github.com/user-attachments/assets/a9134f52-7be5-458e-8071-65e7ff00492d)

![2025-05-27_17-59](https://github.com/user-attachments/assets/a498c4c0-6110-4895-b052-9de95feba689)

---

## Ejercicio 3.1.2 - Stack de monitorización remota con Prometheus, Grafana y Node Exporter

En esta segunda parte de la práctica configuramos una monitorización remota en la que un cliente **Ubuntu 24.10** se encarga de consultar las métricas del servidor a través de `prometheus/node_exporter`, y luego mostrarlas en **Grafana** de forma gráfica y detallada.

Al principio nos encontramos con un error al intentar scrapear las métricas de **Node Exporter** desde **Prometheus**. El *endpoint* estaba mal configurado, apuntando a `localhost:9100`, lo cual no tenía sentido porque **Node Exporter** estaba corriendo en una máquina remota. Esto hacía que Prometheus no pudiera recoger los datos y marcara el servicio como "DOWN".  

![2025-05-27_19-19](https://github.com/user-attachments/assets/775a6814-5f7a-4ca2-bf89-768c716d9704)


Una vez que corregimos la URL y pusimos la IP correcta del servidor `http://192.168.1.175:9100`, Prometheus ya fue capaz de comunicarse con Node Exporter, y cambió el estado a "UP". Ahora sí estaba recogiendo las métricas correctamente.  

![2025-05-27_19-20](https://github.com/user-attachments/assets/ca8f5ecc-9dbc-4044-a370-f1cdc4ab77ef)


También verificamos manualmente que **Node Exporter** estaba funcionando accediendo directamente al puerto `9100` desde el navegador. Vimos la interfaz típica con los enlaces a `/metrics` y perfiles de CPU y memoria, lo cual confirmaba que estaba sirviendo datos.  

![2025-05-27_19-49](https://github.com/user-attachments/assets/c3a0d114-ef30-4c76-b8c1-5fb304ed678b)


Finalmente, abrimos **Grafana** y configuramos correctamente la fuente de datos de Prometheus. Mostrándose los gráficos en tiempo real. Se podía ver claramente el uso de **CPU**, **RAM**, **disco**, **tráfico de red** y otros recursos del servidor.  

![2025-05-27_19-58](https://github.com/user-attachments/assets/0b129686-4531-4752-a1a9-35e411c643b7)


---

## Conclusiones

En el **Ejercicio 3.1.1**, montamos un entorno local para visualizar métricas tanto del sistema como de contenedores Docker. Usamos `docker-compose` para desplegar todos los servicios y nos enfrentamos a errores comunes, como versiones incorrectas en la configuración de Prometheus, lo cual nos dio una valiosa experiencia en resolución de incidencias reales.

Por otro lado, en el **Ejercicio 3.1.2**, simulamos un escenario más cercano a producción, donde un cliente remoto monitoriza un servidor a través de Prometheus y Grafana. Esto nos permitió trabajar con recolección remota de métricas, comprobar la exposición de endpoints y configurar adecuadamente la IP del servidor para que los datos pudieran ser consumidos por el cliente.

Las herramientas utilizadas fueron:

- **Prometheus**: para la recolección y almacenamiento de métricas.
- **Grafana**: para la visualización gráfica y creación de dashboards.
- **Node Exporter**: para exponer métricas del sistema operativo.
- **cAdvisor**: para monitorizar métricas de contenedores Docker.
- **Docker Compose**: para orquestar y levantar los servicios de forma sencilla.
