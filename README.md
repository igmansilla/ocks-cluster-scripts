# Script de Stress Extremo para Nodo Maestro (Python 2.6.6)

Este script esta disenado para consumir una cantidad significativa de CPU y memoria en el nodo donde se ejecuta (tu nodo maestro de Rocks). No intenta conectarse ni afectar a otros nodos del cluster.

**Advertencia:** Este script es intensivo. Asegurate de que tu nodo maestro tiene suficientes recursos y monitorea su comportamiento con Ganglia mientras se ejecuta. Podria ralentizar el sistema.

---

## `extreme_master_stress.py` (Python 2.6.6 - Sin tildes, solo para el maestro)

```python
# -*- coding: utf-8 -*-
import time
import math
import sys

# --- Parametros Configurables ---
CPU_STRESS_PROCESSES = 4 # Numero de procesos o hilos de CPU intensivo
                          # Ajusta segun el numero de CPUs/cores de tu maestro
MEM_ALLOC_MB = 2048       # Cantidad de memoria a intentar asignar en MB (2GB por defecto)
                          # Ajusta segun la RAM disponible en tu maestro (ej. 2GB RAM -> 1500MB)
STRESS_DURATION_SECONDS = 180 # Duracion total del stress en segundos (3 minutos)
# --------------------------------

def cpu_intensive_task(duration):
    """Realiza calculos intensivos de CPU durante una duracion especifica."""
    end_time = time.time() + duration
    counter = 0
    while time.time() < end_time:
        _ = math.factorial(5000) # Un numero grande para calculos pesados
        counter += 1
    # print "  CPU task finished after %s iterations." % counter # Descomentar para debug

def memory_intensive_task(size_mb):
    """Intenta asignar una gran cantidad de memoria."""
    print "Intentando asignar %s MB de memoria..." % size_mb
    try:
        # En Python 2, string * N crea un string de N copias
        # ' ' * 1024 * 1024 crea 1MB de espacios.
        # Creamos una lista de estos para evitar que el GC lo recoja inmediatamente.
        data = []
        for i in range(size_mb):
            data.append(' ' * 1024 * 1024) # 1 MB de datos por elemento
        print "Asignados %s MB de memoria. Manteniendo en RAM por un tiempo." % size_mb
        return data # Devolvemos para mantener la referencia y evitar GC
    except MemoryError:
        print "Error: No se pudo asignar %s MB de memoria. Puede que no haya suficiente RAM disponible." % size_mb
        return None
    except Exception as e:
        print "Ocurrio un error al asignar memoria: %s" % e
        return None

if __name__ == "__main__":
    print "Iniciando test de stress extremo en el nodo maestro..."
    print "Este script estresara la CPU y intentara consumir memoria."

    # --- Stress de CPU ---
    # Para Python 2.6.6, no tenemos multiprocessing.Pool de forma sencilla.
    # Simulamos procesos concurrentes con subprocesos o simplemente ejecutamos un loop largo.
    # Si quieres procesos realmente paralelos en Python 2.6.6, necesitarias el modulo 'multiprocessing'
    # (que puede requerir instalarlo si no viene con 2.6.6) y usar Process.start().
    # Por simplicidad y para asegurar que siempre funcione, lo haremos secuencialmente o con subprocesos
    # si se desea emular un poco de paralelismo sin depender de modulos avanzados.

    # Option 1: CPU intensivo en un solo hilo/proceso (mas sencillo)
    print "Ejecutando tarea intensiva de CPU durante %s segundos..." % STRESS_DURATION_SECONDS
    cpu_intensive_task(STRESS_DURATION_SECONDS)

    # Option 2: Ejecutar multiples procesos de CPU en paralelo (requiere 'multiprocessing' si quieres control)
    # Si no tienes 'multiprocessing' instalado, puedes lanzar via 'nohup python -c ... &'
    # Esto es mas complejo para un script simple sin SSH
    # Asi que la Option 1 es la mas directa para "solo haga trabajar al maestro"

    # --- Stress de Memoria ---
    allocated_memory = memory_intensive_task(MEM_ALLOC_MB)
    if allocated_memory:
        print "Memoria asignada. Manteniendola durante %s segundos..." % STRESS_DURATION_SECONDS
        time.sleep(STRESS_DURATION_SECONDS) # Mantenemos la memoria asignada

    print "\nTest de stress extremo finalizado."
    print "Revisa la interfaz web de Ganglia para ver el impacto en la CPU y memoria del nodo maestro."