# TODO PASA · Especificación técnica para la IA de arte

**Para qué es esto:** el estilo y el contenido de los dibujos salen de las
fotos de referencia que te va a mostrar el autor. Este documento es sólo el
**formato técnico**: en qué medida, con qué fondo, con cuántos colores y con
qué peso tiene que entregarse cada pieza para que entre en el juego.

El juego es un solo archivo HTML que se manda por WhatsApp y se abre en el
navegador del celular. Pesa 731 KB y no puede pasar de 1 MB. Las imágenes
van empotradas adentro del archivo, y ese proceso las agranda un 33%. Por eso
**el presupuesto total de imágenes es de 200 KB**, y cada kilobyte cuenta.

---

## PARTE 1 · Lo que la herramienta puede y lo que no

Ya está preguntado y contestado. Este es el resultado:

| Puede | No puede |
|-------|----------|
| PNG con canal alfa transparente de verdad | Entregar en la medida final chica |
| Hojas de sprites con grilla ordenada | Paleta indexada de 16 colores |
| Mosaicos que repiten sin costura | Salida sin anti-aliasing |
| Mantener el estilo entre entregas | Pasar de 1024 píxeles de lado |
| Trabajar desde fotos de referencia | |
| Varias versiones por pedido | |

Los tres "no puede" se resuelven todos en el mismo paso de postproducción,
que hace el autor en Photopea:

```
Imagen → Tamaño de imagen → medida final, Nearest Neighbor
Imagen → Modo → Color indexado → 16 colores
Archivo → Exportar como → PNG
```

Ese paso arregla la medida, la paleta y el anti-aliasing de una sola vez.

---

## PARTE 2 · El límite de 1024 y cómo se reparte

La herramienta entrega como máximo 1024 píxeles de lado. Eso obliga a **no
pedir hojas largas y chatas**, porque desperdician el lienzo: en una hoja de
1024 × 160 con ocho casillas, cada figura queda con 128 píxeles para dibujar,
que es la medida final, y sale borrosa.

**Regla: cuanto más cuadrada la hoja, mejor rinde.** Las piezas que necesitan
detalle se piden de a una.

| Pieza | Lienzo a pedir | Medida final | Reducción |
|-------|----------------|--------------|-----------|
| Un retrato, de a uno | 819 × 1024 | 128 × 160 | 8× |
| Jugador, hoja de 5 × 2 | 1024 × 614 | 160 × 96 | 6,4× |
| Arquero, hoja de 2 × 2 | 683 × 1024 | 64 × 96 | 16× |
| Un estadio, de a uno | 1024 × 640 | 128 × 80 | 8× |
| Un mosaico de tribuna, de a uno | 1024 × 512 | 64 × 32 | 16× |
| Trofeos, hoja de 3 × 2 | 1024 × 683 | 192 × 128 | 5,3× |

Son veinte pedidos en total: ocho retratos, cinco estadios, cinco tribunas,
una hoja de jugador, una de arquero y una de trofeos.

**Los ocho retratos van en la misma conversación, uno atrás del otro**, y en
cada uno hay que decir "mismo estilo y mismo trazo que el anterior, pero con
el pelo así". Si se abre una charla nueva por cada cara, salen de ocho juegos
distintos.

---

## PARTE 2B · Las piezas, una por una

| # | Pieza | Medida final | Fondo | Peso objetivo |
|---|-------|--------------|-------|---------------|
| 1 | Pantalla de título | 256 × 257 | negro puro | 18 KB · **ya está hecha** |
| 2 | 8 retratos | 128 × 160 cada uno | transparente | 30 KB |
| 3 | Jugador, 10 cuadros | hoja 160 × 96 | transparente | 4 KB |
| 4 | Arquero, 4 cuadros | hoja 64 × 96 | transparente | 2 KB |
| 5 | 5 estadios | 128 × 80 cada uno | negro puro | 20 KB |
| 6 | 5 mosaicos de tribuna | 64 × 32 cada uno | sin transparencia | 8 KB |
| 7 | 6 trofeos | hoja 192 × 128 | transparente | 5 KB |
| | **TOTAL** | | | **87 KB** |

### Qué va en cada casilla

**2 · Retratos** — Ocho cabezas distintas, de frente. **Solo cabeza y cuello:
de los hombros para abajo tiene que quedar transparente.** Sin camiseta y sin
ropa, porque el color de cada club se lo pone el juego por encima.

**3 · Jugador** — Un futbolista de perfil mirando a la derecha, cuerpo entero,
diez poses en este orden exacto, grilla de 5 columnas por 2 filas:

```
Fila 1:  corriendo 1 · corriendo 2 · corriendo 3 · corriendo 4 · parado
Fila 2:  pateando · saltando de cabeza · barriéndose · caído · festejando
```

