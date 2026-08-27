# TODO PASA · Oaky Edition — Cuaderno del proyecto

> Documento maestro. Qué es el juego, cómo está hecho, qué se hizo, qué falta
> y con qué reglas se trabaja. Actualizado al último commit.

---

## 1 · QUÉ ES

Un juego de carrera de futbolista argentino. Empezás siendo un pibe de
inferiores y llegás hasta el retiro, o hasta dirigir, o hasta presidente de un
club. Se juega tomando decisiones sobre cartas: cada pantalla te planta una
situación y seis opciones, y lo que elegís mueve tus números, tu reputación y
tu historia.

**El tono es lo más importante del juego.** No es un manager frío: es el
conurbano de los noventa, Okupas, Maradona, los Redondos, el potrero, la barra,
la mafia de la AFA, los periodistas que te hacen mierda en vivo. La comedia
negra y el costumbrismo argentino son el producto, no la decoración.

Es **un solo archivo HTML** que se manda por WhatsApp y se abre en el navegador
del celular. Sin instalar nada, sin internet, sin librerías externas.

**Autor:** Leandro Musa (Musa). El juego original es suyo; todo el trabajo
posterior es refactor y expansión sobre esa base.

---

## 2 · LAS LEYES DEL PROYECTO

Reglas que no se rompen. Cada una salió de una decisión explícita del autor.

1. **Un solo archivo.** HTML, CSS y JavaScript juntos. Cero librerías, cero
   archivos externos, cero llamadas a internet.
2. **Menos de 1 MB.** Es para mandar por WhatsApp y que se abra rápido. Se
   permite estirar hasta 1,2 MB si algo lo justifica, nunca más.
3. **No se simplifica el motor original.** Nada de reescribir el juego como un
   script reducido ni reemplazar estructuras por marcadores de posición. Todo
   lo que había sigue estando.
4. **Los niveles son invisibles.** El jugador nunca ve un número de "nivel" de
   un club. Se percibe por el nombre, la cancha y cómo te va.
5. **Toda la prensa negativa la encabeza Flavio Azzaro.** Regla del autor,
   verificada por el validador automático.
6. **Nombres oficiales de cada competencia**, siempre. Liga Profesional, Copa
   Argentina, Brasileirão, Coppa Italia, Copa Libertadores, etc.
7. **Los rivales salen del país y del torneo correctos.** Nunca un equipo de
   otro continente por accidente.
8. **Lenguaje del cuerpo, no de máquina.** Nada de "chasis" ni "pistones":
   isquiotibial, pubis, gemelo, tobillo. Y "potrero", nunca "barrial".
9. **Solo campeones históricos ganan el Mundial.** Única excepción: si vos
   llegás a la final.
10. **Del arte: nada de assets de terceros.** Los dibujos son del autor o
    generados por él. Nada sacado de juegos existentes.

---

## 3 · ESTADO ACTUAL

| | |
|---|---|
| Archivo | `todo_pasa.html` |
| Peso | **800.641 bytes (782 KB)** |
| Líneas | 12.032 |
| Funciones | 401 |
| Constantes | 174 |
| Cartas totales (según el validador) | **260** |
| Commits | 30 |
| Rama | `claude/game-logic-refactor-xlrp9c` |

**Reparto del peso:** JavaScript 91%, CSS 6%, HTML 2%. De ese JavaScript, la
mayor parte es contenido de texto (cartas, clubes, prensa), no lógica. La
sangría son 27 KB (3,7%) y los comentarios 43 KB (6%).

---

## 4 · CÓMO ESTÁ CONSTRUIDO

Todo el juego es Vanilla: sin frameworks, sin build, sin dependencias.

- **Interfaz:** HTML + CSS. Pantallas que se muestran y se ocultan.
- **Sprites y escudos:** SVG generado por código, con el patrón real de cada
  club (franjas, banda, aros, mitades).
- **Motor de partido NES:** un `<canvas>` de 256 × 224 con dibujo por
  `fillRect`, escalado sin suavizado.
