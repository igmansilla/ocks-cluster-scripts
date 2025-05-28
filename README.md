# Script de Test de Stress Distribuido para Rocks Cluster (Python 2.6.6)

Este archivo Markdown contiene el codigo fuente de un script Python disenado para generar carga de CPU en los nodos esclavos de tu cluster Rocks. Esta version esta adaptada para Python 2.6.6.

**Importante:** Para que este script funcione correctamente, debes haber configurado SSH sin contrasena desde tu nodo maestro de Rocks hacia cada uno de tus nodos esclavos.

---

## `distributed_stress_test.py` (Python 2.6.6 - Corregido para errores ASCII)

```python
# -*- coding: utf-8 -*-
import os
import subprocess
import time
import math
import sys

def run_stress_on_slave(slave_ip_or_hostname, num_processes, duration_seconds):
    """
    Ejecuta un comando de stress en un nodo esclavo via SSH.
    Asume que tienes SSH sin contrasena configurado desde el maestro a los esclavos.
    """
    # Python 2.6.6 usa 'print' como una sentencia
    print "Enviando stress a %s con %s procesos por %s segundos..." % (slave_ip_or_hostname, num_processes, duration_seconds)
    
    # Comando de Python para consumir CPU en el esclavo.
    # Nota: Usamos 'python' en lugar de 'python3' para invocar Python 2.
    # Algunas funciones como range en Python 2.x pueden devolver listas muy grandes
    # para evitar problemas de memoria, podriamos usar xrange si fuera necesario
    # para iteraciones muy muy grandes, pero 10000 es manejable.
    stress_command = "python -c \"import math, time; start_time = time.time(); while time.time() - start_time < %s: [math.factorial(i %% 1000) for i in range(10000)];\"" % duration_seconds
    
    # El comando completo para ejecutar en el esclavo en segundo plano (nohup &)
    full_command = "ssh %s 'for i in $(seq %s); do nohup %s > /dev/null 2>&1 & done'" % (slave_ip_or_hostname, num_processes, stress_command)

    try:
        # En Python 2.6, Popen de subprocess.Popen funciona de manera similar.
        # shell=True es necesario para que el shell interprete el comando completo.
        subprocess.Popen(full_command, shell=True) 
        print "Comando de stress enviado a %s" % slave_ip_or_hostname
    except Exception as e:
        print "Error al enviar stress a %s: %s" % (slave_ip_or_hostname, e)

def get_slave_hostnames():
    """
    Obtiene la lista de nombres de host de los nodos esclavos desde Rocks.
    """
    try:
        # En Python 2, 'subprocess.check_output' no existe. Usamos 'subprocess.Popen' y 'communicate'.
        # 'text=True' no existe en Python 2, la salida es bytes.
        proc = subprocess.Popen(['/opt/rocks/bin/rocks', 'list', 'host'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        stdout_data, stderr_data = proc.communicate()
        
        # La salida es bytes, asi que necesitamos decodificarla. 'utf-8' es una buena opcion.
        # Si hay un error, proc.returncode sera distinto de 0.
        if proc.returncode != 0:
            raise Exception("Error al ejecutar 'rocks list host': %s" % stderr_data.strip())

        lines = stdout_data.decode('utf-8').splitlines() # Decodificamos y separamos por lineas
        slaves = []
        for line in lines:
            if line.startswith("compute"): # Los nodos esclavos en Rocks se nombran 'compute-X-Y'
                hostname = line.split()[0]
                slaves.append(hostname)
        return slaves
    except Exception as e:
        print "Error al obtener nombres de host de esclavos: %s" % e
        return []

if __name__ == "__main__":
    # Parametros configurables
    NUM_PROCESSES_PER_SLAVE = 2  # Numero de procesos "stress" a ejecutar en cada esclavo
    STRESS_DURATION_SECONDS = 60 # Duracion del stress en segundos

    print "Iniciando test de stress en el cluster Rocks..."

    slave_hosts = get_slave_hostnames()

    if not slave_hosts:
        print "No se encontraron nodos esclavos. Asegurate de que esten registrados y activos."
        sys.exit(1)

    print "Nodos esclavos detectados: %s" % ', '.join(slave_hosts)

    for slave in slave_hosts:
        run_stress_on_slave(slave, NUM_PROCESSES_PER_SLAVE, STRESS_DURATION_SECONDS)

    print "\nScripts de stress enviados a los nodos esclavos. Esperando %s segundos para monitorear el impacto..." % STRESS_DURATION_SECONDS
    print "Mientras tanto, abre la interfaz web de Ganglia en tu navegador para ver los resultados."
    
    time.sleep(STRESS_DURATION_SECONDS + 10) # Da tiempo extra para que Ganglia actualice

    print "\nTest de stress finalizado. Revisa Ganglia para el analisis completo."