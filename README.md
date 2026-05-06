# TP1-Propulsion-Grupo3
=====================================================================
  README — Predimensionamiento de Motores Alternativos Aeronáuticos
  Aplicación web · Modelo Aire-Estándar Frío · Atmósfera ISA
=====================================================================

OBJETIVO
--------
Calcular y comparar los ciclos termodinámicos ideales Otto, Diesel y
Sabathé bajo una misión aeronáutica dada (altitud, potencia, combustible
y mezcla), y dimensionar coherentemente la geometría del motor (cilin-
drada total, diámetro y carrera del pistón) para un motor de 4 cilindros.
La pestaña Geometría incluye además, a título informativo, la compara-
tiva de las soluciones equivalentes con 4, 6 y 8 cilindros.

CONFIGURACIÓN INICIAL POR DEFECTO
---------------------------------
Al abrir la aplicación los parámetros parten de los siguientes valores:
   · Ciclo                : Sabathé
   · r (Otto)             : 8.0
   · r (Diesel)           : 18.0
   · r (Sabathé)          : 12.0
   · k                    : 1.40
   · α (frac. Q a v=cte)  : 0.50
   · φ                    : 0.97
   · Combustible          : n-Heptano (C7H16)
   · Rendimiento mecánico : 0.85
   · RPM                  : 2700
   · Nº de cilindros      : 4 (fijo, no editable)
   · Tmax permitida       : 4200 K
   · pmax permitida       : 10000 kPa (100 bar)

CÓMO EJECUTAR
-------------
1) Abrir el archivo .html en cualquier navegador moderno (Chrome, Firefox,
   Edge, Safari). No requiere instalación, ni servidor, ni dependencias
   locales — sólo conexión a internet la PRIMERA vez para cargar Plotly y
   las tipografías desde CDN.
2) Configurar los parámetros de misión y ciclo en el panel de la izquierda.
3) Los resultados se actualizan AUTOMÁTICAMENTE en cuanto cambia cualquier
   entrada (no hay botón "Calcular": todo es reactivo, recalculando una
   vez por frame para mantener la UI fluida).
4) Navegar entre las pestañas para ver:
     · Resumen
     · Estados Termodinámicos
     · Diagramas p-v / T-s
     · Análisis Paramétrico (η, PME, Tmax vs r)
     · Comparación de Ciclos
     · Geometría del Motor
5) Hacer clic en "⤢ Expandir" sobre cualquier gráfico para verlo en grande.
6) Hover sobre las curvas: aparecen las coordenadas (sólo sobre la línea
   trazada — no sobre áreas vacías).

ENTRADAS
--------
A) Misión:
   · Altitud de operación        [ft o m]    selector de unidad
   · Potencia efectiva requerida [HP o kW]   selector de unidad
   · Combustible (6 hidrocarburos puros, por defecto n-Heptano).
   · Relación de mezcla relativa φ = (F/A)/(F/A)_estequ.   [0 a 1,4]
     ·· φ = 0 representa el caso límite "sin combustible" (Q_in = 0):
        no hay aporte térmico, el ciclo es degenerado y se emite una
        alerta informativa.
B) Parámetros del ciclo:
   · Tipo de ciclo               [Otto | Diesel | Sabathé]
                                 (predeterminado: Sabathé)
   · TRES relaciones de compresión, una por ciclo (r_Otto, r_Diesel,
     r_Sabathé), cada una entre 1 y 22 (o 25 para Diesel). Esto permite
     comparar los tres ciclos cada uno operando en su rango de r realista.
     Para el ciclo Diesel, el r mínimo es dinámico y se calcula para
     garantizar que la combustión ocurra antes del PMI.
   · Coeficiente k = cp/cv       [1,0 a 1,7]
   · Si Sabathé: fracción α de Q_in a v=cte    [0..1]
C) Hipótesis mecánicas:
   · Rendimiento mecánico η_m    [0,70 a 1,00]   default 0,85
   · RPM nominal del motor       [valor numérico]
   · Nº de cilindros             FIJO en 4 (no es editable). La pestaña
     Geometría muestra los tres tamaños 4/6/8 sólo como referencia
     comparativa.
D) Límites de seguridad (configurables):
   · Tmax permitida (default 4200 K)
   · pmax permitida (default 10000 kPa = 100 bar)