- **Sonido:** Web Audio sintetizado en el momento (silbato, gol, error, tic).
  Ningún archivo de audio.
- **Papelitos del festejo:** canvas nativo con partículas.
- **Guardado:** `localStorage`, con dos ranuras — autoguardado y guardado
  manual (`oaky_save_manual_v1`) para que el automático no pise al del jugador.

---

## 5 · EL MOTOR DE CARRERA

### El bucle

Cada temporada se juega en **cinco bloques**:

1. Liga regular (la tabla se mueve fecha a fecha)
2. Copa nacional (eliminatorias)
3. Fase de grupos internacional
4. Playoff de la liga local (si clasificaste)
5. Eliminación directa continental

Después viene el cierre de temporada, el mercado de pases y, cada cuatro años,
el Mundial.

### Los mazos

| Mazo | Cartas | Para qué |
|---|---|---|
| `JUGADAS` | 43 | Momentos de partido (se roban 2 por partido regular) |
| `CAP1` (desde `SITU_C1`) | 60 | Las situaciones del Capítulo 1 |
| `EXTRA` | 87 | Eventos fuera de la cancha |
| `MEDIO` | 43 | Prensa y medios |
| `PRETEMP` | 21 | Pretemporada |
| `LOCURA` | 20 | Eventos de edad |
| `BIZARROS` | 10 | Los raros |
| `MUN_QTE` | 8 | Decisiones de 4 segundos en el Mundial |
| `MUNDIAL_ERA` | 7 | Escenas del Mundial según tu edad |
| `DT_MERCADO` / `DT_PARTIDO` / `DT_PRENSA` | — | Modo director técnico |

Cartas de duelo del motor NES: `CARTAS_ATAQUE` (7), `CARTAS_TIRO` (6),
`CARTAS_DEFENSA` (6), `CARTAS_ULTIMA` (4).

### Cómo se roba una carta (v17) — la bolsa

Antes cada carta salía con `rnd()` puro sobre el mazo disponible: salían
siempre las mismas tres y había cartas que no aparecían nunca en toda la
carrera. Ahora manda `robar(pool, excluir)`:

1. `p.robos` es un contador global de robos. `marcar(id)` anota en qué robo
   salió cada carta (`v.d`), además de cuántas veces (`v.n`) y en qué
   temporada (`v.t`).
2. Mientras queden cartas del pool que **nunca** saliste, se roba sólo de
   ahí. Esa es la bolsa.
3. Cuando la bolsa se vacía vuelven a entrar todas, con peso
   `antigüedad² / (1 + veces²)`: la que salió recién es casi imposible que
   vuelva, la que no sale hace veinte robos entra casi seguro.
4. `robar()` acepta un id a excluir, que se usa para que la segunda decisión
   de un partido nunca repita la carta de la primera.
5. `robarDe(lista, vistas)` hace lo mismo para las listas sueltas de textos
   (mensajes del celu, cartas del entretiempo) que llevan su propio registro.

Verificado en una carrera automática de cuatro temporadas: 63 cartas
distintas, ninguna repetida más de dos veces, cero repeticiones dentro del
mismo partido.

### El resultado de un partido regular (v17)

Un partido regular —liga, clásico, copa hasta cuartos, fase de grupos— se
juega con **dos decisiones** (`jugadasDe(m)` devuelve 2; devuelve 1 para
semifinales, finales y Mundial, que tienen su propio motor). Entre una y otra
avanza el reloj (`segundaJugada`) y se roba otra carta.

| Aciertos | Resultado |
|---|---|
| 2 de 2 | Victoria garantizada |
| 0 de 2 | Derrota garantizada |
| 1 de 2 | Moneda al aire, 50/50 (la inclina el favor de la AFA) |

Ese cálculo ya estaba escrito en `resolverJugada`, pero `m.aciertos` no se
incrementaba en ningún lado: quedaba `undefined`, las dos comparaciones daban
falso y **todos los partidos se resolvían por moneda**. Ahora se lleva la
cuenta de verdad y el resultado depende de lo que elegiste.

