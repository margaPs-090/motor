# Motor de inferencia: diagnóstico de red

Pequeño **sistema experto basado en reglas** escrito en Python. Usa una base de conocimiento de reglas del tipo *SI premisas ENTONCES conclusión* para diagnosticar problemas de red de un equipo del aula.

## Características

- **Encadenamiento hacia delante** (`adelante`): a partir de unos hechos iniciales, deduce todo lo que se puede concluir.
- **Encadenamiento hacia atrás** (`atras`): parte de un objetivo e intenta demostrarlo, preguntando al usuario por los datos que no conoce.
- **Resolución de conflictos** configurable: por orden o por especificidad.
- **Refracción**: una regla no se dispara dos veces, lo que evita bucles infinitos.
- **Módulo de explicación** (`explicar`): muestra el árbol que justifica cómo se llegó a una conclusión.
- **Pregunta «porque»**: durante la consulta, el usuario puede preguntar por qué se le pide un dato.
- Base de conocimiento externa en **YAML o JSON**.

## Estructura del proyecto

```
motor/
├── motor.py          # Motor de inferencia
├── test_motor.py     # Pruebas con pytest
├── red.yaml          # Base de conocimiento (reglas)
├── requirements.txt  # Dependencias
└── README.md
```

## Requisitos

- Python 3.9 o superior (probado con 3.12 y 3.14)
- Las librerías de `requirements.txt`: `pyyaml` y `pytest`

## Instalación

1. Clona el repositorio y entra en la carpeta:

   ```bash
   git clone https://github.com/margaPs-090/motor.git
   cd TU_REPOSITORIO
   ```

2. Crea el entorno virtual:

   ```bash
   python -m venv venv
   ```

   En Mac y Linux puede que tengas que usar `python3` en lugar de `python`.

3. Actívalo:

   | Sistema | Comando |
   |---|---|
   | Linux / Mac | `source venv/bin/activate` |
   | Windows (PowerShell) | `venv\Scripts\Activate.ps1` |
   | Windows (cmd) | `venv\Scripts\activate.bat` |

4. Instala las dependencias:

   ```bash
   pip install -r requirements.txt
   ```

## Uso

Ejecuta el programa de ejemplo, que intenta demostrar `problema_dns` y te pregunta lo que no sabe:

```bash
python motor.py
```

Durante la consulta puedes responder:

- `s`: sí, es cierto.
- `n`: no.
- `porque`: muestra para qué regla se necesita el dato y vuelve a preguntar.

### Uso como librería

```python
from motor import cargar, adelante, atras, explicar, por_especificidad

reglas = cargar("red.yaml")

# Encadenamiento hacia delante
hechos, traza, origen = adelante(
    reglas,
    {"sin_navegacion", "ping_gateway_falla", "sin_ip", "led_red_encendido"},
    estrategia=por_especificidad,
)
print("\n".join(explicar("accion_renovar_ip", origen)))

# Encadenamiento hacia atrás (sin preguntar nada: todo lo desconocido es falso)
hechos = {"ping_gateway_falla", "sin_ip", "led_red_apagado"}
print(atras("accion_revisar_cable", reglas, hechos, lambda hecho, pila: False))
```

Salida de `explicar`:

```
accion_renovar_ip: por R9
  problema_dhcp: por R8
    problema_dhcp_o_cable: por R5
      ping_gateway_falla: dato inicial
      sin_ip: dato inicial
    led_red_encendido: dato inicial
```

## Formato de la base de conocimiento

Las reglas se definen en un archivo `.yaml`, `.yml` o `.json`:

```yaml
reglas:
  - id: R1
    si: [sin_navegacion, ping_ip_externa_ok]
    entonces: problema_dns
  - id: R2
    si: [problema_dns]
    entonces: accion_revisar_config_dns
```

Cada regla tiene un identificador único (`id`), una lista de premisas que deben cumplirse todas (`si`) y una conclusión (`entonces`).

## Tests

Desde la carpeta del proyecto, con el entorno virtual activado:

```bash
pytest -v
```

Los tests usan rutas relativas (`red.yaml`), así que hay que ejecutarlos desde la carpeta donde está ese archivo.

## Cómo funciona

| Función | Qué hace |
|---|---|
| `cargar(ruta)` | Lee las reglas de un archivo YAML o JSON y devuelve una lista de objetos `Regla`. |
| `adelante(reglas, hechos, estrategia)` | Dispara reglas hasta que no queda ninguna aplicable. Devuelve los hechos, la traza y el origen de cada hecho deducido. |
| `atras(objetivo, reglas, hechos, preguntar)` | Intenta demostrar un objetivo, preguntando por las premisas primitivas. |
| `explicar(hecho, origen)` | Devuelve el árbol de justificación de un hecho como lista de líneas. |
| `por_orden`, `por_especificidad` | Estrategias para elegir una regla cuando hay varias aplicables. |
| `preguntar_consola(hecho, pila)` | Pregunta por consola y soporta la respuesta «porque». |
