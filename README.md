### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Nicolas Sebastian Achuri Macias y Ricardo Andres Villamizar Mendez

---------------------------------------
### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)




2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

![image](https://github.com/user-attachments/assets/dbe8a08e-8a9b-461f-948a-b9389aaba8ed)


4. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

   ![image](https://github.com/user-attachments/assets/b6f96a7d-5d09-4dde-9921-3c6436979488)

5. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`



6. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

   ![image](https://github.com/user-attachments/assets/4d1145ee-f330-45e2-a5ba-7dd8878017d9)


8. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

![image](https://github.com/user-attachments/assets/a392fc30-5e08-4bdd-aedd-30fac234cfd0)


8. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000
  
![image](https://github.com/user-attachments/assets/4f2c3ab1-b76c-4df8-8ff8-f5e0f64aaa6f)

![image](https://github.com/user-attachments/assets/8238d6b2-c42f-4b89-87a7-8f6700327bfc)

![image](https://github.com/user-attachments/assets/959320e0-c1c4-4afb-891e-59923b7f2949)



10. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![image](https://github.com/user-attachments/assets/4bbfc51e-e855-4aeb-985d-5e031681bfdf)



9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

    ![image](https://github.com/user-attachments/assets/56e9c3bd-b113-4689-bb69-102751bdc41b)


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

![image](https://github.com/user-attachments/assets/71e0b430-c578-46fd-afb7-172e62769365)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

    ![image](https://github.com/user-attachments/assets/ed60accf-134c-48be-aa77-d14d11a0f6f9)

    ![image](https://github.com/user-attachments/assets/e17adb39-4bd0-4778-847f-03474ef89474)


13. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
14. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   Son alrededor de 7 recursos:
   * Una maquina virtual VERTICAL-SCALABILITY
   * Una dirección IP pública VERTICAL-SCALABILITY-ip
   * Un grupo de seguridad VERTICAL-SCALABILITY-ip
   * Una red virtual VERTICAL-SCALABILITY-vnet
   * Una interfaz de red VERTICAL-SCALABILITY571_z1
   * Un disco VERTICAL-SCALABILITY-disk
   * Claves SSH VERTICAL-SCALABILITY_Key
2. ¿Brevemente describa para qué sirve cada recurso?

     **Máquina virtual:** Las Azure Virtual Machines son instancias escalables bajo demanda que actúan como servidores, proporcionando hardware virtual (CPU, memoria, almacenamiento) de forma aislada del sistema subyacente.
     
      **Dirección IP pública:** Facilitan la comunicación entrante de Internet a recursos de Azure y la conexión de estos con servicios públicos y la red.
      
      **Grupo de seguridad:** Permiten configurar reglas de seguridad en la red agrupando máquinas virtuales y sus políticas.
      
      **Red virtual:** Azure Virtual Network crea redes privadas seguras que conectan recursos de Azure entre sí, con Internet y con redes locales, ofreciendo escalabilidad y aislamiento.
      
      **Interfaz de red:** Una NIC permite que las máquinas virtuales de Azure se comuniquen con redes internas, externas y locales.
      
      **Disco:** Azure Managed Disks ofrece almacenamiento de bloques duradero para máquinas virtuales con opciones como Ultra Disk, SSD Premium, SSD estándar y HDD estándar, facturados según tamaño y uso.
   
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

      Al ejecutar npm FibonacciApp.js en SSH, el proceso se asocia al terminal activo y se termina al cerrar la sesión. Para mantenerlo activo, se usa forever para ejecutarlo en segundo plano.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.


   La función presenta un tiempo de ejecución creciente con cada iteración debido a la acumulación de operaciones. Esto hace que el desempeño dependa directamente del número de iteraciones, pudiendo aumentar de forma proporcional o exponencial según la complejidad interna. Optimizar su diseño es esencial para manejar volúmenes altos de iteraciones eficientemente.

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

    ![image](https://github.com/user-attachments/assets/24c2af28-030b-4a1d-97ca-dd8ff9721b68)

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
      ![image](https://github.com/user-attachments/assets/fe669f2e-a454-4fd6-8139-6bab893d4bb2)
      ![image](https://github.com/user-attachments/assets/7055dc81-ce40-4920-bc5c-10946df8f54d)

    * Si hubo fallos documentelos y explique.

      El error "socket hangup" ocurre cuando la conexión entre cliente y servidor se interrumpe inesperadamente, generalmente por cierre abrupto, tiempo de espera agotado, sobrecarga del servidor o problemas de red. Para solucionarlo, ajusta los tiempos de espera, optimiza el servidor, maneja excepciones adecuadamente y verifica la conectividad de la red.
      
      ![image](https://github.com/user-attachments/assets/4123065a-973c-45c1-ba10-e2d5212a48bf)

      
8. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

   
   
   El tamaño B1ls es ideal para tareas ligeras y económicas, como servidores de desarrollo o pruebas básicas, ya que tiene recursos limitados (1 vCPU, 0.5 GiB de memoria) y un costo bajo ($3.80/mes). Por otro lado, el tamaño B2ms ofrece mayor capacidad (2 vCPU, 8 GiB de memoria) a un costo mayor ($58.40/mes), siendo más adecuado para aplicaciones web pequeñas, bases de datos ligeras o entornos que requieren más flexibilidad y rendimiento.
   
9. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
    
   El análisis previo muestra que el escalamiento vertical es una solución efectiva: mientras que el tamaño **B1ls** alcanzaba casi su capacidad máxima bajo múltiples solicitudes, al escalar al tamaño **B2ms**, el uso de CPU se mantuvo por debajo del 50%. Sin embargo, también es crucial optimizar el algoritmo utilizado para mejorar la eficiencia general de la aplicación.
   
10. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
    
    Aumenta el precio
    
12. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

    Si, por que los tiempos se redujeron en gran medida

14. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

No mejoran los tiempos

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

![image](https://github.com/user-attachments/assets/1ce134f8-8cca-4b13-abb2-348909473a2a)


#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

![image](https://github.com/user-attachments/assets/185d2f3d-fa8e-4456-a9ef-88a6322e522b)



5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```


![image](https://github.com/user-attachments/assets/90c39953-23ba-4d5f-8244-7863a6764be9)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```


![image](https://github.com/user-attachments/assets/4462acf4-9fd0-49f4-bbad-48e746697e64)


## **Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

### Tipos de Balanceadores de Carga en Azure

1. **Application Gateway (Nivel de Aplicación)**:
   - Opera en la capa 7 del modelo OSI.
   - Dirige el tráfico según detalles específicos como URL o encabezados HTTP.
   - Ofrece seguridad avanzada como cifrado SSL y protección contra DDoS.

2. **Load Balancer (Nivel de Transporte)**:
   - Funciona en la capa 4 del modelo OSI.
   - Distribuye tráfico según IP, puerto y protocolo.

3. **Gateway VPN**:
   - Establece conexiones seguras entre redes locales y virtuales de Azure mediante VPN.
   - Facilita la integración entre servicios locales y en la nube.

---

### SKU en Azure

Los SKU (Stock Keeping Units) son identificadores únicos que definen las características, capacidades y precios de los recursos en Azure. 

#### Ejemplos de SKU:
- **Máquinas Virtuales**: Núcleos de CPU, RAM, almacenamiento y rendimiento de red.
- **Bases de Datos**: Tamaño, rendimiento y disponibilidad optimizados.
- **Almacenamiento**: Capacidad, rendimiento y durabilidad configurables.
- **Servicios de Red**: Ancho de banda, seguridad y disponibilidad.

---

### Importancia de la IP Pública en Balanceadores de Carga

Para permitir el acceso a recursos de Azure detrás de un balanceador de carga, es imprescindible asignarle una **IP pública**. Esta dirección actúa como punto de entrada para el tráfico externo, garantizando conectividad y acceso a los servicios en la nube.


* ¿Cuál es el propósito del *Backend Pool*?


   El BackEnd Pool distribuye el tráfico entrante hacia recursos como máquinas virtuales o instancias de contenedor en grupos de escalado. Utiliza algoritmos como **Round Robin** o **Hash de IP** para garantizar una carga balanceada y eficiente entre los recursos disponibles.

  
* ¿Cuál es el propósito del *Health Probe*?

  El **Health Probe** monitorea los recursos en el Backend Pool para garantizar que solo los que están en buen estado reciban tráfico. Envía solicitudes periódicas (HTTP o TCP) para verificar su disponibilidad:

- **Recurso saludable**: Responde correctamente y sigue recibiendo tráfico.
- **Recurso no saludable**: No responde o muestra errores, y se excluye del tráfico.

  


* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

   
     La **Load Balancing Rule** dirige el tráfico entrante hacia las instancias de backend de manera eficiente y equitativa. Permite configurar el enrutamiento según criterios como rutas URL o puertos, asegurando una distribución uniforme de solicitudes.

#### Sesiones Persistentes
- **IP Affinity o Cookie Affinity**: Mantienen la continuidad de la sesión entre un cliente y una instancia específica de backend.
- **Impacto**: Aunque mejoran la experiencia del usuario, pueden limitar la escalabilidad, ya que el balanceador debe redirigir todas las solicitudes de un cliente a la misma instancia, lo que puede generar cargas desiguales.

Es importante equilibrar la persistencia de sesiones con la escalabilidad del sistema para evitar cuellos de botella y garantizar un rendimiento óptimo.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
  
1. **Virtual Network (VNet)**:
   - Servicio para crear una red virtual aislada en la nube.
   - Permite definir un rango de direcciones IP, subredes, reglas de seguridad y puertas de enlace para conectividad con Internet o redes locales.

2. **Subnet**:
   - Subdivisión de una VNet para segmentar la red en partes más pequeñas.
   - Cada Subnet tiene su propio rango de direcciones IP dentro del espacio de la VNet y reglas de seguridad específicas.

3. **Address Space**:
   - Rango de direcciones IP privadas asignado a una VNet.
   - Define las direcciones disponibles para la VNet y sus subredes.

4. **Address Range**:
   - Rango de direcciones IP asignado a una Subnet dentro de la VNet.
   - Los recursos de la Subnet reciben direcciones IP de este rango.
  
   
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?


   Una Availability Zone es un grupo de centros de datos interconectados dentro de una misma región, ubicados en sitios físicos independientes y aislados de fallas en otras zonas. Esto garantiza mayor disponibilidad y resiliencia para las aplicaciones alojadas.
   
   Distribuir los recursos de una aplicación en tres zonas de disponibilidad mejora su disponibilidad y rendimiento, ya que permite operar incluso si una o dos zonas fallan. Además, el tráfico se distribuye de manera más eficiente entre las zonas.
   
   La IP zone-redundant es una dirección IP pública asignable a recursos como máquinas virtuales o balanceadores de carga. Está disponible en todas las zonas de una región, asegurando acceso continuo incluso en caso de fallos en una de las zonas.

* ¿Cuál es el propósito del *Network Security Group*?

El propósito del Network Security Group (NSG) en Azure es proporcionar un nivel adicional de seguridad al permitir o denegar el tráfico de red hacia y desde los recursos de Azure, como máquinas virtuales, subredes y redes virtuales. El NSG actúa como un firewall virtual que filtra el tráfico basándose en reglas que especificas.


  

* Presente el Diagrama de Despliegue de la solución.

![image](https://github.com/user-attachments/assets/42728a7f-349b-4ffb-a14f-5436226dd483)