### La pretemporada (v17)

`cartaPretemporada()` corre tres pasos en orden: `menuPretemporada()` mira
`p.plata` y te deja elegir entre seis pretemporadas (de gratis a U$D 340k; las
que no podés pagar salen con candado y el precio a la vista), después
`tiendaBolsin()` te vende hasta tres cábalas del brujo con lo que te quedó, y
recién ahí sale la carta de siempre del mazo `PRETEMP`. Cada pretemporada
suma aguante para todo el año (`pretempAgt()`). El bolsín se arma de cero
cada temporada, pero el bono de estadísticas de cada cábala se cobra una sola
vez en toda la carrera.

### Formato de una carta

```js
{id, max, cd, req, t, x, o:[{b, s, st, base, ok, mal, fijo}]}
```

- `id` único, `max` cuántas veces puede salir, `cd` cooldown en temporadas
- `req` condiciones (edad, nivel de club, plata, banderas...)
- `t` título, `x` texto
- `o` las opciones: `b` rótulo, `s` subtítulo, `st` estadística que usa,
  `base` probabilidad, `ok`/`mal` los efectos, `fijo` si nunca falla

**Límites de rótulo:** 28 caracteres si la carta tiene reloj, 42 si no. Lo
controla el validador midiendo los tokens ya expandidos.

### Tokens

Se reemplazan en tiempo real por nombres reales de la partida:
`{COMPA} {ELLOS} {PIBE} {PIB} {DT} {DT_C} {BARRA_LIDER} {REPRE} {IDOLO}`

---

## 6 · EL MOTOR DE PARTIDO NES

Los partidos grandes se juegan en una pantalla de cartucho de 256 × 224.

**Cuándo se activa:** semifinal y final de copa nacional, continental y playoff
de liga, y toda la eliminación directa del Mundial. Antes de cada uno, el
jugador elige si lo juega en la cancha o lo lee como carta de siempre.

### La escala de los jugadores (v17)

Los muñecos de v16 se dibujaban a `ESC_J = 1.7` (18×32 px × 1,7 = 31×54): a
esa escala tapaban media cancha, se comían las líneas y el arco. Ahora
`ESC_J = 1.12` y todas las llamadas de cancha pasan por `dibJ()`, que recibe
la misma Y de siempre y la baja `AJ` píxeles —justo lo que se achicó el
sprite— para que los pies queden apoyados sobre el pasto en vez de flotando.
La pelota bajó de radio 4 a 3 y se acercó al pie. El primer plano del remate
también se achicó (rematador 1,9 → 1,35; arquero 1,5 → 1,15), pero sigue
siendo un plano corto.

### Las franjas de la pantalla

```
y   0 –  14   barra: apellido y GUTS
y  14 –  44   cielo con nubes
y  44 –  64   tribuna con gente y baranda de barrotes
y  64 – 134   cancha de costado, rayas horizontales, dos arcos
y 134 – 224   panel: reloj, marcador y radar (o el menú de comandos)
```

### El flujo de una jugada

```
corrida con la cámara siguiéndote
   → cut-in de las dos caras
      → menú nivel 1: CORRER / PASE / GAMBETA  (+ TIRO si estás en el área)
         → menú nivel 2: la carta concreta
            → animación de la resolución
               → texto del resultado
```

Defendiendo: `QUITAR / BLOQUEAR / FALTA`. Si te llegaron al área, salen las
cuatro cartas de último hombre sin menú previo.

### La posición en la cancha

`D.pos` va de 0 (tu arco) a 100 (el de ellos). Ganar un duelo de ataque te
mueve 16 a 28. Al pasar de 70 aparece TIRO. Al bajar de 26 defendiendo, sos el
último hombre.

### El Aguante (ex Guts) — rediseñado en v17

Sale de tu físico, del bolsín del brujo, del tipo de pretemporada que pagaste
y de lo que te hayas tomado en el túnel (92 a 230 internos, mostrados ×8 como
en el cartucho). Cada carta cuesta.

