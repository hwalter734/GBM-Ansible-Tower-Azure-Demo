# Ansible Tower Azure Demo
# Contenido
El propósito del repositorio es poder demostrar la capacidad de integrar aplicaciones web utilizando la plataforma de Ansible Tower y Microsoft Azure. En este ejemplo, se estará creando un grupo de máquinas virtuales escalables. 
# Requisitos
- **Ansible**: Debe tener instalado Ansible en su nodo de control. Independientemente del sistema operativo que utilice, haga uso del manejador de paquetes predeterminado de su máquina (apt|yum|etc). Más información sobre la instalación y requerimientos de Ansible haga click [aquí](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).
- **Ansible Tower**: Debe tener instalado Ansible Tower en su nodo de control. Más información sobre la instalación de Ansible Tower haga click [aquí](https://docs.ansible.com/ansible-tower/2.2.2/html/quickinstall/index.html).
- **Licencia Ansible Tower**: Debe de adquirir una membresía de Ansible Tower para poder utilizar la plataforma, consulte la siguiente [página](https://www.redhat.com/en/technologies/management/ansible/try-it) o contacte a su supervisor para adquirir una membresía.
- **Cuenta Microsoft Azure**: Puede obtener una prueba de 1 mes y $200 de crédito al crear una cuenta nueva en Microsoft Azure. De esta manera puede realizar la demo de este repositorio. Puede ingresar al siguiente [link](https://azure.microsoft.com/en-us/free/) para crear su cuenta.
## Instalación de paquetes IMPORTANTE
No utilicé otro método de internet ya que puede ocasionar conflicto de dependencias en su máquina. Tampoco debe instalar los módulos de Azure por medio de Ansible Galaxy ya que este también le ocasionará problemas. Utilice el siguiente comando para instalar las librerías y módulos necesarios para poder utilizar Ansible con Azure.
`pip install 'ansible[azure]'`
## Autenticación cuenta Microsoft Azure
Para poder autenticar su usuario de Microsoft Azure con Ansible, utilice el siguiente comando `az login`. Dependiendo si lo está haciendo en su máquina local o una remota, el método de log in puede ser interactivo por medio de una página web o bien le proporcione un código que debe ingresar en el link que le aparezca en pantalla.
## Ambiente Ansible Tower
Ansible Tower necesita de sus credenciales para poder ejecutar los playbooks. En la parte de *Credentials*, debe crear una nueva credencial de tipo *Microsoft Azure Resource Manager*. Debe ingresar su username como también contraseña. Para obtener su *Subscription ID* y *Tenant ID* puede correr el siguiente comando en la terminal `az account list`. El ID es su *Subscription ID* y su Tenant ID corresponde con *Tenant ID*. Los demás campos los puede dejar vacíos.
![azureinfocredentials](screenshots/azurecredentials.png?raw=true)

**NOTA IMPORTANTE** Si en la ejecución de los playbooks le sale un error de autenticación, su cuenta puede tener problemas de autenticación con Azure. Como alternativa, puede utilizar Azure Active Directory para crear un nuevo usuario, proporcionarle todos los permisos a sus recursos y utilizar esta cuenta con Ansible Tower. Más información [aquí](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/add-users-azure-active-directory).  

Opcionalmente, si desea utilizar un webhook con la plataforma de Ansible Tower debe asegurarse que su servidor cuente con un certificado SSL válido. De no contar con un certificado, deberá modificar su archivo de configuración nginx. Este se encuentra en el siguiente path: **/etc/nginx/nginx.conf**. Previo a editar el archivo se recomienda hacer una copia en el caso la modificación resulte en errores. Las flechas rojas demuestra como se encuentra el archivo original y las flechas verdes como usted debe modificarlo. Para la dirección IP, no olvide incluir **:80** al final de su dirección. El cuadro amarillo resalta los bloques que debe de eliminar por completo del archivo **nginx.conf**.

![archivonginx](screenshots/archivonginx.png?raw=true)

## Archivo de variables
Recomendamos el uso de **Ansible-Vault** para poder guardar sus credenciales e información de su cuenta de Azure de manera segura. Tome en cuenta que la contraseña que utilice en sus máquinas virtuales (la variable **vmss_password**) tiene que tener un largo de por lo menos 7 caracteres, tener una letra mayúscula y minúscula, tener un número y por último un caracter especial. De lo contrario, al momento de tirar el playbook de configuración de las máquinas virtuales escalables le tirará error en el uso de la contraseña.

### Creación usuario para webhook
Por temas de seguridad, se recomienda crear un usuario que solamente tenga acceso a la ejecución del playbook que se activa por medio del webhook. De esta manera, si alguien logra obtener las credenciales solamente tendrá acceso a la ejecución de un playbook. 
![ansibleuser](screenshots/userpermissions.png?raw=true) 

### Webhook
El webhook tiene el siguiente formato: http://usuario:contraseña@direccionip/api/v2/job_templates/20/launch/. El número después de *job_templates* es depende de su plantilla de job. Puede encontrar el número que corresponde al template entrando a su job template y habilitando temporalmente la opción de **callback** para obtener el número que corresponde.
![jobtemplatenumber](screenshots/jobtemplatenumber.png?raw=true) 


## Ambiente Azure
Normalmente, Azure Active Directory debe estar conectado con todos los recursos que tenga disponible en su cuenta. Si en la ejecución de playbooks tuviera un error relacionado con Azure Active Directory, asegúrese que su cuenta y sus recursos estén linked con Azure Active Directory. Para más información puede ingresar en este [link](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory).

### Setup de alerta
En el portal de Azure, ingrese o busque la aplicación de alertas como se muestra en la imagen.
![alerticon](screenshots/alerticon.png?raw=true)

Deberá crear una acción de alerta parecida a la siguiente. Tome nota que al meter la dirección de su job template, debe indicar que **NO utilice el common alert schema**. De igual manera asegúrese que esta acción esté conectada con su grupo de recursos donde se encuentra sus máquinas virtuales escalables. A continuación, debe crear un alert rule.
![alerticon2](screenshots/alerticon2.png?raw=true)

En esta sección usted determina la condición en la cual se dispara la alerta a su webhook.
![alerticon3](screenshots/alerticon3.png?raw=true)

En nuestra demo establecemos que media vez haya uso de 50% o más del CPU promedio de las máquinas virtuales por más de 5 minutos se notifica al webhook de Ansible Tower. En la sección de Action agrega el grupo de acciones que creo anteriormente.
![alerticon4](screenshots/alerticon4.png?raw=true)

# Recomendaciones Finales
Ansible puede tener problemas de versionamiento con los módulos de Azure dependiendo de su versión de python. En el archivo de **inventory**, puede eliminar la variable de ansible python interpreter o reemplazarla con su interpretador de python preferido. De igual manera puede establecer esta variable en la sección de inventarios y hosts en Ansible Tower.
## Aplicación de Prueba 
La aplicación que se utiliza en los archivos es una aplicación Java. Se empaqueta con Maven y finalmente se ejecuta con el siguiente comando: `java -jar`. Si usted desea utilizar otra aplicación, puede modificar los archivos para que se ejecute correctamente.
