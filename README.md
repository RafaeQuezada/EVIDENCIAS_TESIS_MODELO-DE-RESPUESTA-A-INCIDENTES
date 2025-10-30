# Respaldo Técnico – Entorno de Seguridad del Modelo Integral de Respuesta a Incidentes (MIRI)

Este repositorio contiene el respaldo técnico del entorno de laboratorio desarrollado para la tesis titulada **“Modelo Integral de Respuesta a Incidentes de Seguridad (MIRI)”**, aplicada al call center de la Autoridad de Tránsito Municipal (ATM) de Machala.  
El entorno fue implementado en máquinas virtuales configuradas bajo Oracle VirtualBox, utilizando exclusivamente software libre y herramientas de análisis, monitoreo y respuesta a incidentes.

---

## SECCIÓN 1. Servidor Wazuh (Ubuntu 22.04.5 LTS)

### 1.1 Descripción general

El servidor principal se encuentra configurado sobre una máquina virtual Ubuntu Server 22.04.5 LTS (x64) e integra la plataforma Wazuh en sus tres componentes esenciales: **Manager**, **Indexer** y **Dashboard**.  
Este sistema cumple la función de centro de monitoreo, correlación y visualización de incidentes de seguridad generados por los agentes instalados en equipos cliente.

### 1.2 Especificaciones técnicas

**Nombre de la máquina:** MIRI_Wazuh_Server  
**Tipo de virtualización:** Oracle VirtualBox 7.x  
**Sistema operativo:** Ubuntu Server 22.04.5 LTS  
**Rol:** Servidor SIEM / HIDS centralizado  

### 1.3 Componentes instalados

| Componente | Versión | Función |
|-------------|----------|----------|
| Wazuh Manager | 4.x | Correlación de eventos, ejecución de reglas y generación de alertas. |
| Wazuh Indexer (Elasticsearch) | 4.x | Indexación y búsqueda de eventos. |
| Wazuh Dashboard | 4.x | Interfaz web de visualización y análisis. |
| Filebeat | Integrado | Envío de datos al indexador. |
| Certificados SSL autofirmados | SHA256 | Comunicación cifrada entre servicios. |

### 1.4 Estructura del respaldo

El repositorio incluye la configuración de la máquina virtual y los archivos necesarios para reproducir el entorno Wazuh:

```
/MIRI_Wazuh_Server/
├── MIRI_Wazuh_Server.vbox → Configuración de la máquina virtual
├── README_VM.txt → Resumen de instalación dentro del sistema
├── /config/ → Reglas, logs y archivos de ejemplo
└── README.md → Documento técnico principal
```

**Nota:**  
Por razones de tamaño, el archivo de disco virtual (.vdi) no se incluye en este repositorio.  
El entorno puede reconstruirse importando el archivo `.vbox` y siguiendo las configuraciones detalladas en este documento.

### 1.5 Requisitos mínimos

- VirtualBox 7.0 o superior  
- 4 GB de RAM (recomendado 8 GB)  
- 2 núcleos de CPU  
- 50 GB de espacio libre  
- Red configurada como Adaptador puente o Host-Only

### 1.6 Importación y uso

1. Descargue los archivos de este repositorio y colóquelos en una carpeta local.  
2. En VirtualBox, seleccione **Máquina → Agregar** y elija `MIRI_Wazuh_Server.vbox`.  
3. Inicie la máquina.  
4. Espere entre 1 y 3 minutos hasta que los servicios de Wazuh estén activos.

**Acceso al sistema operativo:**  
Usuario: wazuhadmin  
Contraseña: Miri2025*

**Acceso al Dashboard Web:**

1. Obtener la IP con: `ip a`  
2. En el navegador del equipo anfitrión ingresar:  
   `https://<IP_de_la_VM>:5601`  
   Ejemplo: `https://192.168.56.101:5601`

**Credenciales del Dashboard:**  
Usuario: admin  
Contraseña: VzSiQcXRmi2kyjzcA+mYLEtbGVs=

### 1.7 Archivos de configuración relevantes

| Archivo / Ruta | Descripción |
|-----------------|-------------|
| /var/ossec/etc/ossec.conf | Configuración general del Wazuh Manager. |
| /var/ossec/etc/rules/local_rules.xml | Reglas personalizadas (casos UC-01 a UC-06). |
| /var/ossec/logs/alerts/alerts.json | Registro consolidado de alertas. |
| /etc/filebeat/filebeat.yml | Parámetros de envío de logs al Indexer. |
| /usr/share/wazuh-dashboard/config/opensearch_dashboards.yml | Configuración del Dashboard. |

**Ejemplo de regla personalizada:**