**Piso mínimo.** El aguante nunca llega a cero absoluto: hay un piso del 20%
del máximo (`AGT_PISO`). Si una carta no te entra en el aire que te queda, la
jugás igual "a pulmón" con un castigo chico (−14, que crece de a 4 si
insistís) en vez del −36 de antes, que era perder el partido sin poder hacer
nada. Fuera de la cancha pasa lo mismo con la forma: `segundoAire()` la
levanta a 22 antes de cada partido, y `_prob0()` tiene piso de 14, así que
una jugada nunca es imposible.

**Riesgo y recompensa.** En el menú del partido hay un comando `AGUANTE`
(`menuAguante`, y `duelAguanteHTML` para el modo sin lienzo) con cuatro
opciones: tomar aire, el bidón del utilero, **infiltrarse** y **la vitamina
del utilero**. Antes de salir a la cancha aparece además `cartaTunel()`
cuando venís golpeado o con la forma por el piso.

Las consecuencias no son cosméticas y se cobran al terminar el partido
(`cobrarRiesgos`):

| | Recompensa | Lo que se paga después |
|---|---|---|
| Infiltración | Aguante al tope, piso al 34%, +7 de probabilidad | La zona tapada pierde 10–20 (más cuanto más lo repetís), −8 de forma y chance de romperse del todo |
| Vitamina (dopaje) | Aguante al tope, piso al 30%, +12 de probabilidad | Control antidopaje: 16% a 62% según cuántas veces lo hiciste. Positivo = suspensión, escándalo, −14 mundo, −12 selección, −16 moral |

### Balance verificado

Con un jugador de 76 contra un rival de 74: gana 58%, empata 25%, pierde 17%.
Con 63 contra Real Madrid: gana 17%, pierde 67%.

---

### Las dos pantallas de arranque (v17)

Antes del menú de modo salen dos pantallas de cartucho, dibujadas enteras con
lo que ya tenía el juego (ley 1: cero archivos externos):

- **La tapa.** `TODO PASA`, seis bustos y `APERTURA 2005`. Los bustos son el
  mismo `caraPixel()` de las figuritas, uno por cada camiseta grande, con los
  colores reales de cada club (`TAPA_CLUBES`).
- **El título.** `TODO PASA / EL JUEGO DE LA AFA`, el escudo dibujado con
  `escudo()`, el `INSERT COIN` parpadeando y el pie
  `© AFA PRO / GRONDONA EDITIONS · © Musa 1990`.

`pintarIntro()` las arma al cargar, `introSiguiente()` pasa de una a otra (a
los 5,2 segundos o al tocar) y `introSaltar()` las cierra. Ambas tienen
scanlines por CSS y botón para saltear.

---

## 7 · LAS BASES DE DATOS

- **134 clubes** con colores, patrón de camiseta, escudo y liga
- **117 estadios** con su nombre real (La Bombonera, El Cilindro, San Siro,
  Anfield, Maracaná...)
- **32 selecciones** del Mundial con bombo, confederación y camiseta
- **11 países** jugables, **17 ligas** con su nombre oficial
- Periodistas, hinchas, ídolos de club, barras bravas y representantes

### Formatos de liga reales

Argentina 30 equipos en dos zonas · Brasil 20 · España, Inglaterra e Italia 20
Alemania y Francia 18

---

## 8 · REGLAS DE SIMULACIÓN

Cosas que se ajustaron para que los números den como en la realidad:

- **Tabla de posiciones:** el líder termina con 28-38 puntos en Argentina y
  75-95 en las ligas grandes. Antes daba 154.
- **Topes de goles por puesto**, calibrados con las últimas 15 Botas de Oro:
  delantero 15-28 normal, 31-42 goleador, 50 récord. Arquero 0-1.
- **Sorteo del Mundial:** uno de cada bombo, máximo dos europeos por grupo.
  Verificado con 3.200 sorteos sin una sola violación.
