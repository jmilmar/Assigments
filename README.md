---
# **Apartado Teorico: Preguntas de Opción Multiple**

// Las opciones correctas estarán marcadas con el simbolo más (+).

1. **¿Qué es DevOps?**
   -
  - a) Una herramienta de automatización de tareas
  - **b) Una cultura y práctica que enfatiza la colaboración entre desarrollo y operaciones.** (+)
  - c) Una plataforma de orquestación de contenedores.

2. **¿Cuál es el propósito principal de Ansible en el contexto de DevOps?**
   -
  - a) Gestionar contenedores Docker.
  - **b) Automatizar la configuración y el despliegue de sistemas y aplicaciones.** (+)
  - c) Administrar bases de datos.

4. **¿Cuál es el propósito de Elasticsearch en una infraestructura DevOps?**
   -
  - a) Una herramienta de seguimiento de problemas y errores.
  - **b) Un motor de búsqueda y análisis de datos en tiempo real.** (+)
  - c) Un servidor de aplicaciones.

6. **¿Por qué es importante utilizar Ansible para instalar Elasticsearch en lugar de configurarlo manualmente en cada nodo?**
   -
  - **a) Acelera el proceso de instalación y configuración.** (+)
  - **b) Garantiza que todos los nodos tengan configuraciones idénticas.** (+)
  - c) Facilita la depuración de errores.


---

# **Apartado Practico: Documentación**

### **Requisitos Previos:**

1. **Sistema de Control de Versiones:**
   Asegúrate de tener Ansible instalado en tu máquina local. Puedes instalar Ansible siguiendo la [documentación oficial de Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

2. **Inventario de Ansible:**
   Crea un archivo de inventario de Ansible que incluya los nodos que deseas gestionar. Por ejemplo, puedes tener el archivo `/etc/ansible/hosts` con los siguientes contenidos:

   ```ini
   [NODOS]
   host1 ansible_host=192.168.1.171 ansible_user=jmilmar1
   host2 ansible_host=192.168.1.172 ansible_user=jmilmar2
   ```

3. **Acceso SSH sin Contraseña:**
   Asegúrate de tener configurado el acceso SSH sin contraseña desde la máquina donde ejecutarás Ansible a los nodos `host1` y `host2`. Esto es fundamental para que Ansible pueda conectarse y ejecutar comandos en los nodos de forma segura.
  ---
### **Paso 1: Instalación de Elasticsearch**

Crea un archivo llamado `instalar-elasticsearch.yml` con el siguiente contenido:

```yaml
- name: Instalar Elasticsearch en host1 y host2
  hosts: NODOS
  become: true

  tasks:
	- name: Actualizar caché del sistema y paquetes
  	apt:
    	update_cache: yes
  	when: ansible_os_family == "Debian"

	- name: Instalar Java
  	apt:
    	name: default-jdk
    	state: present

	- name: Descargar Elasticsearch deb
  	get_url:
    	url: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.0-amd64.deb
    	dest: /tmp/elasticsearch.deb

	- name: Instalar Elasticsearch
  	apt:
    	deb: /tmp/elasticsearch.deb
  	notify: Reiniciar Elasticsearch

  handlers:
	- name: Reiniciar Elasticsearch
  	service:
    	name: elasticsearch
    	state: restarted
```
  ---
### **Paso 2: Configuración de Elasticsearch**

Crea un archivo llamado `configurar-elasticsearch.yml` con el siguiente contenido:

```yaml
- name: Configurar Elasticsearch en nodos gestionados
  hosts: NODOS
  become: true

  tasks:
	- name: Editar el archivo de configuración Elasticsearch.yml
  	blockinfile:
    	path: "/etc/elasticsearch/elasticsearch.yml"
    	block: |
      	network.host: 0.0.0.0
      	http.port: 9200
      	discovery.seed_hosts: ["192.168.1.171", "192.168.1.172"]
  	when: inventory_hostname in groups['NODOS']

	- name: Reiniciar Elasticsearch
  	service:
    	name: elasticsearch
    	state: restarted
  	when: inventory_hostname in groups['NODOS']
```
  ---
### **Paso 3: Inicio de Elasticsearch**

Crea un archivo llamado `iniciar-elasticsearch.yml` con el siguiente contenido:

```yaml
- name: Iniciar Elasticsearch en host1 y host2
  hosts: NODOS
  become: true

  tasks:
	- name: Iniciar el servicio Elasticsearch
  	systemd:
    	name: elasticsearch
    	enabled: yes
    	state: started
  	when: inventory_hostname in groups['NODOS']
```
  ---
### **Paso 4: Ejecución de los Playbooks**

Para ejecutar los playbooks, utiliza el comando `ansible-playbook` desde la terminal, asegurándote de especificar el archivo de inventario correspondiente:

```bash
ansible-playbook instalar-elasticsearch.yml
ansible-playbook configurar-elasticsearch.yml
ansible-playbook iniciar-elasticsearch.yml
```

Estos comandos ejecutarán los playbooks en los nodos especificados en el archivo de inventario (`/etc/ansible/hosts`).