COMBUSTIBLES DISPONIBLES
------------------------
La estequiometría usa la reacción genérica para cualquier hidrocarburo:
    CxHy + (x + y/4) O2  →  x CO2 + (y/2) H2O
con aire de 23,2% másico de O2. AFR_real = AFR_estequ / φ.

   Familia      Fórmula   Nombre        PCI [kJ/kg]
   ──────────   ───────   ───────────   ────────────
   Alcanos      C6H14     n-Hexano      44750
                C7H16     n-Heptano     44560   (predeterminado)
                C8H18     Octano        44400
                C10H22    Decano        44240
   Aromáticos   C6H6      Benceno       40140
                C7H8      Tolueno       40520

Las propiedades vienen de NIST WebBook, Engineering Toolbox y Perry's
Chemical Engineers' Handbook (8ª ed.).

HIPÓTESIS DEL MODELO
--------------------
1) Fluido de trabajo: gas ideal (aire).
2) Procesos internamente reversibles e idealizados.
3) Calores específicos constantes (modelo aire-estándar FRÍO).
4) k = 1,40 ; R = 287 J/(kg·K) por defecto (ajustables).
5) Combustión = aporte de calor equivalente Q_in calculado a partir del
   PCI del combustible y la relación de mezcla.
6) Condiciones de entrada: atmósfera ISA en la altitud indicada.

CÁLCULO DEL RENDIMIENTO TÉRMICO
-------------------------------
Las expresiones empleadas para η en cada ciclo son las siguientes:

   Otto      η = 1 − 1 / r^(k−1)
             (balance de energía equivalente: η = 1 − Q_out/Q_in,
              con Q_in = cv·(T3−T2) y Q_out = cv·(T4−T1)).

   Diesel    η = 1 − (1 / r_v^(k−1)) · (r_c^k − 1) / [k · (r_c − 1)]
             donde r_v = V1/V2 (relación de compresión)
                   r_c = V3/V2 = T3/T2 (relación de corte).
             Esta fórmula cerrada es algebraicamente equivalente al
             balance Q_in = cp·(T3−T2), Q_out = cv·(T4−T1) (la
             aplicación calcula los estados por balance y obtiene η
             por la fórmula cerrada; verificadas idénticas hasta el
             error de coma flotante IEEE 754).

   Sabathé   η = 1 − Q_out / Q_in
             con Q_in = α·Q_in_total + (1−α)·Q_in_total partido en
             aporte a v=cte (2→3) y a p=cte (3→4), y
             Q_out = cv·(T5−T1).

SALIDAS
-------
Termodinámicas:
  · Condiciones de entrada (T, p, ρ, v) según ISA.
  · PCI del combustible y Q_in equivalente.
  · Estados (T, p, v, s) en cada punto del ciclo seleccionado.
  · Calores intercambiados (Q_in, Q_out) y trabajo neto W_neto.
  · Rendimiento térmico η.
  · Presión media efectiva PME [kPa].
  · Temperatura y presión máximas (Tmax [K], pmax [kPa]).

Geométricas:
  · Cilindrada total V_total para la potencia efectiva.
  · Geometría del motor: 4 cilindros (configuración fija de la app).
  · Cilindrada unitaria, diámetro y carrera (S/D ≈ 1,05).
  · Volumen de cámara de combustión Vc = Vd / (r-1).
  · Tabla comparativa con los valores que tendría el motor con 4, 6
    u 8 cilindros, a título informativo (no afecta el dimensionamiento).

Gráficas (interactivas, con tooltip de coordenadas):
  · Diagrama p-v del ciclo seleccionado.
  · Diagrama T-s del ciclo seleccionado.
  · η, PME y Tmax en función de r.
    En los gráficos Tmax vs r de los ciclos Diesel y Sabathé el
    eje horizontal se restringe a r > 7 (rango operativo realista
    de los ciclos por compresión). El gráfico de Otto conserva
    todo el barrido r ∈ [4, 22].
  · Comparación visual de los tres ciclos en p-v y T-s.
  · Botón de cámara fotográfica para exportar gráfico con fondo blanco.

ORGANIZACIÓN DE LAS PANTALLAS
-----------------------------
· Barra superior   : título y botón de descarga del README.
· Sidebar (izq.)   : todas las entradas (sin botón Calcular: auto-recalc).
  Los nombres de los parámetros incluyen tooltips explicativos (hover).