- **Campeón del Mundo:** solo selecciones campeonas históricas. Verificado con
  5.000 sorteos.
- **Transferencias:** sin clubes repetidos y tu club favorito nunca aparece en
  las primeras ofertas. Verificado con 600 sorteos.
- **Los clásicos** siempre traen una jugada contra tu némesis.

---

## 9 · HISTORIA DEL PROYECTO

Los 26 commits, del más viejo al más nuevo:

1. `0bbbdfc` Original sin modificar (la base del autor)
2. `135f0fe` Competiciones: rivales por país, copas nacionales, campaña continental
3. `2f8f12f` Mundial por etapas, Recopa, Mundial de Clubes
4. `56d5dab` Medios argentinos reales, regla Azzaro, ídolos de club
5. `2183740` Arco de la AFA y 17 escándalos históricos
6. `cf50b10` 87 eventos extradeportivos y 10 bizarros
7. `11e0946` Vitrina rediseñada, finales de conurbano, validador
8. `58f4e5b` Ajustes de banners y layout
9. `45a3d40` La Nuca del Jefe pide ola de calor de verdad
10. `6ae854d` La venganza de la AFA pesa en el arbitraje
11. `284d7a4` Carrera exprés
12. `9274e35` El botón exprés dice lo que hace
13. `e1af844` **FIX CRÍTICO:** volver de la ficha dejaba la partida trabada
14. `1cf65af` Guardado y carga manual con botón a la vista
15. `b7ff880` Equipos reales de las 4 copas, DT, barras, representantes
16. `9d01ca6` Niveles invisibles, Mundial realista, vitrina con club por título
17. `5047567` Tokens en todos los mazos
18. `287ff30` Tabla real, topes de goles, fix de puesto
19. `6d605e5` Calendario en 5 bloques con playoff
20. `c6f5a74` Capítulo 1, sorteo FIFA, grupos jugables, 15 módulos de dinamismo
21. `112ebf9` Cuerpo real, 60 situaciones, dos jugadas por partido
22. `9786844` Modo duelo estilo Tsubasa en los partidos grandes
23. `9c9b7b4` Reboot del modo Tsubasa: ahora se juega y se ve
24. `c7fb608` Motor NES: pantalla 256×224, radar y comandos
25. `b0b304d` Especificación técnica del arte
26. `0b9c085` Ajuste de la especificación al límite de 1024
27. `097cd3a` Cuaderno del proyecto para NotebookLM
28. `72a42a6` Variedad, ritmo del partido y las ocho situaciones de Musa
29. `c602e0c` La tapa del diario reemplaza las cinco placas del cierre
30. `5d9cee9` Barras con apuestas, entretiempo y el bolsín del brujo
31. **v17** Dos decisiones por partido, bolsa de cartas, pretemporada por
    presupuesto, aguante con infiltración y dopaje, sprites chicos y las dos
    pantallas de arranque

---

## 10 · BUGS ENCONTRADOS Y ARREGLADOS

Vale la pena tenerlos anotados para no repetirlos.

**El más grave: `p.pos` pisado por la posición en la tabla.** `ordenarTabla()`
escribía `p.pos = 3` (un número) encima del puesto del jugador ("DEL", "ARQ").
Desde la segunda temporada, **todos los jugadores calculaban su OVR con la
fórmula del arquero**. Se separó en `p.posLiga` y se agregó reparación de
partidas viejas.

**La ficha mataba la partida.** Abrir la ficha guardaba `innerHTML` y al
restaurarlo recreaba el DOM, con lo cual todos los `onclick` morían y el juego
quedaba trabado con el botón SEGUIR muerto. Se pasó a guardar y restaurar los
nodos de verdad.

**Los mensajes nuevos no aparecían.** Las 60 situaciones del Capítulo 1 salían
4 veces en toda una carrera de 21 temporadas, porque dependían de un evento que
se disparaba una vez cada dos años. Se ruteó el 42% de los momentos de partido
al mazo `CAP1`: pasó de 4 a más de 100 por carrera.

