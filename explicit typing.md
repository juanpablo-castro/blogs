## **TypeScript con Tipado Explícito: La importancia de encajar “piezas” y “huecos”**

> **“I hp I mk m slf clr” vs. “I hope I make myself clear”**  
>  
> ¿Notas la diferencia? Ambas frases comunican algo parecido, pero la segunda es infinitamente más clara. En programación ocurre algo similar: a veces renunciamos a escribir un poco más de información con tal de “aligerar” el código, pero terminamos haciendo que sea menos comprensible. El **tipado explícito** en TypeScript aporta esa claridad que se suele sacrificar por “menos letras”.
---

## **1. “Menos letras” no siempre equivale a “más claridad”**

En TypeScript, podemos confiar mucho en la inferencia de tipos. El compilador es bastante listo. Sin embargo, al igual que en la frase abreviada _“I hp I mk m slf clr”_, **el hecho de no escribir algunos caracteres** hace que el mensaje se vuelva confuso para la persona que lo lee (o lo mantiene).

Sucede lo mismo con el **tipado implícito**. Puede que “compile bien”, pero complica la lectura y **reduce la explicitud** de lo que estás haciendo. Al final, **tú y tu equipo pasaréis más tiempo** deduciendo qué tipo debería tener esa variable o esa función.

Por ejemplo:

```ts
// Inferencia (bienintencionada, pero menos clara)
function obtenerUsuario() {
  return { id: 1, nombre: "Alice" };
}
```

Sí, TypeScript infiere que esto es un `{ id: number; nombre: string }`. Pero quien lo lea por primera vez no tiene esa pista visual inmediata ni la confirmación de si hay otras propiedades que se omiten. 

---

## **2. Inferencia (bienintencionada) vs. Tipado Explícito (contrato fuerte)**

En TypeScript, la inferencia de tipos puede ser muy útil en casos sencillos, pero **no siempre** nos da la claridad necesaria para mantener un contrato fuerte en todo nuestro código. Veámoslo con un ejemplo:

### **2.1. Inferencia bienintencionada, pero menos clara**

```ts
// Función con inferencia automática
function obtenerUsuario() {
  // Aquí toda la responsabilidad recae en este return...
  return { id: 1, nombre: "Alice" };
}

// Asignación de la variable: no hay anotación de tipo
// TypeScript infiere { id: number; nombre: string } 
// pero no es explícito para quien lee el código.
const usuario = obtenerUsuario();
procesarUsuario(usuario.id);

```

- A simple vista, puede **no ser obvio** si el objeto devuelto tendrá siempre la misma estructura (¿existe un campo "edad"? ¿"apellido"?).  
- Cuando un proyecto crece, este enfoque puede causar **incertidumbre** o errores ocultos, porque no tenemos un “contrato” claramente definido.  
- Si en un futuro la función `obtenerUsuario` retorna algo distinto, TypeScript podría inferir un tipo diferente, **sin que te des cuenta** de los cambios hasta que uses `usuario` en otra parte del código y explote por un campo faltante.

### **2.2. Tipado explícito, un contrato sólido en varias partes del código**

```ts
// Definimos un contrato llamado "Persona"
interface Persona {
  id: number;
  nombre: string;
}

// Función con un contrato de salida claro
function obtenerUsuario2(): Persona {
  // Ahora sé que debe cumplir la forma "Persona"
  return { id: 1, nombre: "Alice" };
}

// La variable también declara explícitamente que es "Persona"
const usuario2: Persona = obtenerUsuario2();

procesarUsuario({id: usuario2.id})
```

- Aquí, **la forma** (`Persona`) queda definida en un solo lugar.  
- Toda función que devuelva un `Persona` **debe** cumplir con `{ id: number; nombre: string }`.  
- Si el día de mañana modificas `Persona` (por ejemplo, añades `edad: number;`), TypeScript te avisará si `obtenerUsuario2` no provee la nueva propiedad y no coincidirá con el contrato.  
- **El resultado**: un código más auto-documentado y legible para cualquier persona que lo lea en un futuro (o para ti mismo dentro de unos meses).

