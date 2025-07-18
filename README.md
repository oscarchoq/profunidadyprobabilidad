# Explicación Detallada: Profundidad y Probabilidad de Error en IA de Damas

## 1. Configuración de Niveles de Dificultad

En el archivo configuracion.py, se definen tres niveles de dificultad:

```python
NIVELES_DIFICULTAD = {
    1: {
        "nombre": "Principiante",
        "profundidad": 1,
        "error_probabilidad": 0.3,  # 30% de probabilidad de hacer un movimiento subóptimo
        "descripcion": "IA muy básica, comete errores frecuentes"
    },
    2: {
        "nombre": "Intermedio",
        "profundidad": 3,
        "error_probabilidad": 0.1,  # 10% de probabilidad de error
        "descripcion": "IA competente, pocos errores"
    },
    3: {
        "nombre": "Experto",
        "profundidad": 5,
        "error_probabilidad": 0.0,  # Sin errores intencionales
        "descripcion": "IA máxima, juego perfecto"
    }
}
```

### Significado de cada parámetro:

- **`profundidad`**: Número de movimientos que la IA puede "ver" hacia adelante
- **`error_probabilidad`**: Probabilidad de que la IA elija intencionalmente un movimiento subóptimo

## 2. Implementación de la Profundidad

### 2.1 Obtención de la Profundidad desde la Configuración

```python
class ConfiguracionIA:
    def __init__(self, nivel=3):
        self.nivel_actual = nivel
    
    def obtener_profundidad(self):
        # Devuelve la profundidad del nivel actual
        return NIVELES_DIFICULTAD[self.nivel_actual]["profundidad"]
```

### 2.2 Uso de la Profundidad en el Algoritmo Minimax

```python
class AlgoritmoMinimax:
    def obtener_mejor_movimiento(self, tablero, jugador_actual):
        if tablero.es_final(jugador_actual):
            return None
        
        # ✅ AQUÍ SE OBTIENE LA PROFUNDIDAD DEL NIVEL SELECCIONADO
        profundidad_maxima = self.config.obtener_profundidad()
        
        if jugador_actual == JUGADOR_BLANCO:
            _, mejor_movimiento = self._max_valor(tablero, profundidad_maxima, JUGADOR_BLANCO)
        else:
            _, mejor_movimiento = self._min_valor(tablero, profundidad_maxima, JUGADOR_NEGRO)
        
        # [Código de aplicación de errores aquí...]
        
        return mejor_movimiento
```

### 2.3 Recursión con Control de Profundidad

```python
def _max_valor(self, tablero, profundidad, jugador_turno):
    # ✅ CONDICIÓN DE PARADA: Profundidad 0 o juego terminado
    if tablero.es_final(jugador_turno) or profundidad == 0:
        return self.evaluador.calcular_utilidad(tablero, jugador_turno), None
    
    mejor_valor = -math.inf
    mejor_movimiento = None
    
    # ✅ EXPLORACIÓN RECURSIVA: Cada llamada reduce la profundidad en 1
    for movimiento in tablero.movimientos_disponibles(jugador_turno):
        nuevo_tablero = tablero.aplicar_movimiento(movimiento)
        oponente = tablero.obtener_jugador_oponente(jugador_turno)
        
        # 🔍 AQUÍ SE REDUCE LA PROFUNDIDAD: profundidad - 1
        valor, _ = self._min_valor(nuevo_tablero, profundidad - 1, oponente)
        
        if valor > mejor_valor:
            mejor_valor = valor
            mejor_movimiento = movimiento
    
    return mejor_valor, mejor_movimiento
```

### 2.4 Ejemplo Visual del Árbol de Búsqueda

```
Profundidad 3 - Nivel Intermedio:

Profundidad 3: [MI TURNO] - Evalúo mis opciones
    ├─ Movimiento A
    │   └─ Profundidad 2: [TU TURNO] - ¿Qué harías?
    │       ├─ Tu respuesta 1
    │       │   └─ Profundidad 1: [MI TURNO] - Mi contra-ataque
    │       │       └─ Profundidad 0: EVALUAR ⭐
    │       └─ Tu respuesta 2
    │           └─ Profundidad 1: [MI TURNO] - Mi contra-ataque
    │               └─ Profundidad 0: EVALUAR ⭐
    └─ Movimiento B
        └─ Profundidad 2: [TU TURNO] - ¿Qué harías?
            └─ [Similar expansión...]
```

## 3. Implementación de la Probabilidad de Error

### 3.1 Determinación de si Debe Cometer Error

```python
class ConfiguracionIA:
    def debe_cometer_error(self):
        # Obtiene la probabilidad de error del nivel actual
        probabilidad_error = NIVELES_DIFICULTAD[self.nivel_actual]["error_probabilidad"]
        
        # ✅ random.random() genera un número entre 0.0 y 1.0
        # Si es menor que la probabilidad de error, comete el error
        return random.random() < probabilidad_error
```

### 3.2 Ejemplos de Funcionamiento del Error