**Otros:** `f_causa` leía la variable global en vez del parámetro · los tokens
crudos `{COMPA}` se veían en pantalla en tres mazos distintos · el guardado
manual lo pisaba el automático · `p.temp` quedaba nulo después del VAR · 29
rótulos pasaban de 42 caracteres al expandir el nombre del DT · el líder de la
liga terminaba con 154 puntos · el bonus de final sorteaba rivales de otro país.

---

## 11 · CÓMO SE VERIFICA

Ningún cambio se entrega sin pasar por acá:

1. **Sintaxis:** se extrae el JavaScript del HTML y se corre `node --check`.
2. **`validarMazos()`:** validador propio adentro del juego. Revisa ids
   duplicados, largo de rótulos con tokens expandidos, efectos desconocidos,
   zonas del cuerpo inválidas, ligas, confederaciones, la regla Azzaro y que
   cada carta tenga rama de fracaso.
3. **Sondas estadísticas:** miles de sorteos para verificar las reglas
   (Mundial, transferencias, tabla, balance de duelos).
4. **Carrera completa en navegador real** con Playwright sobre Chromium:
   se juega de punta a punta afirmando cero tokens crudos, cero niveles
   visibles y cero errores de JavaScript.

---

## 12 · LO QUE FALTA — LOS 13 PUNTOS

Lista viva. El autor da la orden de arranque con la palabra **wasabi**.

| # | Punto | Estado |
|---|---|---|
| 01 | Sprites y retratos dibujados por el autor, empotrados en base64 | esperando el arte |
| 02 | Peso del archivo: minificar o no | a definir |
| 03 | Partido 30% más corto (9 turnos → 7, cut-in solo en momentos grandes) | aprobado |
| 04 | RÁPIDO por defecto | aprobado |
| 05 | Botón de velocidad más legible | aprobado |
| 06 | Las 5 placas de fin de temporada → una sola tapa de diario | aprobado, falta definir contenido |
| 07 | Portada nueva del autor | imagen lista, falta bajarla a 236×277 |
| 08 | Techo de peso: 1 MB por disciplina, hasta 1,2 MB si vale la pena | decidido |
| 09 | Postales de las 5 canchas antes del partido | esperando el arte |
| 10 | Mosaicos de tribuna por cancha | esperando el arte |
| 11 | Copa del Mundo y trofeos propios en las placas | esperando el arte |
| 12 | Pantalla de título de la AFA | imagen lista, falta sacarle el INSERT COIN |
| 13 | Estética NES en las pantallas iniciales (fondo negro, tipografía de píxeles) | aprobado, es solo código |

### Pendiente viejo

Ninguno. Los escenarios del Mundial 2026-2050 por edad ya se hicieron
(`MUNDIAL_ERA`, 7 escenas).

### Decisiones tomadas que faltan resolver

- **Punto 06:** cabecera del diario. Propuesta: inventar una (GOLAZO, LA DOCE,
  EL POTRERO) en vez de usar el nombre de un diario real.
- **Punto 06:** el autor pidió que la tapa muestre campeón del torneo, algo que
  quedó sin entender en el dictado, y cómo les fue a los compañeros de camada.
- **Punto 12:** el "© Musa 1990" queda. Falta decidir si el año se deja fijo
  como chiste o lo escribe el código con el año de la carrera.

---

## 13 · EL ARTE

**Estado:** la pantalla de título ya está hecha y aprobada. Faltan 19 piezas.

**Pipeline:**

```
dibujo o referencia del autor
   → IA de arte (máximo 1024 de lado, entrega PNG con alfa)
      → Photopea: bajar con Nearest Neighbor + indexar a 16 colores
         → PNG final en la medida exacta
            → se empotra en el HTML como base64
```

**Lo que la IA de arte puede:** PNG con canal alfa real, hojas de sprites con
grilla ordenada, mosaicos sin costura, mantener el estilo entre entregas,
trabajar desde referencias, varias versiones por pedido.

