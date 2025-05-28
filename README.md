# Contenido del Script Python

Este archivo Markdown contiene el codigo fuente de un script Python. Puedes copiarlo y ejecutarlo directamente en tu nodo maestro de Rocks o en cualquier otro entorno Python.

---

## `simple_cpu_stress.py`

```python
import time
import math
import os

print("Iniciando script simple de stress de CPU en el nodo maestro...")
print("Este script realizara calculos intensivos para consumir CPU.")

# Duracion del stress en segundos
STRESS_DURATION_SECONDS = 30

start_time = time.time()
counter = 0

while time.time() - start_time < STRESS_DURATION_SECONDS:
    # Realiza un calculo intensivo de CPU
    _ = math.factorial(2000) # Calcula el factorial de un numero grande
    counter += 1

end_time = time.time()

print(f"\nScript de stress de CPU finalizado despues de {round(end_time - start_time, 2)} segundos.")
print(f"Realizo {counter} iteraciones de calculos de factorial.")
print("Ahora, puedes ir a la interfaz web de Ganglia para ver el impacto en la CPU del nodo maestro.")