En el contexto de la **analogía del puzzle**, podríamos decir:

- **Hueco** (la función `obtenerUsuario2` que retorna `Persona`): indica qué forma debe tener la pieza de salida (con campos `id` y `nombre`).  
- **Pieza** (el objeto real que retornas `{ id: 1, nombre: "Alice" }`): debe encajar perfectamente en ese hueco.  
- **Cuando asignas a la variable** `usuario2: Persona`, estás **reforzando** el contrato: el hueco `usuario2` solo admite piezas de tipo `Persona`. Así, si cambias la forma de la pieza de forma incompatible, TypeScript te lo hará saber de inmediato en vez de que el error surja en otro lugar más adelante.
---

## **3. Contratos claros entre función llamadora y función llamada**

Este concepto de “piezas” y “huecos” se traduce, en la práctica, a un “contrato” entre quien llama a la función (caller) y la función llamada (callee). Observa este ejemplo:

```ts
interface Producto {
  id: number;
  nombre: string;
  precio: number;
}

interface ProductoFinal {
  id: number;
  nombre: string;
  precio: number;
  descuento: number;
}

// Hueco: requiere un Producto y devuelve un Producto
function funcionLlamada(): Producto {
  return {
    id: 1,
    nombre: "Zapatos",
    precio: 50
  };
}

// Hueco: recibe un Producto y devuelve un ProductoFinal
function funcionLlamada2(producto: Producto): ProductoFinal {
  const descuentoAplicado = producto.precio > 30 ? 10 : 0;
  return {
    ...producto,
    descuento: descuentoAplicado
  };
}

async function callerFunction(): Promise<void> {
  // Pieza devuelta por funcionLlamada (que encaja en el hueco Producto)
  const productoInicial: Producto = funcionLlamada();

  // Pieza devuelta por funcionLlamada2 (que encaja en el hueco ProductoFinal)
  const productoConDescuento: ProductoFinal = funcionLlamada2(productoInicial);

  console.log(productoInicial, productoConDescuento);
}
```

- **funcionLlamada** → hueco de retorno es `Producto`. La pieza real que retorna es `{ id, nombre, precio }`.  
- **funcionLlamada2** → hueco de retorno es `ProductoFinal` y, a la vez, su “hueco” de parámetro es `Producto`. La pieza real que recibe es `productoInicial`.  
- **callerFunction** → Al asignar cada retorno a `productoInicial` (`Producto`) y `productoConDescuento` (`ProductoFinal`), especificamos **el hueco** donde esa pieza debe encajar.

Si en un futuro se cambia `funcionLlamada` para devolver un objeto distinto, la definición explícita de `productoInicial: Producto` te avisará al instante si no coincide. **Evitas** desperfectos en otras partes del código y ahorras horas de debugging.

---

## **4. Ejemplos para reforzar la idea**

### 4.1. Actualizar estados con tipos de unión

```ts
type EstadoPedido = 'pendiente' | 'en-transito' | 'entregado' | 'cancelado';

interface Pedido {
  id: number;
  cliente: string;
  estado: EstadoPedido;
}

function actualizarEstadoPedido(hueco: Pedido, nuevoEstado: EstadoPedido): Pedido {
  hueco.estado = nuevoEstado;
  return hueco;
}

function mainPedido(): void {
  // Pieza: un pedido
  const miPedido: Pedido = {
    id: 100,
    cliente: "Juan",
    estado: "pendiente",
  };

  // Cambiando estado, encaja “en-transito” en el hueco EstadoPedido
  const pedidoActualizado: Pedido = actualizarEstadoPedido(miPedido, 'en-transito');
  console.log(pedidoActualizado);
}
```

- El “hueco” en la definición de `nuevoEstado` solo acepta `'pendiente' | 'en-transito' | 'entregado' | 'cancelado'`.  
- Si intentas pasar `"reembolsado"`, TypeScript protestará porque esa pieza no encaja en ese hueco.