**Lo que no puede:** entregar en la medida final chica, paleta indexada de 16
colores, salida sin anti-aliasing, pasar de 1024 de lado. Los tres primeros se
arreglan en el paso de Photopea.

**Consecuencia del límite de 1024:** las hojas largas y chatas desperdician el
lienzo. Los retratos, los estadios y las tribunas se piden **de a uno**; solo
el jugador, el arquero y los trofeos van en hoja.

### Presupuesto de imágenes

Libre contra el mega: 269 KB. El empotrado infla 33%, así que el presupuesto
real de PNG es **200 KB**.

| Pieza | Peso PNG | Empotrado |
|---|---|---|
| Pantalla de título | 18 KB | 24 KB |
| 8 retratos | 30 KB | 40 KB |
| Jugador, 10 cuadros | 4 KB | 5 KB |
| Arquero, 4 cuadros | 2 KB | 3 KB |
| 5 estadios | 20 KB | 27 KB |
| 5 tribunas | 8 KB | 11 KB |
| 6 trofeos | 5 KB | 7 KB |
| **Total** | **87 KB** | **117 KB** |

Resultado final estimado: **848 KB**. Adentro del mega.

**Detalle clave del sprite del jugador:** la camiseta y el short van en blanco
puro para que el juego los tiña con los colores de cada club. Si van pintados,
el sprite sirve para un equipo en vez de 134.

**Detalle clave de los retratos:** solo cabeza y cuello, transparente de los
hombros para abajo. La camiseta la pinta el código. Así 8 caras cubren los
134 clubes.

La especificación completa para pasarle a la IA de arte está en
`docs/ESPECIFICACION_ARTE.md`.

---

## 14 · GLOSARIO DE FUNCIONES CLAVE

| Función | Qué hace |
|---|---|
| `momento()` | Cada momento de partido. Decide si va a duelo o a carta |
| `elegirModo(m)` | Pregunta si el partido grande se juega o se lee |
| `momentoTexto()` | El camino de carta de toda la vida |
| `duelIniciar(m,cb,op)` | Arranca el partido en el motor NES |
| `duelTurno()` / `duelMenu()` | El bucle de jugadas y el menú de comandos |
| `duelResolver(k,tipo)` | Resuelve la carta elegida y dispara la animación |
| `esPartidoGrande(m)` | Decide si un partido va al motor NES |
| `dibCancha` / `dibHud` / `dibRadar` | El dibujo de la pantalla del cartucho |
| `dibCara` / `caraSprite` | El retrato del cut-in |
| `sprite(kit,pose,f)` | Sprites de jugador cacheados con contorno |
| `validarMazos()` | El validador de todo el contenido |
| `carta(o)` | Pinta cualquier carta de decisión |
| `aplicar(ef)` | Aplica los efectos de una opción |
| `tokens(s)` | Reemplaza `{COMPA}` y compañía por nombres reales |
| `calcOVR()` | Recalcula el OVR del jugador |
| `cierreTemporada()` | Cierre de año, títulos, mercado |

---

## 15 · CÓMO SE TRABAJA

**La palabra de arranque es `wasabi`.** Mientras no aparezca, se debate y se
planifica pero no se toca una línea de código.

**Un solo archivo, un solo editor.** `todo_pasa.html` son 731 KB en un archivo
de casi 11.000 líneas. Si dos herramientas lo editan en paralelo, se pisan y se
pierde trabajo. Las otras IA aportan especificaciones y diagnósticos, no código
para pegar.

**Todo lo que se propone se mide.** Ejemplo real: se dijo que sobraba mucho
espacio en el código; medido, era 3,7% de sangría y 6% de comentarios. La
propuesta era razonable, el dato la puso en escala.

**Sin herramientas de imagen en el entorno.** Las imágenes tienen que llegar ya
bajadas a la medida final. Un PNG de 1024 no se puede procesar del lado del
código.

**Repositorio:** `leanmusa2025-cloud/Todo-pasa`, rama
`claude/todo-pasa-refactor-l0go8w`.