```python
# Ejemplos con diferentes niveles:

# Nivel 1 - Principiante (30% error)
random.random() = 0.25  →  0.25 < 0.3  →  ✅ COMETE ERROR
random.random() = 0.85  →  0.85 < 0.3  →  ❌ NO comete error

# Nivel 2 - Intermedio (10% error)  
random.random() = 0.05  →  0.05 < 0.1  →  ✅ COMETE ERROR
random.random() = 0.45  →  0.45 < 0.1  →  ❌ NO comete error

# Nivel 3 - Experto (0% error)
random.random() = 0.99  →  0.99 < 0.0  →  ❌ NUNCA comete error
```

### 3.3 Aplicación del Error en el Movimiento Final

```python
def obtener_mejor_movimiento(self, tablero, jugador_actual):
    # ... [Código del cálculo Minimax] ...
    
    # ✅ DESPUÉS de calcular el mejor movimiento, verificar si debe cometer error
    if self.config.debe_cometer_error() and mejor_movimiento:
        # Obtener TODOS los movimientos disponibles
        movimientos_disponibles = list(tablero.movimientos_disponibles(jugador_actual))
        
        # ✅ SOLO si hay más de una opción (para evitar errores si solo hay 1 movimiento)
        if len(movimientos_disponibles) > 1:
            # 🚫 REMOVER el mejor movimiento de las opciones
            movimientos_disponibles.remove(mejor_movimiento)
            
            # 🎲 ELEGIR ALEATORIAMENTE entre los movimientos subóptimos
            mejor_movimiento = random.choice(movimientos_disponibles)
    
    return mejor_movimiento
```

## 4. Ejemplo Completo Paso a Paso

### Escenario: IA Nivel 2 (Intermedio) evalúa una posición

```python
# PASO 1: Configuración inicial
nivel = 2
profundidad_maxima = 3  # Del nivel intermedio
probabilidad_error = 0.1  # 10% de probabilidad de error

# PASO 2: Ejecución del Minimax
# La IA explora el árbol hasta profundidad 3
def _max_valor(tablero, profundidad=3, jugador_turno):
    # Evalúa: Mi movimiento → Tu respuesta → Mi contra → EVALUAR
    
    # Supongamos que encuentra estos valores para cada movimiento:
    movimientos_evaluados = {
        "Mover_pieza_A": 85,   # ⭐ MEJOR MOVIMIENTO
        "Mover_pieza_B": 70,
        "Mover_pieza_C": 60,
        "Mover_pieza_D": 45
    }
    
    mejor_movimiento = "Mover_pieza_A"  # El óptimo según Minimax

# PASO 3: Verificación de error
debe_error = random.random() < 0.1  # Supongamos que sale 0.15

if debe_error:  # 0.15 < 0.1 → False, NO comete error
    # La IA mantiene su mejor movimiento
    movimiento_final = "Mover_pieza_A"
else:
    # Si hubiera sido True, habría elegido entre B, C, o D aleatoriamente
    pass

# RESULTADO: La IA ejecuta "Mover_pieza_A" (el óptimo)
```

## 5. Diferencias de Comportamiento por Nivel

### Nivel 1 - Principiante
```python
profundidad = 1      # Solo ve SU próximo movimiento
error_probabilidad = 0.3  # 30% de movimientos subóptimos

# Comportamiento:
# - Evaluación superficial (no anticipa respuestas)
# - Frecuentes decisiones impulsivas
# - Vulnerable a trampas simples
```

### Nivel 2 - Intermedio
```python
profundidad = 3      # Ve 3 movimientos adelante
error_probabilidad = 0.1  # 10% de movimientos subóptimos

# Comportamiento:
# - Planificación táctica básica
# - Evita la mayoría de trampas obvias
# - Ocasionalmente comete errores "humanos"
```

### Nivel 3 - Experto
```python
profundidad = 5      # Ve 5 movimientos adelante
error_probabilidad = 0.0  # Jamás comete errores

# Comportamiento:
# - Planificación estratégica avanzada
# - Juego "perfecto" dentro de su alcance
# - Extremadamente difícil de vencer
```

## 6. Impacto en el Rendimiento Computacional

### Complejidad Exponencial de la Profundidad

```python
# Aproximación del número de posiciones evaluadas:
# (Asumiendo ~8 movimientos promedio por posición)

Profundidad 1: ~8¹ = 8 posiciones
Profundidad 3: ~8³ = 512 posiciones  
Profundidad 5: ~8⁵ = 32,768 posiciones

# Tiempo estimado:
Nivel 1: ~0.001 segundos ⚡
Nivel 2: ~0.050 segundos 🟡
Nivel 3: ~1.500 segundos 🔴
```

### Optimización con Alfa-Beta

```python
class AlgoritmoMinimaxAlfaBeta:
    def _max_valor(self, tablero, alfa, beta, profundidad, jugador_turno):
        # ... [código similar] ...
        
        for movimiento in tablero.movimientos_disponibles(jugador_turno):
            valor, _ = self._min_valor(nuevo_tablero, alfa, beta, profundidad - 1, oponente)
            
            # ✅ PODA ALFA-BETA: Evita evaluar ramas innecesarias
            alfa = max(alfa, mejor_valor)
            if beta <= alfa:
                break  # 🚫 PODA: No evalúa más movimientos
        
        # Resultado: Reduce significativamente el tiempo de cálculo
```

Esta implementación crea una progresión natural y realista de dificultad, simulando tanto la profundidad de análisis como los errores humanos típicos de jugadores de diferentes niveles.
