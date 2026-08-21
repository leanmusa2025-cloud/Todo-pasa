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

## PARTE 1 · Contestá esto antes de dibujar

Necesito saber qué podés hacer y qué no. Contestá una por una, con sinceridad.
Si algo no lo podés hacer, decilo y buscamos otra forma.

1. ¿Podés exportar **PNG con canal alfa**, es decir con fondo transparente de
   verdad, no un damero gris ni un fondo blanco?

2. ¿Podés entregar en una **medida exacta** que yo te pida (por ejemplo
   128 × 160 píxeles), o solamente entregás cuadrados de 1024 × 1024?

3. ¿Podés limitar la imagen a una **paleta de 16 colores** como máximo?

4. ¿Podés entregar **sin anti-aliasing**, con los bordes duros de un píxel,
   sin difuminado ni degradés suaves?

5. ¿Podés armar una **hoja de sprites con grilla exacta**: por ejemplo 5
   columnas por 2 filas, con todas las casillas del mismo tamaño y cada figura
   centrada y sin tocarse entre sí?

6. ¿Podés hacer un **mosaico que repita sin costura** (seamless tile), donde
   el borde izquierdo encaje con el derecho al repetirlo?

7. ¿Podés mantener **el mismo personaje y el mismo estilo** a lo largo de
   varias imágenes distintas, o cada pedido te sale diferente?

8. ¿Podés trabajar **a partir de una foto o un dibujo de referencia** que te
   doy, o sólo generás desde cero con texto?

9. ¿Cuál es la **resolución máxima** que entregás?

10. ¿En qué **formatos** entregás? ¿PNG, JPG, WebP?

11. ¿Podés entregar la misma imagen **en varias versiones** de una sola vez,
    para elegir la mejor?

**Importante:** si la respuesta a las preguntas 2, 3 y 4 es que no, no hay
drama. Entregá grande y con los colores que salgan: el autor la baja después
a la grilla exacta con Photopea usando interpolación Nearest Neighbor. Lo que
sí necesito sí o sí es la 1 (fondo transparente) donde el listado lo pida, y
la 5 (grilla ordenada) en las hojas.

---

## PARTE 2 · Las piezas que hacen falta

| # | Pieza | Medida final | Fondo | Colores | Peso objetivo |
|---|-------|--------------|-------|---------|---------------|
| 1 | Pantalla de título | 256 × 257 | negro puro | 32 | 18 KB |
| 2 | Retratos de jugadores | hoja 1024 × 160 (8 casillas de 128 × 160) | transparente | 16 | 30 KB |
| 3 | Jugador en la cancha | hoja 160 × 96 (5 × 2 casillas de 32 × 48) | transparente | 16 | 4 KB |
| 4 | Arquero | hoja 128 × 48 (4 casillas de 32 × 48) | transparente | 16 | 2 KB |
| 5 | Estadios | hoja 640 × 80 (5 casillas de 128 × 80) | negro puro | 16 | 20 KB |
| 6 | Mosaicos de tribuna | hoja 320 × 32 (5 casillas de 64 × 32) | sin transparencia | 16 | 8 KB |
| 7 | Trofeos | hoja 384 × 64 (6 casillas de 64 × 64) | transparente | 16 | 5 KB |
| | **TOTAL** | | | | **87 KB** |

### Qué va en cada casilla

**2 · Retratos** — Ocho cabezas distintas, de frente. **Solo cabeza y cuello:
de los hombros para abajo tiene que quedar transparente.** Sin camiseta y sin
ropa, porque el color de cada club se lo pone el juego por encima.

**3 · Jugador** — Un futbolista de perfil mirando a la derecha, cuerpo entero,
diez poses en este orden exacto:

```
Fila 1:  corriendo 1 · corriendo 2 · corriendo 3 · corriendo 4 · parado
Fila 2:  pateando · saltando de cabeza · barriéndose · caído · festejando
```

**La camiseta y el short tienen que estar en blanco puro (#FFFFFF), lisos, sin
números ni escudos.** El juego los tiñe después con los colores de cada club.
Si van pintados de un color, el sprite sirve para un solo equipo en vez de 134.

**4 · Arquero** — En este orden: parado · volando arriba · volando abajo ·
atrapando la pelota.

**5 · Estadios** — Cinco canchas vistas desde afuera, de noche, con las luces
encendidas. Siluetas simples, reconocibles de lejos, cielo negro.

**6 · Tribunas** — Cinco fragmentos de tribuna llena de gente, vistos de frente
y de lejos, cada uno con una combinación de colores distinta. **Tienen que
repetir sin costura a lo ancho.** Adelante va una baranda de barrotes blancos
verticales.

**7 · Trofeos** — Seis copas de fútbol distintas entre sí, cada una centrada
en su casilla, con un brillo blanco del lado izquierdo.

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

**Sobre las medidas:** si no podés entregar en la medida exacta, entregá grande
(1024 o lo que puedas) y el autor la baja. Pero mantené **la proporción** de la
casilla: si la pieza es de 128 × 160, entregala en una proporción 4:5, no en un
cuadrado, porque al bajarla se deforma.

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
retratos.png
jugador.png
arquero.png
estadios.png
tribunas.png
trofeos.png
```

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
