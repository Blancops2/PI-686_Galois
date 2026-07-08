# Extracción de Grupos de Galois desde LMFDB — Documentación de archivos

## Fuente de los datos

Todos los datos contenidos en estos archivos provienen de **LMFDB** (*The L-functions and Modular Forms Database*, [www.lmfdb.org](https://www.lmfdb.org)), específicamente de dos tablas de su base de datos:

- **`gps_transitive`**: metadatos de los grupos de Galois transitivos (propiedades algebraicas del grupo: orden, si es abeliano, resoluble, etc.).
- **`nf_fields`**: campos numéricos (polinomios mónicos irreducibles) junto con sus invariantes aritméticos (discriminante, número de clase, regulador, etc.), filtrados por grado y por el grupo de Galois asociado (`galt`).

La conexión se hace mediante la librería `lmfdb-lite`, que expone acceso directo a la base de datos PostgreSQL de LMFDB.

**Importante:** LMFDB no es una base de datos exhaustiva de *todos* los polinomios posibles — es una recopilación de varios proyectos de investigación (Klüners–Malle, Jones–Roberts, entre otros), cada uno con métodos y límites de búsqueda distintos según el grupo de Galois. Por eso el número de polinomios almacenados para cada grupo no es proporcional al orden del grupo (ver sección "Notas sobre completitud" más abajo).

---

## Qué hemos hecho

Extrajimos, para dos grados distintos, **todos los polinomios mónicos irreducibles** disponibles en LMFDB para **cada uno de los grupos transitivos** de ese grado:

| Grado | Grupo simétrico | # de grupos transitivos | Archivos generados |
|---|---|---|---|
| 9  | $S_9$  | 34 (`9T1`…`9T34`)  | `galois_d9_json_separado/`, `galois_d9_COMPLETO.json` |
| 10 | $S_{10}$ | 45 (`10T1`…`10T45`) | `galois_d10_json_separado/`, `galois_d10_COMPLETO.json` |

Para cada grado se generaron **dos versiones equivalentes en contenido, pero con distinta organización de archivo**:

### 1. Versión `_COMPLETO` (un solo archivo JSON gigante)

- `galois_d9_COMPLETO.json`
- `galois_d10_COMPLETO.json`

Es un **único archivo** que contiene un diccionario con **todos** los grupos transitivos de ese grado como claves. Fue la primera forma en que se extrajeron los datos (versión original, adaptada de la extracción de $S_8$).

```json
{
  "9T1":  { "label": "9T1",  "degree": 9, "t_index": 1,  "group_metadata": {...}, "polynomials": [...] },
  "9T2":  { "label": "9T2",  "degree": 9, "t_index": 2,  "group_metadata": {...}, "polynomials": [...] },
  ...
  "9T34": { "label": "9T34", "degree": 9, "t_index": 34, "group_metadata": {...}, "polynomials": [...] }
}
```

**Ventaja:** un solo archivo, fácil de mover.
**Desventaja:** para leer un solo grupo (ej. `9T3`) hay que cargar el archivo completo en memoria — con grados grandes (como 10) puede pesar varios cientos de MB.

### 2. Versión `_separado` (un JSON por cada grupo transitivo)

- `galois_d9_json_separado/9T1.json`, `9T2.json`, … `9T34.json`
- `galois_d10_json_separado/10T1.json`, `10T2.json`, … `10T45.json`

Es una **carpeta** donde cada grupo transitivo tiene su propio archivo independiente. Fue la adaptación que hicimos después, para no tener que cargar todo en memoria si solo se necesita consultar un grupo puntual.

```
galois_d10_json_separado/
├── 10T1.json
├── 10T2.json
├── 10T3.json
├── ...
└── 10T45.json
```

**Ventaja:** cada archivo es pequeño e independiente; se puede abrir/subir/consultar uno solo sin tocar los demás.
**Desventaja:** si se necesitan todos a la vez, hay que iterar sobre la carpeta y juntarlos (ver más abajo).

### Ambas versiones contienen exactamente la misma información

La única diferencia es **estructural** (cómo está organizado el archivo/carpeta), no de contenido. Cada entrada de `_COMPLETO.json` es idéntica al archivo correspondiente en `_separado/`:

```python
# Esto es siempre verdadero:
completo["9T3"] == json.load(open("galois_d9_json_separado/9T3.json"))
```

Y se puede reconstruir una versión a partir de la otra en cualquier momento:

```python
import json, pathlib

# de "separado" a "completo"
repositorio = {}
for path in sorted(pathlib.Path("galois_d9_json_separado").glob("9T*.json"),
                    key=lambda p: int(p.stem.split("T")[1])):
    with open(path, encoding="utf-8") as f:
        repositorio[path.stem] = json.load(f)
```

---

## Estructura de cada entrada de grupo (ambas versiones)

Ya sea dentro de `_COMPLETO.json[label]` o como archivo independiente `label.json`, la estructura es siempre la misma:

```json
{
  "label": "9T1",
  "degree": 9,
  "t_index": 1,

  "group_metadata": {
    "label": "9T1",
    "order": 9,
    "name": "C(9)=9",
    "pretty": "$C_9$",
    "prim": 0,
    "solv": 1,
    "cyc": 1,
    "ab": 1,
    "parity": 1,
    "nilpotency": 1,
    "num_conj_classes": 9,
    "gapidfull": "[9,1]",
    "abstract_label": "9.1",
    "subfields": [ [[3, 1], 1] ],
    "siblings": [],
    "quotients": [ [3, [3, 1], 1] ],
    "arith_equiv": 0,
    "transitivity": 1,
    "gens": [[[1,2,3,4,5,6,7,8,9]]],
    "auts": 9,
    "malle_ainv": 6,
    "malle_b": 1
  },

  "polynomials": [
    {
      "label": "9.9.16983563041.1",
      "coeffs": [-1, 5, 10, -20, -15, 21, 7, -8, -1, 1],
      "disc_abs": 16983563041,
      "disc_sign": 1,
      "r2": 0,
      "ramps": [19],
      "num_ram": 1,
      "rd": 13.698400752,
      "grd": 13.69840075203899,
      "class_number": 1,
      "regulator": 87.132402653,
      "galois_label": "9T1",
      "is_galois": true,
      "cm": false,
      "conductor": 19,
      "narrow_class_number": 1,
      "used_grh": false
    }
    // ... el resto de polinomios de este grupo
  ]
}
```

### Explicación de los campos

**Nivel raíz**

| Campo | Contenido |
|---|---|
| `label` | Etiqueta del grupo transitivo, ej. `9T1`, `10T45` |
| `degree` | Grado del polinomio (9 o 10) |
| `t_index` | El número "T" (el índice dentro de ese grado, ej. el `1` de `9T1`) |
| `group_metadata` | Metadatos algebraicos del grupo (ver abajo) |
| `polynomials` | Lista de todos los polinomios mónicos irreducibles con ese grupo de Galois disponibles en LMFDB |

**`group_metadata`** (tabla `gps_transitive` de LMFDB)

| Campo | Significado |
|---|---|
| `label` | Mismo label del grupo |
| `order` | Orden del grupo (cantidad de elementos) |
| `name` | Nombre corto del grupo (ej. `C9`, `S10`) |
| `pretty` | Nombre en formato LaTeX para mostrar (ej. `$C_9$`) |
| `prim` | Si el grupo es primitivo (`1`) o no (`0`) — **entero, no booleano** |
| `solv` | Si el grupo es resoluble (`1`/`0`) |
| `cyc` | Si el grupo es cíclico (`1`/`0`) |
| `ab` | Si el grupo es abeliano (`1`/`0`) |
| `parity` | `1` = par (subgrupo de $A_n$), distinto de `1` = impar |
| `nilpotency` | Clase de nilpotencia (`-1` si no es nilpotente) |
| `num_conj_classes` | Número de clases de conjugación |
| `gapidfull` | ID del grupo en la biblioteca GAP, como **string** `"[orden, índice]"` |
| `abstract_label` | Etiqueta del grupo abstracto subyacente |
| `subfields` | Subcampos: lista con formato `[[n, t], multiplicidad]` |
| `siblings` | Grupos "hermanos" (misma acción, distinto conjunto): mismo formato que `subfields` |
| `quotients` | Cocientes del grupo: formato `[orden_cociente, [n, t], multiplicidad]` (nota el campo extra al inicio respecto a `subfields`) |
| `arith_equiv` | Índice de equivalencia aritmética |
| `transitivity` | Grado de transitividad de la acción del grupo |
| `gens` | Generadores del grupo, como lista de ciclos de permutación |
| `auts` | Número de automorfismos del grupo |
| `malle_ainv`, `malle_b` (`malle_c`, `malle_d` si existen) | Constantes de la conjetura de Malle (conteo asintótico de campos numéricos); no todos los grupos tienen las 4 definidas en LMFDB |

**`polynomials`** (tabla `nf_fields` de LMFDB — una entrada por cada campo numérico con ese grupo de Galois)

| Campo | Significado |
|---|---|
| `label` | Etiqueta LMFDB del campo numérico, ej. `9.9.16983563041.1` |
| `coeffs` | Coeficientes del polinomio mónico en orden ascendente `[a0, a1, ..., an]` (siempre `an = 1`) |
| `disc_abs` | Valor absoluto del discriminante |
| `disc_sign` | Signo del discriminante (`1` o `-1`) |
| `r2` | Número de pares de embebimientos complejos |
| `ramps` | Lista de primos que ramifican |
| `num_ram` | Cantidad de primos que ramifican |
| `rd` | Raíz del discriminante (*root discriminant*) |
| `grd` | Discriminante de Galois relativo |
| `class_number` | Número de clase del campo numérico |
| `regulator` | Regulador del campo |
| `galois_label` | Grupo de Galois asociado (debe coincidir con `label` a nivel raíz) |
| `is_galois` | Si el campo mismo es una extensión de Galois |
| `cm` | Si es un campo CM (multiplicación compleja) |
| `conductor` | Conductor del campo |
| `narrow_class_number` | Número de clase estricto (*narrow class number*) |
| `used_grh` | Si el cálculo asumió la Hipótesis de Riemann Generalizada |

---

## Notas sobre completitud de los datos

Al comparar los conteos entre grupos del mismo grado, se observa que **el número de polinomios NO es proporcional al orden del grupo**. Por ejemplo, en grado 10, el grupo `10T43` (orden 28,800) tiene muchos más polinomios almacenados que `10T45 = S10` (orden 3,628,800, el grupo simétrico completo).

Esto es un comportamiento **esperado de LMFDB**, no un error de extracción:

- LMFDB es una compilación de varios proyectos de investigación distintos, cada uno con su propio método y límite de búsqueda (discriminante máximo explorado) según el grupo.
- Grupos resolubles se tabulan mediante teoría de cuerpos de clases, un método sistemático que permite explorar discriminantes muy grandes de forma relativamente eficiente.
- Grupos grandes o primitivos (como $A_n$, $S_n$) suelen buscarse con métodos de fuerza bruta acotados por discriminante, mucho más costosos computacionalmente — por lo que su catálogo en LMFDB suele ser menos profundo, aunque matemáticamente sean el "caso genérico" (la mayoría de polinomios aleatorios de un grado dado tienen Galois group $S_n$).

En resumen: estos archivos contienen **todo lo que existe actualmente en LMFDB** para cada grupo transitivo — ni más ni menos — reflejando el estado real y desigual de completitud de esa base de datos.