**La camiseta y el short tienen que estar en blanco puro (#FFFFFF), lisos, sin
números ni escudos.** El juego los tiñe después con los colores de cada club.
Si van pintados de un color, el sprite sirve para un solo equipo en vez de 134.

**4 · Arquero** — Grilla de 2 columnas por 2 filas, en este orden:

```
Fila 1:  parado · volando arriba
Fila 2:  volando abajo · atrapando la pelota
```

**5 · Estadios** — Cinco canchas vistas desde afuera, de noche, con las luces
encendidas. Siluetas simples, reconocibles de lejos, cielo negro. Una por
pedido.

**6 · Tribunas** — Cinco fragmentos de tribuna llena de gente, vistos de frente
y de lejos, cada uno con una combinación de colores distinta. **Tienen que
repetir sin costura a lo ancho.** Adelante va una baranda de barrotes blancos
verticales. Una por pedido.

**7 · Trofeos** — Seis copas de fútbol distintas entre sí, grilla de 3 columnas
por 2 filas, cada una centrada en su casilla, con un brillo blanco del lado
izquierdo.

---

## PARTE 3 · Reglas de formato, para todas las piezas

```
- Pixel art estilo NES / Famicom de 1990
- Paleta limitada: máximo 16 colores por pieza (32 en la pantalla de título)
- Sin anti-aliasing: bordes duros de un píxel, nada de degradés suaves
- Contorno negro de un píxel alrededor de cada figura
- Sombreado plano de dos tonos por color, nada más
- Formas simples, que se entiendan a tamaño chico
- Sin ningún texto escrito adentro de la imagen
- No imitar personajes de videojuegos que ya existan
```

**Sobre el texto:** salvo el logo del título, ninguna pieza lleva letras. Los
textos los escribe el juego con su propia tipografía de píxeles, así se pueden
cambiar el año, los nombres y los carteles sin volver a dibujar nada.

**Sobre las medidas:** entregá siempre en el lienzo que pide la tabla de la
Parte 2, con el lado más largo en 1024. La **proporción tiene que ser exacta**:
si se entrega un cuadrado cuando la pieza es 4:5, al bajarla se deforma y hay
que rehacerla.

---

## PARTE 4 · Cómo entregar

**Una hoja por categoría, no archivos sueltos.** Ocho retratos en una sola
imagen pesa bastante menos que ocho imágenes separadas, porque comparten la
paleta y comprimen mejor.

En cada hoja:
- Todas las casillas **del mismo tamaño exacto**
- Cada figura **centrada** en su casilla
- Las figuras **no se tocan ni se pisan** entre sí
- **Sin líneas de separación** dibujadas entre casillas
- El orden de las casillas es **el que dice este documento**, de izquierda a
  derecha y de arriba hacia abajo

Nombrá los archivos así:

```
titulo.png
retrato_1.png ... retrato_8.png
jugador.png
arquero.png
estadio_bombonera.png  estadio_monumental.png  estadio_cilindro.png
estadio_gasometro.png  estadio_libertadores.png
tribuna_1.png ... tribuna_5.png
trofeos.png
```

Las piezas que van de a una se pueden mandar sueltas: el autor las junta
después en Photopea, o las manda sueltas y se juntan del otro lado. Juntarlas
en una hoja ahorra unos kilobytes, pero no es obligatorio.

---

## PARTE 5 · Lo que arruina una pieza

Estas cinco cosas obligan a rehacer todo. Revisalas antes de entregar.

1. **Fondo blanco en vez de transparente.** Queda un cuadrado blanco alrededor
   de la figura adentro del juego.
2. **Bordes difuminados.** Al bajarlo a la grilla queda todo barroso y no hay
   forma de arreglarlo salvo redibujar.
3. **Casillas de distinto tamaño** en una hoja. El juego corta por medida fija;
   si una casilla mide distinto, salen todas cortadas al medio.
4. **Figuras que se salen de su casilla** o que se pisan con la de al lado.
5. **Camisetas pintadas de color** en el sprite del jugador. Tienen que ir en
   blanco puro para que el juego las pueda teñir.

---

## PARTE 6 · Prioridad

Si no se puede hacer todo, este es el orden de importancia:

1. **Retratos** (pieza 2) — es lo que más se ve y lo que más cambia el juego
2. **Jugador** (pieza 3) — se ve en cada jugada de cada partido
3. **Estadios** (pieza 5) — aparece antes de cada partido grande
4. **Tribunas** (pieza 6)
5. **Trofeos** (pieza 7)
6. **Arquero** (pieza 4)

La pantalla de título (pieza 1) ya está hecha.
