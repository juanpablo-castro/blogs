## **TypeScript con Tipado Explícito: Las piezas de puzzle**

---

## **1. La analogía del puzzle: cada pieza tiene una forma**

Imagina tu código como un **puzzle** donde cada pieza representa un tipo específico. Cuando queremos conectar dos piezas, necesitamos que **encajen perfectamente**. En TypeScript, **el tipo** de la pieza debe coincidir con el hueco que estamos intentando llenar.

Por ejemplo, si tu función `processSomeData` requiere un objeto con las propiedades `data: string` e `id: string`, esa **forma** (o tipo) encaja perfectamente cuando le pasas un objeto que satisfaga esa estructura:

```ts
function processSomeData(input: { data: string; id: string }): void {
  // ... lógica de la función
}

// En otra parte del código...
const id: string = fromSomeOtherFunction();
processSomeData({ id, data: "Este es el contenido" });
```

Aquí, **el tipo explícito** actúa como “guía” que nos dice cómo debe lucir el objeto `{ data; id; }`.  
- **La pieza**: `{ data: string; id: string }`  
- **El hueco**: la función `processSomeData(input: { data: string; id: string })`

Si las formas no coinciden, TypeScript nos advertirá antes de tiempo. Esto se traduce en **menos errores en tiempo de ejecución** y un **código más legible**.

---

## **2. Contrato entre la función llamadora y la función llamada**

Cuando declaras explícitamente el tipo de los parámetros de una función y lo que esta función devuelve, estás estableciendo un **contrato claro** entre la función que llama (caller) y la función llamada (callee). Observa este ejemplo:

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
  descuento?: number;
}

function funcionLlamada(): Producto {
  // Imaginemos que realiza validaciones o transformaciones
  const producto: Producto = {
    id: 1,
    nombre: "Camiseta",
    precio: 20,
  };
  return producto;
}

function funcionLlamada2(producto: Producto): ProductoFinal {
  // Imaginemos que aplica un descuento
  const productoFinal: ProductoFinal = {
    ...producto,
    descuento: producto.precio > 15 ? 5 : 0,
  };
  return productoFinal;
}

// Función que llama a las anteriores
async function callerFunction(): Promise<void> {
  const productoInicial: Producto = funcionLlamada();
  const productoFinal: ProductoFinal = funcionLlamada2(productoInicial);

  console.log("Producto Inicial:", productoInicial);
  console.log("Producto Final:", productoFinal);
}
```

1. **`funcionLlamada`**: Devuelve un `Producto`.  
2. **`funcionLlamada2`**: Recibe un `Producto` y devuelve un `ProductoFinal`.  
3. **`callerFunction`**: Declara explícitamente que los resultados son `Producto` y `ProductoFinal`.  

Así, de un **simple vistazo** sabemos:
- Que `productoInicial` es de tipo `Producto`.
- Que `productoFinal` es de tipo `ProductoFinal`.
- Si quisiéramos reemplazar `funcionLlamada` más adelante, sabríamos inmediatamente que debe devolver `Producto` para que la llamada en `callerFunction` siga funcionando.

**¿Podríamos confiar en la inferencia de TypeScript?** Posiblemente sí, pero aquí entra en juego la parte humana: cuando un nuevo compañero de equipo lea este código, no tendrá que **deducir** qué tipo de dato pasa entre funciones; lo verá **claramente indicado**.

---

## **3. Tipado explícito y documentación viva**

**El tipado explícito** en TypeScript hace las veces de **documentación viva**. No necesitas un README oculto en alguna parte explicando qué datos viajan entre tus funciones: la definición de los tipos es **autoexplicativa** y el compilador se encarga de que no haya desajustes.

### 3.1. Beneficios inmediatos

1. **Claridad de intenciones**: Al ver `funcionLlamada2(producto: Producto): ProductoFinal`, sabes exactamente qué entra (un `Producto`) y qué sale (un `ProductoFinal`).  
2. **Depuración más simple**: Si algo falla, el compilador te indica si el tipo que estás intentando encajar no coincide.  
3. **Mantenibilidad**: Si cambias el tipo de `Producto` (por ejemplo, añades una propiedad), TypeScript te obligará a actualizar todas las partes del código donde se usa, evitando sorpresas en producción.

### 3.2. Legibilidad vs. longitud del código

Mucha gente dirá: “Pero así escribimos más letras, el código se ve más largo”. Sin embargo, **menos letras no siempre equivale a más legible**. Un ejemplo extremo:

```txt
"I hp I mk m slf clr" vs. "I hope I make myself clear"
```

Claramente, **menos letras** no es necesariamente más fácil de leer. Lo mismo ocurre cuando infieres tipos en situaciones complejas. Puede terminar en un “rompecabezas” donde cada lector tiene que deducir las formas del puzzle sin ayuda alguna.

---

## **4. Ejemplos adicionales para reforzar la idea**

Veamos ahora un par de escenarios que ilustran la importancia de especificar tipos, tanto en los parámetros como en los retornos de funciones.

### 4.1. Ejemplo: Gestión de pedidos

```ts
type EstadoPedido = 'pendiente' | 'en-transito' | 'entregado' | 'cancelado';