### 4.2. Procesar datos con interfaces compuestas

```ts
interface DatosIniciales {
  valores: number[];
  fuente: string;
}

interface DatosCalculados {
  promedio: number;
  maximo: number;
  minimo: number;
}

function calcularEstadisticas(hueco: DatosIniciales): DatosCalculados {
  const { valores } = hueco;
  const sum = valores.reduce((acc, val) => acc + val, 0);
  const max = Math.max(...valores);
  const min = Math.min(...valores);

  return {
    promedio: sum / valores.length,
    maximo: max,
    minimo: min
  };
}

function mainDatos(): void {
  const pieza: DatosIniciales = { valores: [10, 5, 8], fuente: "Sensor" };
  const resultado: DatosCalculados = calcularEstadisticas(pieza);

  console.log("Promedio:", resultado.promedio);
}
```

El hueco `DatosIniciales` declara `valores` (array de numbers) y `fuente` (string). La pieza que pasamos (`{ valores: [10, 5, 8], fuente: "Sensor" }`) **encaja perfectamente**.  
Análogamente, la función retorna un “hueco” de tipo `DatosCalculados`, y la pieza real que construimos coincide con esa definición.

---

## **5. Tipado explícito y código sostenible**

En el libro **Clean Code** de Robert C. Martin, se insiste en la idea de que **el código se lee más veces de las que se escribe**. Añadir anotaciones de tipos puede parecer tedioso inicialmente, pero **facilita**:

1. **Auto-documentación**: El propio código describe lo que espera y devuelve.  
2. **Comunicación en el equipo**: Cualquiera que llegue sabe el contrato exacto sin investigar en la lógica interna.  
3. **Menos errores**: Te adelantas a fallos en tiempo de ejecución, pues TypeScript te avisa antes del compilado.  
4. **Refactorizaciones sin miedo**: Si cambias la forma de un `Producto` o `Pedido`, TypeScript te guiará por todas las partes del código que dependan de él.

---

## **6. ¿Cuándo conviene el tipado explícito y cuándo la inferencia?**

No se trata de **demonizar la inferencia**. En casos sencillos (por ejemplo, `const nombre = "Alice";`), la inferencia te ahorra escribir `: string` redundantes. Donde **realmente** brilla el tipado explícito es en **funciones de cierto peso**, **modelos de dominio**, o **cuando intervienen varias interfaces compuestas**. En esos escenarios, la claridad que aporta es **oro puro** a la hora de mantener el proyecto.

---

## **7. Conclusiones finales**

- **“Menos letras” no siempre es sinónimo de “código más legible”**. A veces, unos pocos caracteres extra hacen que la intención sea cristalina.  
- **Huecos vs. piezas**: Definir tus funciones y variables con tipos explícitos crea huecos bien formados donde solo encajan piezas del tipo correcto.  
- **Contratos sólidos**: Si cambias la forma de la pieza, el hueco te avisará al instante. De lo contrario, el error puede manifestarse más tarde y en otro lugar, haciendo más complejo el debugging.  
- **Código sostenible**: En proyectos grandes, el tipado explícito se convierte en un aliado esencial para la mantenibilidad y la colaboración en equipo.

En resumen, **“I hp I mk m slf clr”** puede funcionar, pero **“I hope I make myself clear”** es infinitamente mejor.  
Con TypeScript y el **tipado explícito** sucede lo mismo: no es obligatorio, pero **clarifica y previene** muchos problemas. Y al final, un código claro es un código que se puede evolucionar sin sobresaltos.

---

### **¡Gracias por leer/escuchar esta charla!**  
Espero que esta analogía y estos ejemplos te ayuden a ver el valor de hacer un pequeño esfuerzo adicional para explicitar tus tipos. Así, cada pieza encaja perfectamente en su hueco, y tu proyecto crece de forma sólida y legible para todos.