```
<group name="local,axiscloud,">
  <rule id="100020" level="10">
    <if_group>json</if_group>
    <field name="source">axiscloud</field>
    <field name="status">success</field>
    <field name="mfa">disabled</field>
    <description>UC-01: Acceso exitoso a Axis Cloud sin MFA habilitado</description>
    <group>incident,uc01,</group>
  </rule>
</group>
```

### 1.8 Comandos de verificación

**Estado de servicios:**
```
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

**Reinicio manual:**
```
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard
```

**Apagado seguro:**
```
sudo shutdown now
```

### 1.9 Agregar una nueva IP de agente en Wazuh Manager

Para integrar un nuevo agente al servidor Wazuh Manager:

1. Ingrese al servidor Wazuh con privilegios de administrador.  
2. Ejecute el asistente de agentes:
   ```
   sudo /var/ossec/bin/manage_agents
   ```
3. En el menú:
   - Seleccione la opción **A** (Add agent).  
   - Ingrese un nombre para identificarlo (por ejemplo, AgenteWindows01).  
   - Especifique la dirección IP del agente (por ejemplo, 192.168.56.102).  
   - Confirme la operación.  
4. Copie la clave que se genera para el nuevo agente.  
5. En el equipo cliente (agente), ejecute:
   ```
   agent-auth -m <IP_DEL_SERVIDOR> -A <nombre_del_agente> -k <clave_copiada>
   ```
6. Verifique la conexión desde el Dashboard, en el menú **Agents → All Agents**.  
   El agente aparecerá con estado *Active* una vez autenticado correctamente.

---

## SECCIÓN 2. Sistema de gestión de tickets

### 2.1 Descripción general

Como parte del modelo MIRI, se desarrolló una aplicación web local denominada **Sistema de Gestión de Incidentes de Seguridad**, programada en **HTML, JavaScript y Bootstrap 5**, que permite registrar y controlar el ciclo de vida de los incidentes detectados durante las simulaciones.

### 2.2 Objetivo

Proporcionar una interfaz ligera y funcional para documentar cada incidente, asignar responsables y dar seguimiento a su resolución, simulando el flujo institucional de respuesta a incidentes de seguridad.

### 2.3 Características técnicas

- Tipo de aplicación: Local (sin servidor).  
- Persistencia: localStorage del navegador.  
- Componentes:
  - Formulario de creación de tickets.  
  - Tabla dinámica con listado de incidentes.  
  - Opción para actualizar o eliminar registros.  
- Estados disponibles: “Abierto”, “En análisis” y “Resuelto”.

### 2.4 Fragmento representativo del código

Formulario HTML:
```
<form id="ticketForm" class="card p-3 shadow-sm mb-3">
  <div class="row g-3">
    <div class="col-md-4">
      <label for="titulo">Título</label>
      <input type="text" id="titulo" class="form-control" required>
    </div>
    <div class="col-md-8">
      <label for="descripcion">Descripción</label>
      <input type="text" id="descripcion" class="form-control" required>
    </div>
    <div class="col-md-3">
      <label for="severidad">Severidad</label>
      <select id="severidad" class="form-select">
        <option value="Baja">Baja</option>
        <option value="Media">Media</option>
        <option value="Alta">Alta</option>
      </select>
    </div>
  </div>
  <div class="text-end mt-3">
    <button type="submit" class="btn btn-primary">Agregar ticket</button>
  </div>
</form>
```

Lógica JavaScript:
```
ticketForm.addEventListener('submit', e => {
  e.preventDefault();
  const titulo = document.getElementById('titulo').value.trim();
  const descripcion = document.getElementById('descripcion').value.trim();
  const severidad = document.getElementById('severidad').value;
  const nuevoTicket = {
    id: Date.now(),
    titulo,
    descripcion,
    severidad,
    estado: "Abierto"
  };
  const tickets = JSON.parse(localStorage.getItem('tickets')) || [];
  tickets.push(nuevoTicket);
  localStorage.setItem('tickets', JSON.stringify(tickets));
  mostrarTickets();
});
```

### 2.5 Propósito del sistema

El sistema de tickets permite simular el proceso institucional de registro, análisis y cierre de incidentes de seguridad.  
Cada registro representa un evento detectado por Wazuh, permitiendo mantener trazabilidad documental y control del flujo de atención durante la validación del modelo MIRI.

---

## Autores

**Carlos Luis Pulla Freire**  
**Anthony Rafael Quezada Crespo**  
Proyecto de titulación – Ingeniería en Tecnologías de la Información  
Universidad Estatal de Milagro (UNEMI), 2025  
Repositorio: [https://github.com/usuario/MIRI-Wazuh-Server](https://github.com/usuario/MIRI-Wazuh-Server)

---

## Licencia

Este respaldo se publica con fines académicos y de validación técnica.  
Se prohíbe su uso comercial o la reproducción parcial sin autorización de los autores.
