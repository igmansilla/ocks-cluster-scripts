import time
import math
import os

print("Iniciando script simple de stress de CPU en el nodo maestro...")
print("Este script realizará cálculos intensivos para consumir CPU.")

# Duración del stress en segundos
STRESS_DURATION_SECONDS = 30

start_time = time.time()
counter = 0

while time.time() - start_time < STRESS_DURATION_SECONDS:
    # Realiza un cálculo intensivo de CPU
    _ = math.factorial(2000) # Calcula el factorial de un número grande
    counter += 1

end_time = time.time()

print(f"\nScript de stress de CPU finalizado después de {round(end_time - start_time, 2)} segundos.")
print(f"Realizó {counter} iteraciones de cálculos de factorial.")
print("Ahora, puedes ir a la interfaz web de Ganglia para ver el impacto en la CPU del nodo maestro.")