interface Pedido {
  id: number;
  cliente: string;
  estado: EstadoPedido;
}

// Función que crea un pedido
function crearPedido(cliente: string): Pedido {
  const nuevoPedido: Pedido = {
    id: Date.now(),
    cliente,
    estado: 'pendiente',
  };
  return nuevoPedido;
}

// Función que actualiza el estado de un pedido
function actualizarEstadoPedido(pedido: Pedido, nuevoEstado: EstadoPedido): Pedido {
  // Aquí podemos establecer lógica de validaciones...
  pedido.estado = nuevoEstado;
  return pedido;
}

// Función que maneja todo el flujo
function mainPedido(): void {
  // Contrato claro: crearPedido devuelve un Pedido
  const pedidoInicial: Pedido = crearPedido('Juan');
  
  // Contrato claro: actualizarEstadoPedido recibe un Pedido y un EstadoPedido
  const pedidoEnTransito: Pedido = actualizarEstadoPedido(pedidoInicial, 'en-transito');
  
  console.log(`Pedido actualizado a: ${pedidoEnTransito.estado}`);
}
```

El tipado explícito **aporta una tranquilidad enorme**. No tienes que preguntarte si `cliente` es un `string` o un `number`, o si `estado` es una cadena libre o solo una de esas cuatro opciones.

---

### 4.2. Ejemplo: Procesamiento de datos con “piezas” más complejas

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

function calcularDatos(input: DatosIniciales): DatosCalculados {
  const sum = input.valores.reduce((acc, val) => acc + val, 0);
  const max = Math.max(...input.valores);
  const min = Math.min(...input.valores);

  return {
    promedio: sum / input.valores.length,
    maximo: max,
    minimo: min,
  };
}

function mainDatos(): void {
  const data: DatosIniciales = {
    valores: [10, 5, 8, 20, 2],
    fuente: "Sensor XYZ",
  };

  const resultado: DatosCalculados = calcularDatos(data);

  console.log(`Promedio: ${resultado.promedio}`);
  console.log(`Máximo: ${resultado.maximo}`);
  console.log(`Mínimo: ${resultado.minimo}`);
}
```

Cada función define **claramente** el tipo de entrada y el de salida, permitiéndonos ver **al instante** qué campos tenemos disponibles.

---

## **5. Principios de código sostenible**

En el libro **“Clean Code”** de Robert C. Martin, se enfatiza la idea de que **el código se lee muchas veces más de lo que se escribe**. Añadir tipado explícito:

- **No es solo para la máquina**, sino para otros desarrolladores (o tú mismo en el futuro) que deben entender el flujo de datos.  
- **Facilita el mantenimiento**: Si cambias `Producto` o `Pedido`, el compilador te indicará **todos** los lugares donde se use dicho tipo, guiándote en la refactorización.  
- **Genera un contrato**: Cuando tu código crece, cada parte del sistema “sabe” qué forma esperar, y si algo no coincide, **TypeScript te detendrá** antes de que rompas en producción.

Muchos de estos principios se relacionan con la idea de **“claridad sobre concisión”**, porque, al final, el **tiempo de lectura y depuración** es mayor que el tiempo de escribir las líneas adicionales de tipos.

---

## **6. Reflexiones sobre la inferencia vs. la explicitud**

### 6.1. ¿Es la inferencia mala?

No, **en absoluto**. TypeScript tiene una inferencia de tipos muy potente y, en situaciones simples, puede ahorrarte escribir tipos que se deducen inmediatamente. Por ejemplo:

```ts
// Cuando la inferencia es obvia
const nombre = "Juan"; // TS infiere string
const edad = 30;       // TS infiere number
```

Esto no suele suponer un problema y es, incluso, deseable para no saturar de anotaciones triviales. Sin embargo, **cuando entras en funciones complejas** o con **tipos personalizados**, especificar la forma exacta puede ahorrarte dolores de cabeza.

### 6.2. Contraste y balance

La **inferencia** es suficiente para variables locales “sencillas”. El **tipado explícito** es muy útil y casi imprescindible para funciones de relevancia, modelos de dominio y contratos entre partes del sistema. Al final, **cada equipo** debe encontrar su **propio balance**.

---

## **7. Conclusión: encajando piezas y evitando sorpresas**

- **El tipado explícito** puede parecer “sobre-escritura” cuando trabajas en proyectos pequeños, pero **en proyectos complejos** se convierte en un **aliado esencial**.  
- **Establecer contratos** claros entre función llamadora y función llamada evita confusiones e integra la **documentación** directamente en el código.  
- **Piezas de puzzle**: Cada **tipo** es una **forma** que encaja solo en un sitio. Esto mejora la legibilidad y reduce los errores.  
- **No hay una única verdad**: Usa la inferencia cuando sea obvio y el tipado explícito cuando aporte claridad.  
- **Clean Code y mantenibilidad**: Recuerda que un poco más de detalle hoy puede salvar horas de debugging mañana.