· Workspace (der.) : 6 pestañas con resultados (resumen → geometría).
· Panel de alertas : mensajes ✓ (OK) / ▲ (warn) / ✗ (error) / i (info).
· Modal de gráficos: al hacer clic en "Expandir".

CONTROLES INTERACTIVOS
----------------------
· Sliders para φ, r (×3 ciclos), k, α, η_mec.
  IMPORTANTE — comportamiento dual de los sliders:
    - Click + arrastre sobre el thumb: movimiento continuo.
    - Click sobre la pista: incrementa UN paso en esa dirección.
  Los valores numéricos a la derecha de los sliders son *inputs
  editables*: se puede escribir directamente el número deseado.
· Selectores de combustible (6 opciones), ciclo (3 opciones) y
  unidades (altitud: ft/m, potencia: HP/kW).
· Tooltips en todas las etiquetas del panel lateral: dejá el mouse
  arriba para ver descripciones conceptuales.
· Botón de descarga de gráficos en la barra flotante de Plotly
  adaptado para generar imágenes con fondo blanco aptas para informes.

ESTRUCTURA DEL CÓDIGO
---------------------
El código está organizado en módulos independientes, separando claramente
los cálculos termodinámicos del manejo de la interfaz:

   ATM     — Atmósfera ISA (T, p, ρ por altitud)
   COMB    — Propiedades de combustibles + AFR + Q_in
   CICLO   — Estados termodinámicos: otto(), diesel(), sabathe()
             (η de Diesel calculado por fórmula cerrada)
   GEO     — Geometría del motor: cilindrada, D, S, opciones 4/6/8
   VAL     — Validación de entradas, conversión de unidades, alertas

   PLOT    — Generación de gráficos Plotly (con limitación r>7 en
             Tmax vs r para Diesel/Sabathé)
   UI      — Lectura de inputs, render de tablas/cards/alertas, tabs, modal
             (Nº de cilindros fijado en 4 dentro de readInputs)
   README  — Generación del archivo de instrucciones (este archivo)

CONVENCIONES Y UNIDADES INTERNAS
--------------------------------
Toda la termodinámica se realiza en SI: K, Pa, m³, J, kg.
Las conversiones a unidades de presentación (kPa, kJ/kg, cm³, mm, %, °C)
ocurren ÚNICAMENTE en la capa de UI.

VALIDACIONES Y ALERTAS
----------------------
· Validación de tipo y rango sobre cada entrada.
· Alerta ▲ si Tmax supera el límite (default 4200 K).
· Alerta ▲ si pmax supera el límite (default 10000 kPa = 100 bar).
· Alerta ▲ si el ciclo Diesel se limita con r_min por expansión fuera de PMI.
· Error ✗ explícito ante PME ≤ 0 o η fuera de [0, 1].

COMENTARIOS SOBRE EL MODELO
---------------------------
El modelo aire-estándar frío sobreestima sistemáticamente Tmax y η porque
ignora la disociación a altas T, la variación de cp con T, las pérdidas
por transferencia de calor, la combustión finita y la fricción interna.

HISTORIAL DE VERSIONES
----------------------
v1 — Versión inicial.
v2 — Sliders duales, presiones en kPa, 12 combustibles, r independientes.
v3 — Decimales con coma (es-AR), auto-recalc optimizado, n-Heptano.
v4 — Límite dinámico (r mínimo) para Diesel basado en Q_in. Inputs
     numéricos editables junto a los sliders. Tooltips descriptivos en
     las etiquetas del sidebar. Botón "Descargar Gráfico" con tema
     claro/blanco y nombres de archivo generados dinámicamente. Rango
     de k extendido a [1,0; 1,7]. Tmax por defecto en 4800 K.
v5 — Cambios solicitados:
     · Configuración inicial por defecto: ciclo Sabathé, r=12,
       η_mec = 0,85, Tmax = 4200 K.
     · Reducción del catálogo de combustibles a 6 (Hexano, Heptano,
       Octano, Decano, Benceno, Tolueno). Heptano predeterminado.
     · Eliminación de la recomendación automática de cilindros y de
       la columna de evaluación. Predimensionamiento fijado en 4
       cilindros; la tabla 4/6/8 se conserva sólo a título informativo.
     · Cálculo de η del ciclo Diesel reescrito mediante fórmula
       analítica cerrada (verificado equivalente al balance de energía
       hasta el error numérico de coma flotante).
     · Gráficos Tmax vs r de Diesel y Sabathé restringidos a r > 7.
=====================================================================
