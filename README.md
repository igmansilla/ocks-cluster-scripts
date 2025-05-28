# Contenido del Script Python (Adaptado para Python 2.6.6)

Este archivo Markdown contiene el codigo fuente de scripts Python adaptados para Python 2.6.6. Ten en cuenta que esta es una version antigua y el codigo se ajusta a sus particularidades. Puedes copiarlo y ejecutarlo directamente en tu nodo maestro de Rocks o en cualquier otro entorno Python 2.6.6.

---

## `simple_cpu_stress.py` (Python 2.6.6)

```python
import time
import math
import os
import sys # Para sys.exit

# Python 2.6.6 usa 'print' como una sentencia, no una funcion
print "Iniciando script simple de stress de CPU en el nodo maestro..."
print "Este script realizara calculos intensivos para consumir CPU."

# Duracion del stress en segundos
STRESS_DURATION_SECONDS = 30

start_time = time.time()
counter = 0

while time.time() - start_time < STRESS_DURATION_SECONDS:
    # Realiza un calculo intensivo de CPU
    _ = math.factorial(2000) # Calcula el factorial de un numero grande
    counter += 1

end_time = time.time()

# Usando str() y format para compatibilidad con print sentencia
print "\nScript de stress de CPU finalizado despues de %s segundos." % round(end_time - start_time, 2)
print "Realizo %s iteraciones de calculos de factorial." % counter
print "Ahora, puedes ir a la interfaz web de Ganglia para ver el impacto en la CPU del nodo maestro."